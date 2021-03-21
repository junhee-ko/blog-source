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

## 배치 처리

수백만 건의 데이터를 배치 처리할 때, 엔티티를 계속 조회하면 영속성 컨텍스트에 많은 엔티티가 쌓인다.
그러면, 메모리 부족 오류가 발생할 것이다. 그래서, 적절한 단위로 영속성 컨텍스트를 초기화해아한다.

### 등록 배치

100만건의 엔티티를 DB 에 저장한다고 해보자.
엔티티 100 건을 저장할 때마다 em.flush() 를 호출하고, em.clear() 할 수 있다.

### 수정 배치

100만건의 데이터를 조회해서 수정한다고 해보자. 한 번에 메모리에 올려둘 수 없어서,

1. paging 
   paging 처리 방법을 이용할 수 있다. 즉, 한 번에 100 건씩 페이징 쿼리로 조회하면서 엔티티를 수정하고 영속성 컨텍스트를 flush 하고 clear 한다.

2. cursor 
   cursor 를 이용할 수 있다. 하이버네이트는 scroll 이라는 이름으로 JDBC cursor 를 지원한다.

## 트랜잭션을 지원하는 쓰기 지연

```sql
insert(member1); // INSERT INTO...
insert(member2); // INSERT INTO...
insert(member3); // INSERT INTO...
insert(member4); // INSERT INTO...
insert(member5); // INSERT INTO...

commit();
```

네트워크 호출은 비용이 많이 드는 작업이다.
위의 경우에, SQL 을 직접 다루면 한 번의 커밋과 다섯 번의 insert SQL 로, 총 여섯 번의 데이터베이스 통신을 한다.
최적화 화려면 어떻게 해야할까 ? insert SQL 을 모아서 한 번에 데이터베이스에 보내면 된다. 
JPA 는 flush 기능으로 이것이 가능하다.

### SQL Batch

```sql
em.persist(new Member()); // 1
em.persist(new Member()); // 2
em.persist(new Member()); // 3
em.persist(new Child());  // 4
em.persist(new Member()); // 5
em.persist(new Member()); // 6
em.persist(new Member()); // 7
```

하이버네이트에서는, hibernate.jdbc.batch_size 속성의 값에 50 을 주면, 최대 50 건씩 모아서 SQL 배치를 실행한다. 그런데 주의할 점은, 같은 SQL 일 때만 유효하다. 
예를 들어 위의 경우에, 1-3 을 모아서 SQL 배치 실행, 4 를 실행, 5-7 을 모아서 SQL 실행한다.

### 식별자 생성 전략 IDENTITY

엔티티의 식별자 생성 전략이 IDENTITY 이면, 쓰기 지연을 활용한 성능 최적화를 할 수 없다. 
왜냐하면, 엔티티가 영속 상태가 되려면 식별자가 필요한데, 식별자를 구하려면 데이터베이스에 저장해야 구할 수 있어서 em.persist() 를 호출하면 바로 insert SQL 이 데이터베이스에 전달된다.

## DB table row lock time 최소화

트랜잭션을 지원하는 쓰기 지연의 본질적인 장점은, DB table row lock time 최소화이다.
영속성 컨텍스를 flush 하기 전까지는 데이터베이스에 로우에 락을 걸지 않기 떄문이다.
다음 로직을 보자.

```sql
update(member); // UPDATE SQL 
logicA();
logicB();
commit();
```

JPA 를 사용하지 않고 SQL 을 직접 다루면, update(member); 를 호출할 때 UPDATE SQL 이 실행되면서 DB table row 에 lock 을 건다.
이 lock 은 commit 을 호출될 때까지 유지된다. 그러면, 현재 수정 중인 데이터를 수정하려는 다른 트랜잭션은 lock 이 풀릴 때까지 기다려야한다.

JPA 는 commit() 에서 UPDATE SQL 을 실행하고 바로 트랜잭션을 커밋한다. 즉, 데이터베이스 락에 걸리는 시간을 최소화한다.
데이터베이스 락에 걸리는 시간을 최소화하면 뭐가 좋을까? 동시에 더 많은 트랜잭션을 처리할 수 있다.

---
자바 ORM 표준 프로그래밍 <김영한>