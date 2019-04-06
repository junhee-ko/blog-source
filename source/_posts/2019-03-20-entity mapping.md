---
layout: post
title:  "[자바 ORM 표준 JPA 프로그래밍] 4장_앤티티 매핑"
date:   2019-03-20
categories: JPA
---

엔티티와 테이블을 정확히 매핑하기 위해 JPA는 다양한 어노테이션을 지원합니다. 이번 포스팅에서는 다음 매핑에 대해 다룹니다.

- 객체와 테이블 매핑
  - @Entity, @Table
- 기본 키 매핑
  - @Id
- 필드와 컬럼 매핑
  - @Column

#### @Entity

JPA를 사용해서 테이블을 매핑할 클래스는 @Entity 를 필수로 붙여야합니다. @Entity 적용시 기본 생성자는 필수입니다.

#### @Table

엔티티와 매핑할 테이블을 지정합니다. 

#### 데이터베이스 스키마 자동 생성

JPA는 클래스의 매핑 정보와 데이터베이스 방언을 사용해서 데이터베이스 스키마를 생성합니다. 다음과 같이 설정하면 기존 테이블을 삭제하고 새로 생성합니다 (DROP+CREATE).  create-drop / update / validate / none 옵션이 있습니다.

```java
spring.jpa.hibernate.ddl-auto=create
```

#### 기본 키 (Primary Key) 매핑

JPA가 제공하는 데이터베이스 기본 키 생성 전략은 다음과 같습니다.

- 직접 할당
  - 애플리케이션에서 직접 할당합니다.
- 자동 생성
  - IDENTITY
    - 기본 키 생성을 데이터베이스에 위임합니다.
  - SEQUENCE
    - 데이터베이스 시퀀스를 사용해 기본 키를 할당합니다.
  - TABLE
    - 키 생성 테이블을 사용합니다.

#### @Column

객체 필드를 테이블 칼럼에 매핑합니다. 속성 중에 name, nullable 이 주로 사용됩니다.

#### Reference

자바 ORM 표준 프로그래밍 <김영한>