---
layout: post
title:  "[자바 ORM 표준 JPA 프로그래밍] 3장_영속성 관리"
date:   2019-03-20
categories: JPA
---

이번 포스팅에서는 Mapping 한 Entity 를 Entity Manager 를 통해 어떻게 사용하는지 정리합니다.

#### Entity Manager Factory, Entity Manager

엔티티 메니저 펙토리는 한 개만 만들어서 애플리케이션 전체에서 공유합니다. 엔티티 메니저 펙토리는 서도 다른 스레드 간에 공유해도 되지만, 엔티티 메니저는 여러 스레드가 동시에 접근하면 동시성 문제가 발생하므로 스레드 간에 절대 공유하면 안됩니다.

#### Persistence Context

엔티티를 여구 저장하는 환경입니다. 엔티티 메니저로 엔티티를 저장하거나 조회하면 엔티티 메니저는 영속성 컨텍스트에 엔티티를 보관하고 관리합니다.

#### 엔티티의 생명주기

4가지 상태가 있습니다.

- 비영속
  - 영속성 컨텍스트와 관계 없는 상태 
- 영속
  - 영속성 컨텍스트에 저장된 상태
- 준영속
  - 영속성 컨텍스트에 저장되었다가 분리된 상태
- 삭제
  - 삭제된 상태

#### 영속성 컨텍스트의 특징

- 영속 상태는 식별자 값이 반드시 있어야합니다.
- 보통 Transaction 을 Commit 하는 순간 영속성 컨텍스트에 저장된 엔티티를 데이터베이스에 반영합니다. (Flush)
- 1차 캐쉬 / 동일성 보장 / 쓰기 지연 / 변경 감지 / 지연 로딩

#### 엔티티 조회

영속성 컨텍스트는 내부에 1차 캐시를 가지고 있습니다. 쉽게 이야기해, Map 처럼 키는 @Id로 매핑한 식별자이고 값은 엔티티 인스턴스입니다. 1차 캐시에 저장된 엔티티를 조회할 때 , 1차 캐시에 엔티티가 있으면 데이터베이스를 조회하지 않고 메모리에 있는 1차 캐시에서 엔티티를 조회합니다. 만약, 1차 캐시에 없으면 엔티티 메니저는 데이터베이스를 조회해서 엔티티를 생성하고 1차 캐시에 저장한 후에 영속 상태의 엔티티를 반환합니다. 

```java
entityManger.find(Member.class, "member1");
```

#### 영속 엔티티의 동일성 보장

영속성 컨텍스트는 성능상 이점과 엔티티의 동일성을 보장합니다. 따라서, 다음 코드의 결과는 참입니다.

```java
Member a = em.find(Member.class, "member1");
Member b = em.find(Member.class, "member1");

System.out.println(a==b);
```

#### 엔티티 등록

엔티티 메니저는 트랜잭션을 커밋하기 직전까지 데이터베이스에 엔티티를 저장하지 않고 내부 쿼리 저장소에 INSERT SQL을 모아둡니다. 그리고 커밋할 때 모아둔 쿼리를 데이터베이스에 보냅니다. 이것을 Transactional Write Behind (쓰기 지연) 라고 합니다.

#### 엔티티 수정

엔티티의 변경사항을 데이터베이스에 자동으로 반영하는 기능을 Dirty Checking (변경 감지) 이라고 합니다. 순서는 다음과 같습니다.

1. 커밋하면 엔티티 메니저 내부에서 flush() 가 호출됩니다.
2. 엔티티와 스냅샷 (JPA는 엔티티를 영속성 컨텐스트에 보관할 때, 최초 상태를 복사해서 저장합니다 )을 비교해서 변경된 엔티티를 찾습니다.
3. 변경된 엔티티가 있으면 수정 쿼리를 생성해서 쓰기 지연 SQL 저장소에 보냅니다.
4. 데이터베이스 트랜잭션을 커밋합니다.

그리고 UPDATE SQL 을 생성할 때, 변경된 부분만 사용해서 동적으로 생성되는 것이 아니라, 엔티티의 모든 필드를 업데이트 합니다.

#### 엔티티 삭제

엔티티 등록과 비슷하게 삭제 쿼리를 쓰기 지연 SQL 저장소에 등록하고 트랜잭션을 커밋해서 flush()를 호출하면 데이터베이스에 삭제 쿼리를 전달합니다.

```java
Member a = em.find(Member.class, "member1");
em.remove(a);
```

#### 플러쉬

영속성 컨테르를 플러쉬하는 방법은 3가지입니다.

- em.flush() 직접 호출
- 트랜잭션 커밋 시 플러쉬 자동 호출
- JPQL 쿼리 실행시 플러쉬 자동 호출

#### Reference

자바 ORM 표준 프로그래밍 <김영한>