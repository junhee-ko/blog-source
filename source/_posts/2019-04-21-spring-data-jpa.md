---
layout: post
title: Spring Data JPA
date: 2019-04-21
categories: JPA
---

데이터 접근 계층 (Data Access Layer) 는 CRUD 로 불리는 등록, 수정, 삭제, 조회 코드를 반복해서 개발해야 한다. 
JPA 를 사용해서 데이터 접근 계층을 개발할 때도 문제가 발생한다.

```java
public class MemberRepository{
  
  @PersistenceContext
  EntityManager em;
  
  public void save(Member member) {...}
  public Member findOne(Long id) {...}
  public List<Member> findAll() {...}
  
  public Member findByUsername(String username) {...}
}

public class ItemRepository{
  
  @PersistenceContext
  EntityManager em;
  
  public void save(Item item) {...}
  public Member findOne(Long id) {...}
  public List<Member> findAll() {...}
}
```

위 코드를 보면, 회원 리포지토리와 상품 리포지토리가 하는 일이 비슷하다.
이 문제를 해결하려면 제네릭과 상속을 적절히 사용해서 공통 부분을 처리하는 부모 클래스를 만들면 된다. 이것을 보통 GenericDAO 라고 한다. 
하지만 이것은, 공통 기능을 구현하는 부모 클래스에 종속되고 구현 클래스 상속이 가지는 단점이 있다. 

## 스프링 데이터 JPA

스프링 데이터 JPA 는 스프링 프레임워크에서 JPA 를 편리하게 사용할수 있도록 지원하는 프로젝트이다. 
이 프로젝트는 데이터 접근 계층을 개발할 때 지루하게 반복되는 CRUD 문제를 세련된 방법으로 해결한다. 
데이터 접근 계층을 개발할 때 구현 클래스 없이 인터페이스만 작성해도 개발을 완료할 수 있다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
  Member findByUsername(String username);
}

public interface ItemRepository extends JpaRepository<Item, Long> {
}
```

회원과 상품 리포지토리 구현체는 애플리케이션 실행 시점에 스프링 데이터 JPA 가 생성해서 주입해준다. 즉, 개발자가 직접 구현체를 개발하지 않아도 된다.
일반적인 CRUD 메소드는 JpaRepository 인터페이스가 공통으로 제공하지만, 
MemberRepository.findByUsername(...) 처럼 직접 작성한 공통으로 처리할 수 없는 메소드는 스프링 데이터 JPA 가 메소드 이름을 분석해서 JPQL 을 실행한다.

## 스프링 데이터 프로젝트

스프링 데이터 JPA 프로젝트는 JPA 에 특화된 기능을 제공한다.

![](/image/spring-data-jpa.png)

## JpaRepository 계층 구조

JpaRepository interface의 계층 구조를 직접 확인해보자.

JpaRepository 는 org.springframework.data.jpa.repository 에 속해있다.
JpaRepository 는 다시 PagingAndSortingRepository 를 상속하고 있다.

![](/image/spring-data-jpa-repository.png)

PagingAndSortingRepository interface 의 package 는 org.springframework.data.jpa.repository 가 아니라, org.springframework.data.repository 이다.
PagingAndSortingRepository 는 다시 CrudRepository 를 상속한다.

![](/image/spring-data-jpa-paging-and-sorting-repository.png)


CrudRepository interface 의 package 는 org.springframework.data.jpa.repository 가 아니라, org.springframework.data.repository 이다.
CrudRepository 는 다시 Repository 를 상속한다.

![](/image/spring-data-jpa-crud-repository.png)


Repository interface 의 package 는 org.springframework.data.jpa.repository 가 아니라, org.springframework.data.repository 이다.

![](/image/spring-data-jpa-parent-repository.png)

정리하면 다음 계층구조이다.

![](/image/spring-data-jpa-inheritance.png)

## 스프링 데이터 JPA 가 사용하는 구현체

스프링 데이터 JPA 가 제공하는 공통 인터페이스는 org.springframework.data.jpa.repository.support.SimpleJpaRepository 클래스가 구현한다.

```java
@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID extends Serializable> implements JpaRepository<T, ID>, JpaSpecificationExecutor<T> {
  ...
}
```

직접 확인해보자.

의존성을 다음과 같이 추가한다.

```groovy
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    testImplementation 'com.h2database:h2'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

그리고 Repository 를 정의한다.

```java
import org.springframework.data.jpa.repository.JpaRepository;

public interface TestRepository extends JpaRepository<Test, Long> {
  
}
```

TestRepository 룰 출력해보자.

```java
@SpringBootTest
class DemoSpringDataJpaApplicationTests {

  @Autowired
  private TestRepository testRepository;

  @Test
  void test() {
    System.out.println(testRepository);
  }
}
```

결과는, SimpleJpaRepository 클래스 명이 출력된다.
repository interface 를 정의했을 뿐인데, 구체 클래스가 주입이 되었다.

![](/image/spring-data-simpe-jpa-repository.png)

## Query Method

스프링 데이터 JPA 가 제공하는 쿼리 메소드 기능은 크게 세 가지이다.

1. 메소드 이름으로 쿼리 생성
2. 메소드 이름으로 JPA NamedQuery 호출
3. @Query 어노테이션을 사용해서 리포지토리 인터페이스에 쿼리 직접 정의

### 메소드 이름으로 쿼리 생성

```java
public interface MemberRepository extends Repository<Member, Long> {
  List<Member> findByEmailAndName (String email, String name);
}
```

findByEmailAndName(…) 를 실행하면 스프링 데이터 JPA 는 메소드 이름을 분석해서 다음 JPQL 을 생성하고 실행한다.

```sql
select m from Member m where m.email = ?1 and m.name =?2
```

### JPA NamedQuery

스프링 데이터 JPA 는 메소드 이름으로 JPA Named 쿼리를 호출하는 기능을 제공합니다.

```java
@Entity
@NamedQuery{
  name = "Member.findByUsername",
  query = "select m from Member m where m.username = :username"
}
public class Member{
  ...
}
```

이렇게 정의한 Named 쿼리를 다음과 같이 호출한다.

```java
//JPA를 직접 사용해서 Named Query 호출
public class MemberRepository{
  public List<Member> findByUserName(String username){
    ...
    List<Member> resultList =
      	em.createNaemdQuery("Member.findByUsername", Member.class)
      	  .setParameter("username", "회원1")
          .getResultList();
  }
}

// 스프링 데이터 JPA 로 호출
public interface MemberRepository extends JpaRepository <Member, Long> {
  List <Member> findByUserName(@Param("username") String username);
}
```

스프링 데이터 JPA 로 호출하는 경우, "도메인 클래스.메소드이름" 으로 Named Query 를 찾아서 실행한다. 
위 예제는, Member.findByUsername 이라는 Named Query 를 실행한다. 
만약, Named Query 가 없으면 메소드 이름으로 쿼리 생성 전략을 사용한다.

### @Query, 리포지토리 메소드에 쿼리 정의

실행할 메소드에 직접 정적 쿼리를 작성하므로, 이름 없는 Named Query 라고 할 수 있다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
  @Query("select m from Member m where m.username = ?1")
  Member findByUserName(String username);
}
```

## 파라미터 바인딩

스프링 데이터 JPA 는 위치 기반 파라미터 바인딩과 이름 기반 파라미터 바인딩을 모두 지원한다. 
다음은 이름 기반 파라미터 바인딩 예제이다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
  @Query("select m from Member m where m.username = :name")
  Member findByUserName(@Param("name") String username);
}
```

## 벌크성 수정 쿼리

```java
// JPA 를 사용한 벌크성 수정 쿼리
int bulkPriceUp(String stockAmout){
  ...
  String sqlString = "update Product p set p.price = p.price * 1.1 where p.stockAmout < :stockAmout";
      
  int resultCount = em.createQuery(sqlString)
        .setParameter("stockAmout", stockAmout)
        .executeUpdate();
}

// 스프링 데이터 JPA 를 사용한 벌크성 수정 쿼리
@Modifying
@Query("update Product p set p.price = p.price * 1.1 where p.stockAmout < :stockAmout")
int bulkPriceUp(@Param("stockAmout") String stockAmout);
```

## 반환 타입

결과가 한건 이상이면 컬렉션 인터페이스, 단건이면 반환 타입을 지정한다.

```java
List<Member> findByName (String name); // 컬렉션
Member findByEmail (String email); // 단건
```

## 페이징과 정렬

쿼리 메소드에 페이징과 정렬 기능을 사용할 수 있도록 두 가지 파라미터를 제공한다.

1. org.springframework.data.domain.sort
2. org.springframework.data.domain.Pageable

파라미터에 Pageable 을 사용하면, 반환타입으로 List 나 org.springframework.data.domain.Page 를 사용할 수 있다.
반환 타입으로 Page 를 사용하면 검색된 전체 데이터 건수를 조회하는 count 쿼리를 추가로 호출할 수 있다.

Pageable 은 인터페이스이다.
실제 사용할 때는 이를 구현한 PageRequest 객체를 사용한다.

```java
// 페이징과 정렬 사용 예제
Page<Member> findByName(String name, Pageable pageable);
List<Member> findByName(String name, Pageable pageable);
List<Member> findByName(String name, Sort sort);

// Page 사용 예제 정의 코드
public interface MemberRepository extends Repository<Member, Long> {
  Page<Member> findByNameStartingWith(String name, Pageable pageable);
}

// Page 사용 예제 실행 코드
PageRequest pageRequest = new PageRequest(0, 10, new Sort(Direction.DESC, "name"));
Page<Member> result = memberRepository.findByNameStartingWith("김", pageRequest);

List<Member> members = result.getContent();
int totalPage = result.getTotalPages();
boolean hasNextPage = result.hasNextPage();
```

## Web 확장

스프링 데이터 프로젝트는 스프링 MVC 에서 사용할 수 있는 기능을 제공한다.

1. 도메인 클래스 컨버터 : HTTP 파라미터로 넘어온 엔티티의 아이디로 엔티티 객체를 찾아서 바인딩해준다.
2. 페이징과 정렬 기능

다음과 같이 설졍하면, 도메인 클래스 컨버터와 페이징과 정렬을 위한 HandlerMethodArgumentResolver 가 스프링 빈으로 등록된다.

```java
@Configuration
@EnableWebMvc
@EnableSpringDataWebSupport
public class WebAppConfig{
  ...
}
```
---
자바 ORM 표준 프로그래밍 <김영한>