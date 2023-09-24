---
layout: post
title: Persistence Context
date: 2019-03-19
categories: JPA
---

JPA 의 영속성 컨텍스트를 정리한다.

## Entity Manager Factory, Entity Manager

엔티티 메니저는 엔티티를 관리하는 관리자로서, 엔티티를 저장/수정/삭제/조회 등 엔티티와 관련된 모든 일을 처리한다.

엔티티 메니저 펙토리는 만드는 비용은 크기 때문에, 한 개만 만들어서 애플리케이션 전체에서 공유한다.
반면에, 엔티티 메니저를 생성하는 비용은 거의 들지 않는다.

엔티티 메니저 펙토리는 서도 다른 스레드 간에 공유해도 된다.
반면에, 엔티티 메니저는 여러 스레드가 동시에 접근하면 동시성 문제가 발생하므로 스레드 간에 절대 공유하면 안된다.

엔티티 메니저는 데이터베이스 연결이 꼭 필요한 시점까지 connection 을 얻지 않는다.
보통 transaction 을 시작할 때 connection 을 얻는다.

hibernate 를 포함한 JPA 구현체들은 EntityManagerFactory 를 생성할 때, connection pool 을 생성한다.

## Persistence Context

영속성 컨텍스트란, 엔티티를 저장하는 환경이다. 
엔티티 메니저로 엔티티를 저장하거나 조회하면 엔티티 메니저는 영속성 컨텍스트에 엔티티를 보관하고 관리한다.
영속성 컨텍스트는, 엔티티 메니저를 생성할 때 하나 만들어지며 엔티티 메니저를 통해서 영속성 컨텍스트에 접근할 수 있고 관리할 수 있다.

```java
entityManger.persist(member);
```

persist() 메서드는 엔티티 메니저를 사용해서 회원 엔티티를 영속성 컨텍스트에 저장한다.

## Entity Lifecycle 

네 가지 상태가 있다.

1. 비영속 (New or Transient): 영속성 컨텍스트와 관계 없는 상태 
2. 영속 (Managed): 영속성 컨텍스트에 저장된 상태
3. 준영속 (Detached): 영속성 컨텍스트에 저장되었다가 분리된 상태
4. 삭제 (Removed): 삭제된 상태

### 비영속

엔티티 객체를 생성했지만, 아직 저장하지 않은 순수한 객체이다.

```java
Member m = new Member();
m.setId("member01");
m.setUsername("jko");
```

### 영속

엔티티 메니저를 통해서 엔티티를 영속성 컨텍스트에 저장한다. 
그러면, 영속성 컨텍스트가 관리하는 엔티티가 되며 영속 상태가 된다.

```java
em.persist(member);
```

### 준영속

영속성 컨텍스트가 관리하던 영속 상태의 엔티티를 영속성 컨텍스트가 관리하지 않으면, 준영속 상태가 된다.
다음 방법 중 하나로 준영속 상태로 맏들 수 있다.

```java
em.detatch(member); // or
em.close();         // or
em.clear();
```

### 삭제

엔티티를 영속성 컨텍스트와 데이터베이스에서 삭제한다.

```java
em.remove(member);
```

## 영속성 컨텍스트의 특징

1. 영속성 컨텍스트와 식별자 값: 영속 상태는 식별자 값이 반드시 있어야한다. ( @Id 로 테이블의 기본 키와 매핑한 값 )
2. 영속성 컨텍스트와 DB 저장: 보통 Transaction 을 Commit 하는 순간, 영속성 컨텍스트에 저장된 엔티티를 데이터베이스에 반영한다. (== Flush)
3. 장점: 1차 캐쉬 / 동일성 보장 / 쓰기 지연 / 변경 감지 / 지연 로딩

### 엔티티 조회

영속성 컨텍스트는 내부에 1차 캐시라고 불리는 캐시를 가지고 있는데, 영속 상태의 엔티티는 모두 1차 개쉬에 저장된다.
1차 캐시란, Map 처럼 키는 @Id 로 매핑한 식별자이고 값은 엔티티 인스턴스이다.

아래 코드를 실행하면, 1차 캐쉬에 회원 엔티티를 저장하고, 회원 엔티티는 아직 DB 에 저장되지 않는다.

```java
Member m = new Member();
m.setId("member01");
m.setUsername("jko");

em.persist(m);
```

이제, 엔티티를 조회해보자.

```java
entityManger.find(Member.class, "jko");
```

em.find() 를 호출하면, 1차 캐시에 엔티티가 있으면 데이터베이스를 조회하지 않고 메모리에 있는 1차 캐시에서 엔티티를 조회한다.
만약, 1차 캐시에 없으면 엔티티 메니저는 데이터베이스를 조회해서 엔티티를 생성하고 1차 캐시에 저장한 후에 영속 상태의 엔티티를 반환한다. 

#### 영속 엔티티의 동일성 보장

다음 코드로, 식별자가 같은 엔티티 인스턴스를 조회해서 비교해보자.

```java
Member a = em.find(Member.class, "jko");
Member b = em.find(Member.class, "jko");

System.out.println(a==b); // true
```

"em.find(Member.class, "jko")" 를 반복해서 호출해도 영속성 컨텍스트는 1차 캐쉬에 있는 같은 엔티티 인스턴스를 반환한다.
즉, 영속성 컨텍스트는 엔티티의 동일성 (identity) 를 보장한다.


### 엔티티 등록

엔티티 등록해보자.

```java
EntityTransaction transaction = em.getTransaction();
transaction.begin(); 

em.persist(memberA);
em.persist(memberB);    // 여기 까지 INSERT SQL 을 DB 에 보내지 않음

transaction.commit();   // 커밋하는 순간, INSERT SQL 을 DB 에 보냄
```

1. 엔티티 메니저는 트랜잭션을 커밋하기 직전까지 데이터베이스에 엔티티를 저장하지 않고 내부 쿼리 저장소에 INSERT SQL 을 모아둔다. 
2. 그리고 커밋할 때 모아둔 쿼리를 데이터베이스에 보낸다.

이것을 Transactional Write Behind (쓰기 지연) 라고 한다.

트랜잭션을 커밋하면, 엔티티 메니저는 먼저 영속성 컨텍스트를 플러쉬한다.
플러쉬란, 영속성 컨텍스트의 변경 내용을 DB 에 동기화하는 작업이다.
즉, 쓰기 지연 SQL 저장소에 모인 쿼리를 DB 에 보낸다.

이렇게 영속성 컨텍스트의 변경 내용을 DB 에 동기화한 후에, 실제 DB transaction commit 을 하게 된다.

### 엔티티 수정

엔티티를 수정해보자.

```java
EntityTransaction transaction = em.getTransaction();
transaction.begin(); 

Member m = em.find(Membser.class, "jko");

m.setUsername("junhee");
m.setAge(10);

// em.update(member); 이런 코드는 필요하지 않음 

transaction.commit();
```

엔티티를 수정할 때는, 엔티티를 조회해서 데이터만 변경하면 된다.
엔티티의 변경사항을 데이터베이스에 자동으로 반영하는 기능을 Dirty Checking (변경 감지) 라고 한다.

1. transaction 을 commit 하면 엔티티 메니저 내부에서 먼저 flush() 가 호출된다.
2. 엔티티와 스냅샷 (JPA 는 엔티티를 영속성 컨텐스트에 보관할 때, 최초 상태를 복사해서 저장) 을 비교해서 변경된 엔티티를 찾는다.
3. 변경된 엔티티가 있으면 수정 쿼리를 생성해서 쓰기 지연 SQL 저장소에 보낸다.
4. DB transaction 을 commit 한다.

변경 감지는 영속성 컨텍스트가 관리하는 영속 상태의 엔티티에만 적용된다. (== 비영속, 준영속 처럼 관리받지 못하는 엔티티는 값을 변경해도 DB 에 반영 X)

또한, UPDATE SQL 을 생성할 때, 변경된 부분만 사용해서 동적으로 생성되는 것이 아니라, 엔티티의 모든 필드를 업데이트한다.
필드가 많거나 저장되는 내용이 너무 크면 수정된 데이터만 사용해서 동적으로 UPDATE SQL 을 생성하는 전략을 선택할 수 있다.

```java
@Entity
@org.hibernate.annotatinos.DynamicUpate
public class Member {...}
```

### 엔티티 삭제

```java
Member m = em.find(Membser.class, "jko");   // 삭제하려면, 먼저 대상 엔티티를 조회해야한다.
em.remove(m);   // em.remove(m) 를 호출하는 순간 영속성 컨텍스트에서 m 은 제거된다.
```

엔티티 등록과 비슷하게, 삭제 쿼리를 쓰기 지연 SQL 저장소에 등록하고 트랜잭션을 커밋해서 flush()를 호출하면 데이터베이스에 삭제 쿼리를 전달한다.

## Flush

flush() 는 영속성 컨텍스트의 변경 내용을 DB 에 반영한다. flush() 를 실행하면,

1. 변경 감지가 동작해서 영속성 컨텍스트에 있는 모든 엔티티를 snapshot 과 비교해서 수정된 엔티티를 찾는다.
2. 수정된 엔티티는 수정 쿼리를 만들어 쓰기 지연 SQL 저장소에 등록한다.
3. 쓰기 지연 SQL 저장소의 쿼리를 DB 에 전송한다.

영속성 컨텍스를 flush 하는 방법은 세 가지이다.

1. em.flush() 직접 호출
2. transaction commit 시 자동 호출
3. JPQL 쿼리 실행시 자동 호출

JPQL 은 SQL 로 변환되어 DB 에서 엔티티를 조회하는데, 쿼리를 실행하기 직전에 영속성 컨텍스트를 flush 해서 변경 내용을 DB 에 반영한다.

```java
em.persist(memebrA);
em.persist(memebrB;
em.persist(memebrC);

query = em.createQuery("select m from Member m", Member.class);
List<Member> members = query.getResultList();   // memebrA, memebrB, memebrC 가 쿼리 결과에 포함된다.
```

flush 에 대해 중요한 점은, 영속성 컨텍스트에 보관된 엔티티를 지운다고 생각하면 안된다는 것이다.
flush 는, 영속성 컨텍스트의 변경 내용을 DB 에 동기화하는 것이다.

## 준영속

영속 상태의 엔티티가 영속성 컨텍스트에 분리된 것을 준영속 상태라고 한다.
그래서, 준영속 상태의 엔티티는 영속성 컨텍스트가 제공하는 기능을 사용할 수 없다.
영속 상태를 준영속 상태로 만드는 방법은,

1. 엔티티를 준영속 성태로 전환: detach()
2. 영속성 컨텍스트 초기화: em.clear()
3. 영속성 컨텍스트 종료: em.close()

### 엔티티를 준영속 성태로 전환: detach()

```java
// 비영속 상태
Member m = new Member();
m.setId("member01");
m.setUsername("jko");

// 영속 상태
em.persist(m);

// 준영속 상태
em.detach(m);   // 1차 캐쉬, 쓰기지연 SQL 저장소 모두 해당 엔티티를 관리하기 위한 모든 정보가 삭제

// commit
transaction.commit();
```

### 영속성 컨텍스트 초기화: em.clear()

영속성 컨텍스트의 모든 엔티티를 준영속 상태로 만드는 방법이다.

```java
// 영속 상태
Member m = em.find(Member.class, "jko");

// 초기화
em.clear();

// 준영속 상태 -> 변경 감지가 동작하지 않음
m.seetUsername("junhee"); 
```

### 영속성 컨텍스트 종료: em.close() 

영속성 컨텍스트를 종료하면, 해당 영속성 컨텍스트가 관리하던 영속 상태의 엔티티가 모두 준영속 상태가 된다.

### 준영속 상태의 특징

1. 비영속 상태에 가깝다: 영속성 컨텍스트가 제공하는 어떤 기능도 동작하지 않는다. (1차 캐쉬, 쓰기지연, 변경 감지, 지연 로딩)
2. 식별자 값을 가지고 있다: 이미 한 번 영속 상태였으로.

### 병합: merge()

준영속 상태의 엔티티를 다시 영속 상태로 변경하기 위해서는, 병합을 사용하면 된다.
merge() 는 준영속 상태의 엔티티를 받아서, 새로운 영속 상태의 엔티티를 반환한다.

```java
Member mergedMember = em.merge(member);
```

병합은, 

1. 파라미터로 넘어온 엔티의 식별자 값으로 영속성 컨텍스트를 조회하고, 없으면 DB 에서 조회한다.
2. DB 에도 없으면 새로운 엔티티를 생성해서 병합한다.


---
자바 ORM 표준 프로그래밍 <김영한>