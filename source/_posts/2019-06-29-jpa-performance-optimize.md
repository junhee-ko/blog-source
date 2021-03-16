---
layout: post 
title:  "[자바 ORM 표준 JPA 프로그래밍] 15장_고급 주제와 성능 최적화"
date:   2019-06-29
categories: JPA
---

JPA 를 사용하며 주의할 점과, 성능 최적화 내용을 정리한다. 

## 트랜잭션 롤백

트랜잭션이 롤백된 영속성 컨텍스트를 그대로 사용하는 것은 위험하다. 다음 경우를 보자.

1. 엔티티를 조회해서 수정하는 중에 문제가 발생했다.
2. 트랜잭션이 롤백된다.
3. 데이터베이스의 데이터는 원래대로 복구된다.
4. 수정된 객체는 영속성 컨텍스트에 그대로 남아있다.

그럼 어떻게 해야할까 ? 
1. 영속성 컨텍스트를 새로 생성해서 사용하거나
2. EntityManager.clear() 로 영속성 컨텍스트를 초기화해야한다.

스프링에서는,

1. 트랜잭션당 영속성 컨텍스트의 경우 : 트랜잭션을 롤백하면서 영속성 컨텍스트도 함께 종료
2. OSIV 인 경우 : 트랜잭션을 롤백하면서 영속성 컨텍스트를 초기화

## 엔티티 비교

영속성 컨텍스트가 같을 때는, 다음 세 조건을 만족한다.

1. identical : == 비교가 같음
2. equivalent : equals() 비교가 같음
3. DB equivalent : @Id 인 DB 식별자가 같음

영속성 컨텍스트가 다를 때는, identical 비교에 실패한다.

## N+1 문제

코드보 보자.

```java
@Entity
public class Member {
  @Id @GeneratedValue
  private Long id;
  @OneToMany(mappedBy = "member", fetch = FetchType.EAGER)
  private List<Order> orders = new ArrayList<>();
}

@Entity
@Table(name = "ORDERS")
public class Order {
  @Id @GeneratedValue
  private Long id;
  @ManyToOne
  private Member member;
}
```

위와 같은 경우에, Member 하나를 다음 처럼 조회하면 즉시 로딩으로 설정한 주문 정보도 같이 조회한다.

```java
em.find(Member.class, id);
```

실행되는 SQL 은 다음과 같다.

```sql
SELECT M.*, O.*
FROM MEMBER M
OUTER JOIN ORDERS O ON M.ID=O.MEMBER_ID
```

여기까지는 좋다. 문제는, 다음처럼, JPQL 을 사용할 때다.

```java
em.createQuery("select m from Member m", Member.class).getResultList();
```

실행되는 SQL 은 다음과 같다.

```sql
SELECT * FROM MEMBER
```

회원 엔티티와 연관된 주문 컬렉션이 즉시 로딩으로 설정되어 있다. 그래서, 즉시 로딩하려고 다음 SQL 을 추가로 실행한다.

```sql
SELECT * FROM ORDERS WHERE MEMBER_ID = ?
```

조회된 회원이 100명이면, 위와 같은 쿼리가 100번 추가적으로 실행되는 것이다.

### 지연 로딩과 N+1

결론부터 말하면, 지연 로딩의 경우에도 N+1 문제는 동일하게 발생한다.

지연로딩으로 설정하고 JPQL 로 회원들을 조회해보자. 지연 로딩이므로 회원들만 조회되기 때문에, N+1 문제는 여기서 발생하지 않는다.
하지만 다음 처럼, 회원이 100명이면 100명의 주문도 100번 조회된다. 이것이 문제다.

```java
for (Member m : members){
  m.getOrders().size();  
}
```

다음 처럼,

```sql
SELECT * FROM ORDERS WHERE MEMBER_ID = 1
SELECT * FROM ORDERS WHERE MEMBER_ID = 2
SELECT * FROM ORDERS WHERE MEMBER_ID = 3
...
```

### Fetch Join

N+1 문제를 해결할 수 있는 간단한 방법이다. fetch join 을 사용하면, 연관된 엔티티를 같이 조회한다.

fetch join 을 사용하는 JPQL 은 다음과 같다.

```sql
selectm from Member m join fetch m.orders
```

실행되는 SQL 은 다음과 같다.

```sql
SELECT M.*, O.*
FROM MEMBER M
INNER JOIN ORDERS O ON M.ID=O.MEMBER_ID
```

## 읽기 전용 쿼리 성능 최적화

영속성 컨텍스트는 변경 감지를 위해 스냅샷 인스턴스를 보관해서, 더 많은 메모리를 사용한다.
데이터를 단순히 조회만 하는 읽기 전용 쿼리의 경우에, 이와 같은 불필요한 메모리 사용이 필요없다.

### 스칼라 타입으로 조회

스칼라 타입으로 모든 타입 조회하면, 영속성 컨텍스트가 결과를 관리하지 않는다.

```sql
select o.id, o.name, o.price from Order o
```

### 읽기 전용 쿼리 힌트 사용

하이버네이트 전용 힌트인 org.hibernate.readOnly 를 사용하면, 영속성 컨텍스트는 스냅샷을 보관하지 않는다.

```java
TypedQuery<Order> query = em.createQuery("select o from Order o", Order.class);
query.setHint("org.hibernate.readOnly", true)
```

### 읽기 전용 트랜잭션 사용

다음 처럼 사용하면, 스프링 프레임워크가 하이버네이트 세션 (JPA Entity Manager 의 구현체) 의 플러쉬 모드를 MANUAL 로 설정한다.
그래서, 강제로 플러쉬를 호출하지 않는 한 플러쉬가 일어나지 않는다. 트랜잭션을 커밋해도 영속성 컨텍스트를 플러쉬 하지 않아서, 스냅샷 비교와 같은 무거운 로직이 수행되지 않는다.

```java
@Transactional(readOnly = true)
```

### 트랜잭션 밖에서 읽기

트랜잭션을 사용하지 않아서, 플러쉬가 일어나지 않아 조회 성능이 향상된다.

```java
@Transactional(propagation = Propagation.NOT_SUPPORTED)
```

---
자바 ORM 표준 프로그래밍 <김영한>