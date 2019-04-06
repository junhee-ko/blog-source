---
layout: post
title:  "JPA Simple Test"
date:   2019-03-18
categories: JPA
---

이번 포스팅에서는 JPA를 이용해 객체를 Database 에 저장하는 Test를 하고자 합니다. Database는 Docker 를 통해 Postgres를 사용합니다.

#### Docker

Docker는 공식 홈페이지에서 쉽게 설치할 수 있습니다 : https://docs.docker.com/docker-for-mac/install/ 

Docker 를 설치하고 다음 명령어를 통해 Postgres Container를 띄웁니다.

```java
docker run -p 5432:5432 -e POSTGRES_PASSWORD=1234 -e POSTGRES_USER=jko -e POSTGRES_DB=springdata --name postgres_boot -d postgres
```

Postgres Bash에 붙습니다.

```java
docker exec -i -t postgres_boot bash
```

postgres 계정으로 springdata 데이터베이스에 접근합니다.

```java
su - postgres
psql springdata
```

Account 테이블을 생성합니다.

```java
CREATE TABLE ACCOUNT (id int, username varchar(255), password varchar(255));
```

#### 의존성 추가

JPA를 사용하기 위한 의존성과 Postgresql 을 사용하기 위한 의존성을 추가합니다.

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<dependency>
	<groupId>org.postgresql</groupId>
	<artifactId>postgresql</artifactId>
</dependency>
```

#### 설정

Database 연결을 위한 정보를 입력하고, Database Schema를 실행시마다 새로 생성되게 (ddl-auto=create) 설정합니다. 그리고, non_contextual_create=true 를 설정해야 어플리케이션 실행시 발생하는 postgres 에러를 해결할 수 있습니다. 

```java
spring.datasource.url=jdbc:postgresql://localhost:5432/springdata
spring.datasource.username=jko
spring.datasource.password=1234

spring.jpa.hibernate.ddl-auto=create
spring.jpa.properties.hibernate.jdbc.lob.non_contextual_creation=true
```

#### Account

Account Class를 다음과 같이 정의합니다. @Entity 에 이름을 따로 정하지 않으면 Class의 이름이 Table의 이름으로 매칭됩니다. 그리고, 각 필드 (id, username, password) 에는 @Column 이 생략되어 있습니다.

```java
package jko.jpatest;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
public class Account {

    @Id
    @GeneratedValue
    private Long id;

    private String username;

    private String password;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}
```

#### JpaRunner

Database 에 Insert Query 를 날리기 위해 JPA api 를 사용할 수도 있고, Hibernate api 를 사용할 수 도 있습니다.

```java
package jko.jpatest;

import org.hibernate.Session;
import org.springframework.boot.ApplicationArguments;
import org.springframework.boot.ApplicationRunner;
import org.springframework.stereotype.Component;
import org.springframework.transaction.annotation.Transactional;

import javax.persistence.EntityManager;
import javax.persistence.PersistenceContext;


@Component
@Transactional
public class JpaRunner implements ApplicationRunner {

    @PersistenceContext
    EntityManager entityManager;

    @Override
    public void run(ApplicationArguments args) throws Exception {

        Account account = new Account();
        account.setUsername("jko");
        account.setPassword("123");

//        1. jpa api
//        entityManager.persist(account);

//		2. hibernate api
        Session session = entityManager.unwrap(Session.class);
        session.save(account);
    }
}
```

#### Reference

https://www.inflearn.com/course/스프링-데이터-jpa