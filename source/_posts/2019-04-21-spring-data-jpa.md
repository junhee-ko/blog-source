---
layout: post
title:  "[자바 ORM 표준 JPA 프로그래밍] 12장_스프링 데이터 JPA"
date:   2019-04-21
categories: JPA
---

데이터 접근 계층 (Data Access Layer) 는 CRUD 로 불리는 등록, 수정, 삭제, 조회 코드를 반복해서 개발해야 합니다. JPA 를 사용해서 데이터 접근 계층을 개발할 때도 문제가 발생합니다.

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

위 코드를 보면, 회원 리포지토리와 상품 리포지토리가 하는 일이 비슷합니다. 이 문제를 해결하려면 제네릭과 상속을 적절히 사용해서 공통 부분을 처리하는 부모 클래스를 만들면 됩니다. 이것을 보통 GenericDAO 라고 합니다. 하지만 이것은, 공통 기능을 구렿낳ㄴ 부모 클래스에 종속되고 구현 클래스 상속이 가지는 단점이 있습니다. 

#### 12.1 스프링 데이터 JPA 소개

스프링 데이터 JPA 는 스프링 프레임워크에서 JPA 를 편리하게 사용할수 있도록 지원하는 프로젝트입니다. 이 프로젝트는 데이터 접근 계층을 개발할 때 지루하게 반복되는 CRUD 문제를 세련된 방법으로 해결합니다. 데이터 접근 계층을 개발할 때 구현 클래스 없이 인터페이스만 작성해도 개발을 완료할 수있습니다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
  Member findByUsername(String username);
}

public interface ItemRepository extends JpaRepository<Item, Long> {
}
```

회원과 상품 리포지토리 구현체는 애플리케이션 실행 시점에 스프링 데이터 JPA가 생성해서 주입해줍니다. 즉, 개발자가 직접 구현체를 개발하지 않아도 됩니다.
일반적인 CRUD 메소드는 JpaRepository 인터페이스가 공통으로 제공하지만, MemberRepository.findByUsername(...) 처럼 직접 작성한 공통으로 처리할 수 없는 메소드는 스프링 데이터 JPA 가 메소드 이름을 분석해서 JPQL 을 실행합니다.

##### 12.1.1 스프링 데이터 프로젝트

![](/image/spring-data-jpa.png)

스프링 데이터 JPA 프로젝트는 JPA에 특화된 기능을 제공합니다. 스프링 프레임워크 + JPA 를 사용한다면, 스프링 데이터 JPA 를 추천합니다.

#### 12.2 스프링 데이터 JPA 설정

- 필요 라이브러리
- 환경 설정

#### 12.3 공통 인터페이스 기능

```java
// JpaRepository 공통 기능 인터페이스
public interface JpaRepository<T, ID extends Serializable> extends PagingAndSortingRepository<T, ID> {
  ...
}

// JpaRepository 를 사용하는 인터페이스
public interface MemberRepository extends JpaRepository<Member, Long> {
}
```

JpaRepsitory 인터페이스의 계층 구조는 다음과 같습니다.

![](/image/spring-data-jpa-inheritance.png)

#### 12.4 쿼레 메소드 기능

스프링 데이터 JPA 가 제공하는 쿼리 메소드 기능은 크게 3가지입니다.

- 메소드 이름으로 쿼리 생성
- 메소드 이름으로 JPA NamedQuery 호출
- @Query 어노테이션을 사용해서 리포지토리 인터페이스에 쿼리 직접 정의

##### 12.4.1 메소드 이름으로 쿼리 생성

```java
public interface MemberRepository extends Repository<Member, Long> {
  List<Member> findByEmailAndName (String email, String name);
}
```

findByEmailAndName(…) 를 실행하면 스프링 데이터 JPA 는 메소드 이름을 분석해서 다음 JPQL을 생성하고 실행합니다.

```sql
select m from Member m where m.email = ?1 and m.name =?2
```

##### 12.4.2 JPA NamedQuery

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

이렇게 정의한 Named 쿼리를 다음과 같이 호출합니다.

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

스프링 데이터 JPA 로 호출하는 경우, "도메인 클래스.메소드이름" 으로 Named Query 를 찾아서 실행합니다. 위 예제는, Member.findByUsername 이라는 Named Query 를 실행합니다. 만약, Named Query 가 없으면 메소드 이름으로 쿼리 생성 전략을 사용합니다.

##### 12.4.3 @Query, 리포지토리 메소드에 쿼리 정의

실행할 메소드에 직접 정적 쿼리를 작성하므로, 이름 없는 Named Query 라고 할 수 있습니다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
  @Query("select m from Member m where m.username = ?1")
  Member findByUserName(String username);
}
```

##### 12.4.4 파라미터 바인딩

스프링 데이터 JPA 는 위치 기반 파라미터 바인딩과 이름 기반 파라미터 바인딩을 모두 지원합니다. 다음은 이름 기반 파라미터 바인딩 예제입니다.

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
  @Query("select m from Member m where m.username = :name")
  Member findByUserName(@Param("name") String username);
}
```

##### 12.4.5 벌크성 수정 쿼리

```java
// JPA 를 사용한 벌크성 수정 쿼리
int bulkPriceUp(String stockAmout){
  ...
  String qlString = "update Product p set p.price = p.price * 1.1 where 
    p.stockAmout < :stockAmout";
      
  int resultCount = em.createQuery(qlString)
      						    .setParameter("stockAmout", stockAmout)
                      .executeUpdate();
}

// 스프링 데이터 JPA 를 사용한 벌크성 수정 쿼리
@Modifying
@Query("update Product p set p.price = p.price * 1.1 where 
    p.stockAmout < :stockAmout")
int bulkPriceUp(@Param("stockAmout") String stockAmout);
```

##### 12.4.6 반환 타입

결과가 한건 이상이면 컬렉션 인터페이스, 단건이면 반환 타입을 지정합니다.

```java
List<Member> findByName (String name); //컬렉션
Member findByEmail (String email); //단건
```

##### 12.4.7 페이징과 정렬

쿼리 메소드에 페이징과 정렬 기능을 사용할 수 있도록 2가지 파라미터를 제공합니다.

- org.springframework.data.domain.sort
- org.springframework.data.domain.Pageable
  - 파라미터에 Pageable 을 사용하면, 반환타입으로 List 나 org.springframework.data.domain.Page 사용 가능
  - 반환 타입으로 Page 사용하면 검색된 전체 데이터 건수 조회하는 count 쿼리 추가로 호출

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

Pageable 은 인터페이스입니다. 실제 사용할 때는 이를 구현한 PageRequest 객체를 사용합니다.

##### 12.4.8 힌트

SQL 힌트가 아니라 JPA 구현체에게 제공하는 힌트입니다.

```java
@QueryHints(value = {@QueryHint(name = "org.hibernate.readOnly",
                                value = "true")}, forCounting = true)
Page<Member> findByName (String name, Pagable pageable);
```

##### 12.4.9 Lock

#### 12.5 명세

도메인 주도 설계에서 명세라는 개념을 소개하는데, 스프링 데이터 JPA 는 JPA Criteria 로 이 개념을 사용할 수 있습니다.
명세를 이해하기 위한 핵심 단어는 술어입니다. 이것은 단순히 참이나 거짓으로 평가됩니다. 스프링 데이터 JPA 는 이 술어를 org.springframework.data.jpa.domain.Specification 클래스로 정의했습니다. Specification 은 컴포지트 패턴으로 구성되어 있어서 여러 Specification 으로 조합할 수 있습니다.즉, 다양한 검색 조건을 조립해서 새로운 검색 조건을 쉽게 만들 수 있습니다.

```java
// JpaSpecificationExecutor 상속
pubilc interface OrderRepsitroy extends JpaRepository<Order, Long>, JpaSpecificationExecutor<Order>{
}

// JpaSpecificationExecutor 인터페이스
public interface JpaSpecificationExecutor<T> {
  T findOne (Specification<T> spec);
  ...
}

// 명세 사용 코드
// Specification 은 명세들을 조립할 수 있도록 도와주는 클래스인데, 
// where(), and(), or(), not() 메소드를 제공
public List<Order> findOrders (String name) {
  List<Order> result = orderRepository.findAll(
  				where(memberName(name)).and(isOrderStatus())
  );
  return result;
}
```

#### 12.6 사용자 정의 리포지토리 구현

스프링 데이터 JPA 로 리포지토리를 개발하면 인터페이스만 정의하고 구현체는 만들지 않습니다. 하지만, 다양한 이유로 메소드를 직접 구현해야 할 때도 있습니다.

```java
// 사용자 정의 인터페이스
public interface MemberRepositoryCustom{
  public List<Member> findMemberCustom();
}

// 사용자 정의 구현 클래스
// 클래스 이름 짓는 규칙 : 리포지토리 인터페이스 이름 + Impl
// 이렇게 하면 스프링 데이터 JPA 가 사용자 정의 구현 클래스로 인식
public class MemberRepositoryImpl implements MemberRepositoryCustom{
  @Override
  public List<Member> findMemberCustom(){
    // 사용자 정의 구현
    // ...
  }  
}

// 사용자 정의 인터페이스 상속
public interface MemberRepository extends JpaRepository<Member, Long>, MemberRepositoryCustom{
}
```

#### 12.7 Web 확장

스프링 데이터 프로젝트는 스프링 MVC 에서 사용할 수 있는 기능을 제공합니다.

- 식별자로 도메인 클래스를 바로 바인딩 해주는 도메인 클래스 컨버터 기능
- 페이징과 정렬 기능

##### 12.7.1 설정

```java
@Configuration
@EnableWebMvc
@EnableSpringDataWebSupport
public class WebAppConfig{
  ...
}
```

설정을 완료하면, 도메인 클래스 컨버터와 페이징과 정렬을 위한 HandlerMethodArgumentResolver 가 스프링 빈으로 등록됩니다.

##### 12.7.2 도메인 클래스 컨버터 기능

도메인 클래스 컨버터는 HTTP 파라미터로 넘어온 엔티티의 아이디로 엔티티 객체를 찾아서 바인딩해줍니다.

##### 12.7.3 페이징과 정렬 기능

스프링 데이터가 제공하는 페이징과 정렬 기능을 스프링 MVC 에서 편리하게 사용할 수 있도록 HandlerMethodArgumentResolver 를 제공합니다. 

#### 12.8 스프링 데이터 JPA 가 사용하는 구현체

스프링 데이터 JPA 가 제공하는 공통 인터페이스는 org.springframework.data.jpa.repository.support.SimpleJpaRepository 클래스가 구현합니다.

```java
@Repository
@Transactional(readOnly = true)
public class SimpleJpaRepository<T, ID extends Serializable> implements JpaRepository<T, ID>, JpaSpecificationExecutor<T> {
  ...
}
```