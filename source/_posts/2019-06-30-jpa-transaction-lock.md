---
layout: post
title:  "[자바 ORM 표준 JPA 프로그래밍] 16장_트랜잭션과 락,2차 캐시"
date:   2019-06-30
categories: JPA
---

## Transaction

트랜잭션이란 논리적인 작업의 단위이다. 이 트랜잭션은 ACID 를 보장해야한다.

1. Atomicity
   트랜잭션 내에서 실행한 작업들은 하나의 작업 처럼, 모두 성공하거나 모두 실패해야한다.
2. Consistency
   트랜잭션은 일관성 있는 데이터베이스 상태를 유지해야한다.
   예를 들면, 데이터베이스의 무결성 제약 조건을 항상 만족해야한다.
3. Isolation
   동시에 실행되는 트랜잭션은 서로 영향을 미치지 않아야한다.
4. Durability
   트랜잭션을 성공적으로 끝내면, 그 결과가 데이터베이스에 항상 기록되어야한다.

문제는 격리성이다. 트랜잭션간에 완벽하게 격리성을 보장하기 위해서는 어떻게 해야할까 ?
트랜잭션을 차례대로 실행해야한다. 그러면, 동시성이 처리 기능이 떨어진다.
그래서 트랜잭션 격리 수준이 등장한다.

## Isolation Level

격리 수준이 낮을 수록 더 많은 문제가 발생한다.
READ UNCOMMITTED, READ COMMITTED , REPEATABLE READ, SERIALIZABLE 으로 격리 수준이 높아진다.
애플리케이션은 대부분 동시성 처리가 중요하기 때문에, 데이터베이스들은 보통 READ COMMITTED 격리 수준이 기본이다. 

### READ UNCOMMITTED

커밋하지 않은 데이터를 읽을 수 있다. DIRTY READ 문제가 발생할 수 있다.

1. 트랜잭션 1 이 데이터를 수정하고 있다.
2. 트랜잭션 2 가 수정 중인 데이터를 조회한다.
3. 트랜잭션 1 이 롤백을 하게 되면 데이터 정합성에 문제가 생긴다.

### READ COMMITTED

커밋한 데이터만 읽을 수 있다. NON-REPEATABLE READ 문제가 발생할 수 있다.

1. 트랜잭션 1 이 회원 A 를 조회중이다.
2. 트랜잭션 2 가 회원 A 를 수정하고 커밋한다.
3. 트랜잭션 1 이 다시 회원 A 를 조회하면 수정된 데이터가 조회된다.

### REPEATABLE READ

한 번 조회한 데이터를 반복해서 조회해도 같읕 데이터가 조회된다. PHANTOM READ 문제가 발생할 수 있다.

1. 트랜잭션 1 이 10살 이하의 회원을 조회했다.
2. 트랜잭션 2 가 5살 회원을 추가하고 커밋했다. 
3. 트랜잭션 1 이 다시 10살 이하의 회원을 조회하면 회원 하나가 추가된 상태로 조회된다.

### SERIALIZABLE

가장 엄격한 격리수준이다. 동시성 처리 성능이 떨어진다.

## Optimistic Lock

트랜잭션 대부분은 충돌이 발생하지 않는다고 낙관적으로 가정하는 방법이다.
데이터베이스가 제공하는 락 기능을 사용하는 것이 아니라, Application 이 제공하는 락이다.
낙관적 락은 트랜잭션을 커밋하기 전까지 트랜잭션 충돌을 알 수 없다.

## Pessimistic Lock

트랜잭션의 충돌이 발생할 것이라 비관적으로 가정하는 방법이다. 그래서, 우선 락을 걸고 본다.
데이터베이스가 제공하는 락 기능을 사용한다.

## Second Lost Updates Problem

트랜잭션 만으로는 해결할 수 없는 두 번의 갱실 문제는, 다음과 같은 경우에 발생한다.

1. 사용자 A 와 B 가 동시에 같은 공지사항을 수정하고 있다.
2. 사용자 A 가 먼저 수정 완료 버튼을 눌렀다.
3. 사용자 B 가 수정 완료 버튼을 눌렀다.
4. 결과적으로, 사용자 B 의 수정사항만 반영된다. 

해결 방법으로는,

1. 마지막 커밋만 인정
2. 최초 커밋만 인정
3. 충돌하는 갱신 내용 병합

## @Version

JPA 가 제공하는 낙관적 락을 사용하기위해서는, @Version 으로 버전 관리 기능을 추가해야한다.
다음 처럼, 엔티티에 버전 관리용 필드를 추가하고 @Version 을 붙이면 된다.

```java
@Entity
public class Person {
   @Id
   private String id;
   private String title;
   @Version
   private Integer version;
}
```

엔티티를 수정할 때 마다, 버전이 하나씩 증가한다. 
그리고, 엔티티를 수정할 때 조회 시점의 버젼과 수정 시점의 버젼이 다르면 예외가 발생한다.
그래서, 버전 정보를 사용하면 최초 커밋만 인정된다.

다음 그림으로 보자.

1. Transaction 01 과 Transaction 02 가 조회한다.
2. Transaction 02 가 title 을 B 로 수정하고 커밋한다.
3. Version 이 2 로 증가한다.
4. Transaction 01 이 title 을 C 로 수정하고 커밋하려는 순간, 예외가 발생한다.

![](/image/jpa-lock-version.png)

## Version 비교 방법

JPA 는 버젼 정보를 어떻게 비교할까 ?

1. 엔티티를 수정하고 트랜잭션을 커밋한다.
2. 영속성 컨텍스트를 flush 하면서, 아래와 같은 update query 를 실행한다.
   ```sql
      UPDATE Person
      SET
         TITLE = ?
         VERSION = ? (version + 1 증가)
      WHERE
         ID = ? 
         AND VERSION = ? 
   ```
3. DB 의 버젼이 이미 증가해서 WHERE 문의 VERSION 값이 다르면 수정할 대상이 없기 때문에, JPA 가 예외를 던진다.

## JPA Lock

> JPA 에서 추천하는 전략 : READ COMMITTED 트랜잭션 격리 수준 + 낙관적 버전 관리 ( 두 번의 갱신 내역 분실 문제 예방 )

JPA 가 제공하는 Lock Option 은 javax.persistence.LockModeType 에 정의되어 있다.

## JPA 낙관적 락

JPA 낙관적 락을 사용하려면 @Version 이 있어야한다.
낙관적 락의 옵션을 하나씩 보자.

### NONE 

Lock Option 을 정의하지 않아도 엔티티에 @Version 을 붙인 필드가 있으면 적용된다.
조회 시점부터 수정 시점까지를 보장하여, Second Lost Updates Problem 을 에방한다.

### OPTIMISTIC

엔티티를 조회만 해도 버젼을 체크한다. 
즉, 한 번 조회한 엔티티는 트랜잭션을 종료할 때까지 다른 트랜잭션에서 변경하지 않음을 보장한다.
트랜잭션을 커밋할 때 버전 정보를 조회해서 (SELECT) 현재 엔티티의 버젼과 같은지 검증한다.
NONE Option 은 엔티티를 수정해야 버전 정보를 확인하지만, OPTIMISTIC Option 은 엔티티 수정 없이 조회만 해도 버젼을 확인한다. 

![](/image/jpa-lock-version-optimistic.png)

### OPTIMISTIC_FORCE_INCREMENT

데이터를 수정하지 않아도 트랜잭션을 커밋할 때 버젼 정보가 증가한다.

![](/image/jpa-lock-version-optimistic-force.png)

## JPA 비관적 락

JPA 비관적 락은 DB 트랜잭션 락 메커니즘에 의존한다.
비관적 락을 사용하면, 락을 획들할 때까지 트랜잭션이 대기한다.
무한정 대기할 수 없으므로 타임아웃을 줄 수 있다.

비관적 락의 옵션을 간단 보자.

### PESSIMISTIC_WRITE

비관적 락이면, 일반적으로 이 옵션이 많이 사용된다.
DB select for update 를 사용해서 락을 건다.
lock 이 걸린 로우는 다른 트랜잭션이 수정할 수 없다.

### PESSIMISTIC_READ

데이터를 읽기만 하고 수정하지 않는 용도로 락을 건다.

### PESSIMISTIC_FORCE_INCREMENT

비관적 락이지만 버젼 정보를 강제로 증가시킨다.

## 1차 캐쉬

영속성 컨텍스트 범위의 캐쉬이다.

![](/image/jpa-first-level-cache.png)

## 2차 캐쉬

애플리케이션 범위의 캐쉬이다. 애플리케이션이 종료될 때까지 캐쉬가 유지된다.
예를 들면, EHCACHE 를 2차 캐쉬로 사용할 수 있다. 

![](/image/jpa-second-level-cache.png)

2차 캐쉬는 캐쉬한 객체의 복사본을 만들어서 반환한다. 왜일까 ?
만약 캐쉬한 객체를 그대로 반환하면, 여러 곳에서 같은 객체를 동시에 수정하는 문제가 발생할 수 있다.
이 문제를 해결하기 위해, 락을 걸면 동시성이 떨어질 수 있다.
그래서, 객체를 복사해서 반환한다.



---
자바 ORM 표준 프로그래밍 <김영한>