---
layout: post
title:  "[자바 ORM 표준 JPA 프로그래밍] 4장_앤티티 매핑"
date:   2019-03-20
categories: JPA
---

엔티티와 테이블을 정확히 매핑하기 위해 JPA 는 다양한 어노테이션을 지원한다.

1. 객체와 테이블 매핑 : @Entity, @Table
2. 기본 키 매핑 :@Id
3. 필드와 컬럼 매핑 : @Column

## @Entity

JPA 를 사용해서 테이블을 매핑할 클래스는 @Entity 를 필수로 붙여야한다. 
그리고, @Entity 적용시 기본 생성자는 필수이다.

## @Table

엔티티와 매핑할 테이블을 지정한다.

## 데이터베이스 스키마 자동 생성

JPA 는 클래스의 매핑 정보와 Database Dialect 을 사용해서 데이터베이스 스키마를 생성한다. 
다음과 같이 설정하면 기존 테이블은 삭제하고 새로 생성한다. ( DROP + CREATE )

```java
spring.jpa.hibernate.ddl-auto=create
```

create-drop / update / validate / none 옵션이 있다.

## Primary Key 매핑

JPA가 제공하는 데이터베이스 기본 키 생성 전략은 다음과 같습니다.

1. 직접 할당 
   애플리케이션에서 직접 할당한다.
2. 자동 생성 
   IDENTITY : 기본 키 생성을 데이터베이스에 위임한다. 
   SEQUENCE : 데이터베이스 시퀀스를 사용해 기본 키를 할당한다. 
   TABLE : 키 생성 테이블을 사용합한다.

## @Column

객체 필드를 테이블 칼럼에 매핑한다.
속성 중에 name, nullable 이 주로 사용된다.

---
자바 ORM 표준 프로그래밍 <김영한>