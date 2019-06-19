---
layout: post
title:  "[자바 ORM 표준 JPA 프로그래밍] 14장_컬렉션과 부가 기능"
date:   2019-06-19
categories: JPA
---

이 장에서 다루는 내용입니다.

- 컬렉션
- 컨버터
  - 엔티티의 데이터를 변환해서 데이터베이스에 저장
- 리스너
  - 엔티티에서 발생한 이벤트 처리
- 엔티티 그래프
  - 엔티티를 조회할 때 연관된 엔티티를 선택해서 같이 조회

#### 14.1 컬렉션

JPA 는 자바에서 기본으로 제공하는 Collection / List / Set / Map 컬렉션을 지원합니다. 

##### 14.1.1 JPA 와 컬렉션

하이버네이트는 엔티티를 영속 상태로 만들 때 컬렉션 필드를 하이버네이트에서 준비한 컬렉션으로 감싸서 사용합니다.

```java
@Entity
public class Team {
  @Id
  private String id;
  
  @OneToMany
  @JoinColumn
  private Collection<Member> members = new ArrayList<Member>();
}

Team team = new Team();
System.out.println("before Persist = " + team.getMembers().getClass());
em.persist(parent);
System.out.println("after Persist = " + team.getMembers().getClass());
```

출력 결과는 

before Persist = class java.util.ArrayList

after Persist = class org.hibernate.collection.internal.PersistentBag

하이버네이트는 컬렉션을 효율적으로 관리하기 위해, 엔티티를 영속 상태로 만들 때 원본 컬렉션을 감싸고 있는 내장 컬렉션을 생성해서 이 내장 컬렉션을 사용하도록 참조를 변경합니다. 하이버네이트가 제공하는 내장 컬렉션은 원본 컬렉션을 감싸고 있어서 래퍼 컬렉션이라고 부릅니다.

##### 14.1.2 Collection, List

중복을 허용하는 컬렉션이고 PersistentBag 을 래퍼 컬렉션으로 사용합니다. 이 인터페이스는 ArrayList 로 초기화하면 됩니다.

##### 14.1.3 Set

중복을 허용하지 않는 컬렉션입니다. 하이버네이트는 PersistentSet 을 컬렉션 래퍼로 사용합니다. HashSet 으로 초기화하면 됩니다.

##### 14.1.4 List + @OrderColumn

List 인터페이스에 @OrderColumn 을 추가하면 순서가 있는 특수한 컬렉션으로 인식합니다. 순서가 있다는 의미는 데이터베이스에 순서 값을 저장해서 조회할 때 사용한다는 의미입니다. 하이버네이트는 내부 컬렉션인 PersistentList 를 사용합니다. 

##### 14.1.5 @OrderBy

데이터베이스의 ORDER BY 절을 사용해서 컬렉션을 정렬합니다. 따라서, 순서용 칼럼을 매핑하지 않아도 됩니다. 그리고 @OrderBy 는 모든 컬렉션에서 사용할 수 있습니다. 

#### 14.2 @Converter

엔티티의 데이터를 변환해서 데이터베이스에 저장합니다. 

#### 14.3 리스너

엔티티의 생명주기에 따른 이벤트를 처리할 수 있습니다.

##### 14.3.2 이벤트 적용 위치

- 엔티티에 직접 적용
  - 엔티티에 이벤트가 발생할 때마다 어노테이션으로 지정한 메소드가 실행됩니다.
- 별도의 리스너 등록
  - 대상 엔티티를 파라미터로 받을 수 있습니다. 
- 기본 리스너 사용 
  - 모든 엔티티의 이벤트를 처리하려면 META-INF/orm.xml 에 default 리스너로 등록하면 됩니다. 

#### 14.4 엔티티 그래프

엔티티를 조회할 때 연관된 엔티티를 함께 조회하려면 글로벌 fetch 옵션을 FetchType.EAGER 로 설정하거나 JPQL 엣 ㅓ페치 조인을 사용합니다. 

엔티티 그래프 기능은 엔티티 조회 시점에 연관된 엔티티를 함께 조회하는 기능입니다. 엔티티 그래프는 정적으로 정의하는 Named 엔티티 그래프와 동적으로 정의하는 엔티티 그래프가 있습니다.