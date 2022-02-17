---
layout: post 
title: Spring Data JPA Paging
date: 2022-02-17
categories: JPA
---

Spring Data JPA 를 이용해서 paging 처리를 할 때, 
부가적인 count query 를 발생시키지 않는 방법을 정리한다. 

## Setup

아래와 같이 의존성으로, Spring Data JPA 와 H2 를 추가하자.
테스트를 위해 Embedded DB 로 H2 를 사용한다.

![](/image/spring-data-jpa-paging-project-setup.png)

Entity 로 Book 을 정의하자.

```kotlin
@Entity
class Book(
    
    @Id @GeneratedValue
    val id: Long? = null,
    
    val name: String
)
```

그리고, BookRepository 를 정의하자. 
Paging 을 위해, PagingAndSortingRepository interface 를 상속한다.

```kotlin
interface BookRepository : PagingAndSortingRepository<Book, Long>
```

## Test

JPA 관련 테스트이기 때문에, @DataJpaTest 를 활용해보자.

```kotlin
@DataJpaTest
internal class BookRepositoryTest {
    
}
```

각 테스트가 실행될 때마다, 데이터를 setUp 하자.

```kotlin
@DataJpaTest
internal class BookRepositoryTest {

    @Autowired
    private lateinit var bookRepository: BookRepository

    @BeforeEach
    internal fun setUp() {
        val books = listOf(
            Book(name = "jpa-paging-1"),
            Book(name = "jpa-paging-2"),
            Book(name = "jpa-paging-3"),
            Book(name = "jpa-paging-4"),
            Book(name = "jpa-paging-5"),
            Book(name = "jpa-paging-6"),
            Book(name = "jpa-paging-7"),
            Book(name = "jpa-paging-8"),
            Book(name = "jpa-paging-9"),
            Book(name = "jpa-paging-10")
        )

        bookRepository.saveAll(books)
    }
}
```

이제 test method 를 작성해보자.

```kotlin
@Test
internal fun `Page 를 return 하면, count query 가 발생한다`() {
    val firstPage = PageRequest.of(0, 4)
    val books: Page<Book> = bookRepository.findAll(firstPage)

    books.map {
        println("name: ${it.name}")
    }
}
```

위 메서드의 실행 결과는 어떻게 나올까 ?
물론, jpa-paging-1, jpa-paging-2, jpa-paging-3, jpa-paging-4 가 순서대로 출력될 것이다.

확인해보자.

![](/image/spring-data-jpa-paging-count-query.png)

예상대로 출력되었다. `그런데, 왜 select 가 두 번 query 되었을까 ?` 

두 번째 select 는 count query 인 것을 알 수 있다.
공식 문서에 따르면, 다음과 같다.

```text
A Page knows about the total number of elements and pages available. 
It does so by the infrastructure triggering a count query to calculate the overall number
```

번역하면,

```text
Page 는 사용 가능한 요소와 페이지의 총 수를 알고 있다.
그래서, 전체 수를 계산하기 위해 카운트 쿼리가 트리거된다.
```

Page interface 의 정의를 보면, totalPage 와 totalElement 를 return 하는 메서드가 정의되어있다.

![](/image/spring-data-jpa-paging-page-interface.png)

만약에 조회해야하는 elements 가 많을 때는, page 별 select 를 할 때마다 count query 가 수행되어 expensive 하다.

## 개선

그럼 어떻게 count query 가 수행되지 않게 할 수 있을까 ?
간단하다. List 를 return 하게 하면 된다. Page 를 build 하기 위한 count query 가 필요없기 때문이다.

직접 확인해보자.
BookRepository 에 Pageable instance 를 받고, List 를 리턴하는 query method 를 정의하자.

```kotlin
interface BookRepository : PagingAndSortingRepository<Book, Long> {

    fun findAllBy(pageable: Pageable): List<Book>
}
```

그리고, 위 메서드를 사용하는 테스트 코드를 작성하자.

```kotlin
@Test
internal fun `List 를 return 하면, count query 가 발생하지 않는다`() {
    val firstPage = PageRequest.of(0, 4)
    val books: List<Book> = bookRepository.findAllBy(firstPage)

    books.map {
        println("name: ${it.name}")
    }
}
```

실행결 결과는 아래처럼, count query 가 발생하지 않는다.

![](/image/spring-data-jpa-paging-not-count-query.png)

---
https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.special-parameters
