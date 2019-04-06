---
layout: post
title:  "[자바 ORM 표준 JPA 프로그래밍] 7장_고급 매핑"
date:   2019-04-06
categories: JPA
---

이번 포스팅에서 다룰 고급 매핑은 다음과 같습니다.

- 상속 관계 매핑
- @MappedSupperclass
- 복합 키와 식별 관계 매핑
- 조인 테이블
- 엔티티 하나에 여러 테이블 매핑하기

#### 7.1 상속 관계 매핑

관계형 데이터베이스에는 상속이라는 개념이 없습니다. 대신에, Super-Type Sub-Type Relationship 이라는 모델링 기법이 객체의 상속 개념과 가장 유사합니다. ORM 의 상속 관계 매핑은 객체의 상속 구조와 데이터베이스의 Super-Type Sub-Type Relationship 을 매핑하는 것입니다.

Super-Type Sub-Type Relationship 논리 모델을 실제 물리 모델인 테이블로 구현할 때는 3가지 방법이 있습니다.

- 각각의 테이블로 변환
  - 조인 전략
- 통합 테이블로 변환
  - 단일 테이블 전략
- 서브타입 테이블로 변환
  - 구현 클래스마다 테이블 전략

##### 7.1.1 조인 전략

엔티티 각각을 모두 테이블로 만들고 자식 테이블이 부모 테이블의 기본 키를 받아서 기본 키 + 외래 키로 사용하는 전략입니다. 
주의할 점은, 객체는 타입으로 구분할 수 있지만 테이블은 타입의 개념이 없기 때문에 타입을 구분하는 칼럼을 추가해야합니다.

```java
@Entity
@Inheritance (strategy = InheritanceType.JOINED) // 상속 매핑은 부모 클래스에 @Inheritance 를 사용
@DiscriminatorColumn (name = "DTYPE")	// 부모 클래스에 구분 칼럼을 지정
public abstract class Item {
  
  @Id @GeneratedValue
  @Column (name = "ITEM_ID")
  private Long id;
  
  private String name; 
  private int price;
}

@Entity
@DiscriminatorColumn ("A")  
public class Album extends Item{
  
  private String artist;
}

@Entity
@DiscriminatorColumn ("M") // 영화 엔티티를 저장하면 구분 칼럼인 DTYPE 에 값 M이 저장
public class Movie extends Item{
  
  private String director;
  private String actor;
}
```

- 장점
  - 테이블이 정규화 됩니다.
  - 외래 키 참조 무결성 제약 조건을 활용할 수 있습니다.
  - 저장공간을 효율적으로 사용할 수 있습니다.
- 단점
  - 조회할 때 조인이 많이 사용되므로 성능이 저하될 수 있습니다.
  - 조회 쿼리가 복잡합니다.
  - 데이터를 등록할 INSERT SQL 을 두 번 실행합니다.
- 특징
  - JPA 표준 명세는 구분 칼럼을 사용하도록 하지만, 하이버네이트는 구분 칼럼 없이도 동작합니다.

##### 7.1.2 단일 테이블 전략

테이블을 하나만 사용하고 구분 칼럼으로 어떤 자식 데이터가 저장되었는지 구분합니다.

```java
@Entity
@Inheritance (strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn (name = "DTYPE")	
public abstract class Item {
  
  @Id @GeneratedValue
  @Column (name = "ITEM_ID")
  private Long id;
  
  private String name; 
  private int price;
}

@Entity
@DiscriminatorColumn ("A")  
public class Album extends Item{
  ...
}

@Entity
@DiscriminatorColumn ("M") 
public class Movie extends Item{
  ...
}
```

- 장점
  - 조인이 필요 없으므로 일반적으로 조회 성능이 빠릅니다.
  - 조회 쿼리가 단순합니다.
- 단점
  - 자식 엔티티가 매핑한 칼럼은 모두 null을 허용해야합니다.
  - 단일 테이블에 모든 것을 저장하므로 테이블이 커질 수 있습니다. 상황에 따라서는 조회 성능이 느려질 수 있습니다.
- 특징
  - 구분 칼럼을 꼭 사용해야합니다.

##### 7.1.3 구현 클래스마다 테이블 전략

자식 엔티티마다 테이블을 만듦니다. 그리고, 자식 테이블에 각각에 필요한 칼럼이 모두 있습니다. 이 전략은 추천하지 않는 전략입니다.

```java
@Entity
@Inheritance (strategy = InheritanceType.TABLE_PER_CLASS)
public abstract class Item {
  
  @Id @GeneratedValue
  @Column (name = "ITEM_ID")
  private Long id;
  
  private String name; 
  private int price;
}

@Entity
public class Album extends Item{
  ...
}

@Entity
public class Movie extends Item{
  ...
}
```

- 장점
  - 서브 타입을 구분해서 처리할 때 효과적입니다.
  - not null 제약조건을 사용할 수 있습니다.
- 단점
  - 여러 자식 테이블을 함께 조회할 때 성능이 느립니다. (SQL 의 UNION 을 사용해야합니다.)
  - 자식 테이블을 통합해서 쿼리하기 어렵습니다.
- 특징
  - 구분 칼럼을 사용하지 않습니다.

#### 7.2 MappedSuperClass

부모 클래스는 테이블과 매핑하지 않고, 부모 클래스를 상속받는 자식 클래스에게 매핑 정보만 제공하고 싶으면 @MappedSuperClass 를 사용합니다.

```java
//테이블과 매핑할 필요가 없고 자식 엔티티에 공통으로 사용되는 매핑 정보만 제공
@MappedSuperClass
public abstract class BaseEntity{ 
  @Id @GeneratedValue
  private Long id;
  
  private String name;
}

@Entity
public class Member extends BaseEntity{
  
  //id, name 상속
  
  private String email;
}

@Entity
public class Seller extends BaseEntity{
  
  //id, name 상속
  
  private String shopName;
}
```

부모로부터 물련 받은 매핑 정보를 재정의 하려면,

```java
@Entity
//부모에게 상속 받은 id 속성의 칼럼명을 MEMBER_ID 로 재정의
@AttributeOverride(name = "id", column = @Column(name = "MEMBER_ID"))
public class Member extends BaseEntity{
  ...
}
```

둘 이상을 재정을 하려면,

```java
@Entity
@AttributeOverrides({
  @AttributeOverride(name = "id", column = @Column(name = "MEMBER_ID")),
  @AttributeOverride(name = "name", column = @Column(name = "MEMBER_NAME"))
})
public class Member extends BaseEntity{
  ...
}
```

- @MappedSuperClass 의 특징
  - @MappedSuperClass 로 지정한 클래스는 엔티티가 아닙니다. 따라서, em.find() 나 JPQL 을 사용할 수 없습니다.
  - 이 클래스를 직접 생성해서 사용할 일은 거의 없으므로, 추상 클래스로 만드는 것을 권장합니다.

#### 7.3 복합 키와 식별 관계 매핑

##### 7.3.1 식별 관계 VS 비식별 관계

- 식별 관계
  - 부모 테이블의 기본 키를 내려받아서, 자식 테이블의 기본 키 + 외래 키로 사용하는 관계입니다.
- 비식별 관계
  - 부모 테이블의 기본 키를 내려받아서, 자식 테이블의 외래키로만 사용하는 관계입니다.
    - 필수적 비식별 관계
      - 외래 키에 NULL을 허용 하지 않습니다. 연관관계를 필수적으로 맺어야 합니다.
    - 선택적 비식별 관계
      - 외래 키에 NULL을 허용합니다. 연관관계를 맺을지 선택할 수 있습니다.

##### 7.3.2 복합키: 비식별 관계 매핑

둘 이상의 칼럼으로 구성된 복합 기본키를 다음과 같이

```java
@Entity
publi class Hello {
  @Id
  private String id;
  
  @Id
  private String id2;
}
```

해보면 매핑 오류가 발생합니다.

JPA 에서 식별자를 둘 이상 사용하려면 별도의 식별자 클래스를 만들어야합니다. 

복합키를 지원하기 위해, 관계형 데이터베이스에 가까운 방법인 @IdClass 와, 객체지향에 가까운 @EmbededId 두 가지 방법을 제공합니다. 

###### IdClass

```java
@Entity
@IdClass (ParentId.class)
public class Parent {
  
  @Id @Column (name = "PARENT_ID1")
  private String id1; //ParerntId.id1 과 연결
  
	@Id @Column (name = "PARENT_ID2")
  private String id2; //ParerntId.id2 과 연결
  
  private String name;
}

public class ParentId implements Serializable { // Serializable 를 구현해야합니다.
  
  // 식별자 클래스의 속성=명과 엔티티에서 사용하는 식별자의 속성명이 같아야합니다.
  private String id1; //Parernt.id1 과 연결
  private String id2; //Parernt.id2 과 연결
  
  //기본 생성장가 있어야합니다.
  public ParentId(){ 
  }
 
  public ParentId(String id1, String id2){
    this.id1 = id1;            
    this.id2 = id2;
  }
  
  // equals 와 hashCode를 구현해야합니다.
  @Override
  public boolean equals(Object o){...} 
  @Override
  public int hashCode(){...}
  
}
```

복합키를 사용하는 엔티티를 저장하면,

```java
Parent parent = new Parent;
parent.setId1("myId1");
parent.setId2("myId2");
parent.setName("ParentName");
em.persist(parent)
```

영속성 컨텍스트에 엔티티를 등록하기 직전에, 내부에서 Parent.id1, Parent.id2 값을 사용해서 식별자 클래스인 ParentId 를 생성하고 영속성 컨텍스트의 키로 사용합니다.

ParentId 를 사용해서 엔티티를 조회합니다.

```java
ParentId parentId = new ParentId("myId1", "myId2");
Parent parent = em.find(Parent.class, parentId);
```

자식 클래스는,

```java
@Entity
public class Child {
  
  @Id
  private String id;
  
  // 부모 테이블의 기본 키 칼럼이 복합 키이므로, 자식 테이블의 외래 키도 복합키
  // 외래 키 매핑 시 여러 칼럼을 매핑해야하므로 @JoinColums를 사용
	@ManyToOne
  @JoinColums({
    @JoinColum(name = "PARENT_ID1", referencedColumnName = "PARENT_ID1"),
    @JoinColum(name = "PARENT_ID2", referencedColumnName = "PARENT_ID2"),
  })
  
  private Parent parent;
}
```

###### EmbeddedID

```java
@Entity
public class Parent {
  
  @EmbeddedId
  private ParentId id;
  
  private String name;
  
  ...
}

@Embeddable
public class ParentId implements Serializable { //Serializable 를 구현
  
  @Column(name = "PARENT_ID1")
  private String id1;
  
  @Column(name = "PARENT_ID2")
  private String id2;
  
  //equals and hashCode 구현
  ...
}
```

엔티티를 저장하면,

```java
Parent parent = new Parent;
ParentId parentId = new ParentId("myId1", "myId2");

parent.setId(parentId);
parent.setName("ParentName");
em.persist(parent)
```

엔티티를 조회하면,

```java
ParentId parentId = new ParentId("myId1", "myId2");
Parent parent = em.find(Parent.class, parentId);
```

##### 7.3.3 복합키: 식별 관계 매핑

부모, 자식, 손자까지 계속 기본 키를 전달하는 식별관계를 생각해보려고 합니다.

###### IdClass

```java
@Entity
public class Parent {
  
  @Id @Column (name = "PARENT_ID")
  private String id;
  private String name;
  ...
}

@Entity
@IdClass(ChildId.class)
public class Child {
  
  // 식별 관계는 기본 키와 외래키를 같이 매핑해야합니다.
  // 따라서, 식별자 매핑인 @Id 와 연관관계 매팽인 @ManyToOne 을 같이 사용합니다.
  @Id @ManyToOne @JoinColumn (name = "PARENT_ID")
  private Parent parent;
  
  @Id @Column (name = "CHILD_ID")
  private Parent childId;
  
  private String name;
  ...
}

public class ChildId implements Serializable {
   
  private String parent; //Child.parent 매핑
  private String childId; //Child.childId 매핑
  
  //equals, hashCode
  ...
}

@Entity
@IdClass(GrandChildId.class)
public class GrandChild {
  
  @Id 
  @ManyToOne
  @JoinColumns({
    @JoinColumn(name = "PARENT_ID"),
    @JoinColumn(name = "CHILD_ID")
  })
  private Child child;
  
  @Id @Column (name = "GRANDCHILD_ID")
  private String id;
  
  private String name;
  ...
}

public class GrandChildId implements Serializable {
   
  private ChildId child; //GrandChild.child 매핑
  private String id; //Child.id 매핑
  
  //equals, hashCode
  ...
}
```

###### EmbeddeId

```java
@Entity
public class Parent {
  
  @Id @Column (name = "PARENT_ID")
  private String id;
  
  private String name;
  ...
}

@Entity
public class Child {
  
  @EmbeddedId
  private ChildId id;
  
  @MapsId("parentId") //ChildId.parentId 매핑. 외래 키와 매핑한 연관관계를 기본 키에도 매핑.
  @ManyToOne
  @JoinColumn (name = "PARENT_ID")
  private Parent parent;
  
  private String name;
  ...
}

@Embeddable
public class ChildId implements Serializable {
   
  private String parentId; // @MapsId("parentId") 로 매핑
  
  @Column(name = "CHILD_ID")
  private String id;
  
  //equals, hashCode
  ...
}

@Entity
@IdClass(GrandChildId.class)
public class GrandChild {
  
  @EmbededId
  private GrandChildId id;
  
  @MapsId ("childId") // GrandChildId.childId 매핑
  @ManyToOne
  @JoinColumns({
    @JoinColumn(name = "PARENT_ID"),
    @JoinColumn(name = "CHILD_ID")
  })
  private Child child;
    
  private String name;
  ...
}

@Embeddable
public class GrandChildId implements Serializable {
   
  private ChildId child; // @MapsId("childId") 로 매핑
  
  @Column ( name = "GRANDCHILD_ID")
  private String id;
  
  //equals, hashCode
  ...
}
```

##### 7.3.4 비식별 관계로 구현

방금 예를, 비식별 관계로 변경하려고 합니다.

```java
@Entity
public class Parent {
  
  @Id @GeneratedValue @Column (name = "PARENT_ID")
  private String id;
  
  private String name;
  ...
}

@Entity
public class Child {
  
  @Id @GeneratedValue @Column (name = "CHILD_ID")
  private Long id;
  
  private String name;
  
  @ManyToOne @JoinColumn (name = "PARENT_ID")
  private Parent parent;
  
  ...
}

@Entity
public class GrandChild {
  
  @Id @GeneratedValue @Column (name = "GRANDCHILD_ID")
  private Long id;
  
  private String name;
  
  @ManyToOne @JoinColumn(name = "CHILD_ID")
  private Child child;
    
  ...
}
```

##### 7.3.5 일대일 식별 관계

자식 테이블의 기본 키 값으로 부모 테이블의 기본 키 값만 사용합니다.

```java
@Entity
public class Board {
  
  @Id @GeneratedValue @Column (name = "BOARD_ID")
  private String id;
  
  private String title;
  
  @OneToOne(mappedBy = "board")
  private BoardDetail boardDetail;
  ...
}

@Entity
public class BoardDetail {
  
  @Id
  private Long boardId;
  
  @MapsId //BoardDetail.boardId 매핑
  @OneToOne
  @JoinColumn (name = "BOARD_ID")
  private Board board;
  
  private String content;
  ...
}
```

#### 7.4 조인 테이블

데이터베이스 테이블의 연관관계를 설계하는 방법은 두 가지 입니다.

- 조인 칼럼 사용 ( 외래키)
  - 회원이 사물함을 사용하기 전까지는 아직 둘 사이의 관계가 없으므로, MEMBER Table 의 LOKCER_ID 외래키에 null
  - 외래 키애 null 허용하는 관계를 선택적 비식별 관계라고 합니다.
  - null 허용하므로, 회원과 사물함 조인할 때는 OUTER JOIN 사용해야합니다.
  - 회원과 사물함이 아주 가끔 관계를 맺으면, 외래 키 대부분 값에 null 이 저장이 됩니다.
- 조잍 테이블 사용 (테이블)
  - 조인 테이블에서 두 테이블의 외래 키를 가지고 연관 관계를 관리합니다.

##### 7.4.1 일대일 조인 테이블

```java
@Entity
public class Parent {
  
  @Id @GeneratedValue @Column (name = "PARENT_ID")
  private Long id;
  
  private String name;
  
  @OneToOne
  @JoinTable(name = "PARENT_CHILD", // 매핑할 조인 테이블 이름
            joinColums = @JoinColumn(name = "PARENT_ID"), // 현재 엔티티를 참조하는 외래 키
            inverseJoinColums = @JoinColum(name = "CHILD_ID") // 반대방향 엔티티를 참조하는 외래 키
  )
  
  private Child child;
  ...
}

@Entity
public class Child {
  
  @Id @GeneratedValue @Column (name = "CHILD_ID")
  private Long id;
  
  private String name;
  ...
}
```

##### 7.4.2 일대다 조인 테이블

```java
@Entity
public class Parent {
  
  @Id @GeneratedValue @Column (name = "PARENT_ID")
  private Long id;
  
  private String name;
  
  @OneToMany
  @JoinTable(name = "PARENT_CHILD", // 매핑할 조인 테이블 이름
            joinColums = @JoinColumn(name = "PARENT_ID"), // 현재 엔티티를 참조하는 외래 키
            inverseJoinColums = @JoinColum(name = "CHILD_ID") // 반대방향 엔티티를 참조하는 외래 키
  )
  
  private List<Child> child = new ArrayList<Child>();
  ...
}

@Entity
public class Child {
  
  @Id @GeneratedValue @Column (name = "CHILD_ID")
  private Long id;
  
  private String name;
  ...
}
```

##### 7.4.3 다대일 조인 테이블

```java
@Entity
public class Parent {
  
  @Id @GeneratedValue @Column (name = "PARENT_ID")
  private Long id;
  
  private String name;
  
  @OneToMany (mappedBy = "PARENT_ID")
  private List<Child> child = new ArrayList<Child>();
  ...
}

@Entity
public class Child {
  
  @Id @GeneratedValue @Column (name = "CHILD_ID")
  private Long id;
  
  private String name;

  @ManyToOne(optional = false)
  @JoinTable(name = "PARENT_CHILD", // 매핑할 조인 테이블 이름
            joinColums = @JoinColumn(name = "CHILD_ID"), // 현재 엔티티를 참조하는 외래 키
            inverseJoinColums = @JoinColum(name = "PARENT_ID") // 반대방향 엔티티를 참조하는 외래 키
  )
  
  private Parent parent;
  ...
}
```

##### 7.4.4 다대다 조인 테이블

```java
@Entity
public class Parent {
  
  @Id @GeneratedValue @Column (name = "PARENT_ID")
  private Long id;
  
  private String name;
  
  @ManyToMany
  @JoinTable(name = "PARENT_CHILD", // 매핑할 조인 테이블 이름
            joinColums = @JoinColumn(name = "PARNET_ID"), // 현재 엔티티를 참조하는 외래 키
            inverseJoinColums = @JoinColum(name = "CHILD_ID") // 반대방향 엔티티를 참조하는 외래 키
  )
  private List<Child> child = new ArrayList<Child>();
  ...
}

@Entity
public class Child {
  
  @Id @GeneratedValue @Column (name = "CHILD_ID")
  private Long id;
  
  private String name;
  ...
}
```

#### 7.4 엔티티 하나에 여러 테이블 매핑

```java
@Entity
@Table ( name = "BOARD") // BOARD 테이블과 매핑

// BOARD_DETAIL 테이블을 추가로 매핑
@SecondaryTable ( name = "BOARD_DETAIL", //매핑할 다른 테이블 이름
                 //매핑할 다른 테이블의 기본 키 칼럼 속성
                pkJoinColums = @PrimaryJoinColumn(name = "BOARD_DETAIL_ID"))
public class Board {
  
  @Id @GeneratedValue @Column (name = "BOARD_ID")
  private Long id;
  
  private String title;
  
  // content 필드는 BOARD_DETAIL 테이블의 칼럼에 매핑
  @Column(table = "BOARD_DETAIL")
  private String content;
  ...
}
```

@SecondaryTable 를 사용해서 두 테이블을 하나의 엔티티에 매핑하는 방법 보다는, 테이블당 엔티티를 각각 만들어서 일대일 매핑하는 것을 권장합니다.