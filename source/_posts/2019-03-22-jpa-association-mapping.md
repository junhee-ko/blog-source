---
layout: post
title: 연관관계 매핑
date: 2019-03-22
categories: JPA
---

다대일, 일대다, 일대일, 다대다 연관관계 매핑에 대해 정리한다.

## 다대일

다대일 단방향과 다대일 양방향이 있다.

### 다대일 단방향

```kotlin
@Entity
class Member(
    
    @Id
    @Column(name = "MEMBER_ID")
    val id: String,

    val username: String,

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    var team: Team? = null
)

@Entity
class Team(
    
    @Id
    @Column(name = "TEAM_ID")
    val id: String,

    val name: String
)
```

Member 는 Member.team 으로 팀 엔티티를 조회할 수 있다.
하지만, 팀은 멤버를 참조하는 필드가 없다. 

그리고, @JoinColumn(name = "TEAM_ID") 를 통해서 Member.team 필드를 외래 키와 매핑했다.
즉, Member.team 필드로 멤버 테이블의 TEAM_ID 외래 키를 관리한다.

### 다대일 양방향

```kotlin
@Entity
class Member(
    
    @Id
    @Column(name = "MEMBER_ID")
    val id: String,

    val username: String,

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    var team: Team? = null
) {

    fun setTeam(team: Team) {
        this.team = team
        
        if(!team.members.contains(this)){
            team.members.add(this)
        }
    }
}

@Entity
class Team(
    
    @Id
    @Column(name = "TEAM_ID")
    val id: String,

    val name: String,

    @OneToMany(mappedBy = "team")
    val members: MutableList<Member> = mutableListOf()
){
    
    fun addMember(member: Member){
        this.members.add(member)
        
        if(member.team != this){
            member.setTeam(this)
        }
    }
}
```

양방향은 외래 키가 있는 쪽이 연관 관계의 주인이다. 
여기서는 다 쪽인 멤버 테이블이 외래 키를 가지고 있기 때문에, Member.team 이 연관 관계의 주인이다.

양방향 연관 관계는 항상 서로를 참조해야한다. 어느 한 쪽만 참조하면 양항향 연관 관계라고 할 수 없다.
그리고, Member 클래스의 setTeam 과 Team 클래스의 addMember 처럼 서로 참조할 때는 연관 관계 편의 메서드를 사용하는 것이 좋다. 

## 일대다 

일대다 단방형과 일대다 양방향이 있다.

### 일대다 단방향

```kotlin
@Entity
class Team(

    @Id
    @Column(name = "TEAM_ID")
    val id: String,

    val name: String,

    @OneToMany
    @JoinColumn(name = "TEAM_ID") // MEMBER 테이블의 TEAM_ID (FK)
    val members: MutableList<Member> = mutableListOf()
)

@Entity
class Member(
    
    @Id
    @Column(name = "MEMBER_ID")
    val id: String,

    val username: String
)
```

팀 엔티티의 Team.members 로 멤버 테이블의 TEAM_ID 외래 키를 관리한다.
보통, 자신이 매핑한 테이블의 외래 키를 관리하는데, 일대다 단방향 매핑은 반대쪽 테이블의 외래 키를 관리한다.

본인 테이블에 외래 키가 있으면 엔티티의 저장과 연관 관계 처리를 INSERT SQL 한 번으로 끝낼 수 있다.
하지만, 다른 테이블에 외래 키가 있으면 연관 관계 처리를 위한 UPDATE SQL 을 추가로 실행해야한다.

```kotlin
fun save(){
    val member1 = Member(id = "member1", username = "jko")
    val member2 = Member(id = "member2", username = "junhee")

    val team = Team(id = "team1", name = "This is Team")
    team.members.add(member1)
    team.members.add(member2)

    entityManager.persist(member1)
    entityManager.persist(member2)
    entityManager.persist(team)
}
```

실행된 결과 SQL 은,

```sql
insert into Member ...
insert into Member ...
insert into Team ...
update Member ...
update Member ...
```

멤버 엔티티를 저장할 때는 멤버 테이블의 TEAM_ID 외래 키에 아무 것도 저장되지 않는다. 
대신, 팀 엔티티를 저장할 때, Team.members 참조 값을 확인해서 멤버 테이블의 TEAM_ID 외래 키를 업데이트한다.

그래서, 일대다 단방향 보다는 다대일 양방향 매핑을 사용하는 것이 좋다.
다대일 양방향 매핑에서는, 관리해야하는 외래 키가 본인 테이블에 있기 때문이다.

### 일대다 양방향

일대다 양방향은 존재하지 않는다. 대신 다대일 양방향 매핑을 사용할 수 있다.
(일대다 양방향과 다대일 양방향이 같은 말이지만, 왼쪽을 연과관계의 주인으로 가정하고 분류하자)

더 정확히는, @OneToMany 는 연관관계의 주인이 될 수 없다.
왜냐하면, 관계형 데이터베이스에서는 항상 다 쪽에 외래 키가 있기 때문이다.

## 일대일

일대일 관계에서는, 주 테이블이나 대상 테이블 중에 누가 외래 키를 가질지 선택해야한다.

### 주 테이블에 외래 키

#### 주 테이블에 외래 키 > 단방향

멤버와 라커의 일대일 단방향을 예로 들어보자.

```kotlin
@Entity
class Member(
    
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    val id: Long,

    val username: String,

    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    val locker: Locker
)

@Entity
class Locker(
    
    @Id
    @GeneratedValue
    @Column(name = "LOCKER_ID")
    val id: Long,

    val name: String
)
```

#### 주 테이블에 외래 키 > 양방향

```kotlin
@Entity
class Member(
    
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    val id: Long,

    val username: String,

    @OneToOne
    @JoinColumn(name = "LOCKER_ID")
    val locker: Locker
)

@Entity
class Locker(
    
    @Id
    @GeneratedValue
    @Column(name = "LOCKER_ID")
    val id: Long,

    val name: String,

    @OneToOne(mappedBy = "locker")
    val member: Member
)
```

양방향이므로 연관 관계의 주인을 정해야한다. 
멤버 테이블이 외래 키를 가지고 있으므로, Member.locker 가 연관 관계의 주인이다.
Locker.member 는 mappedBy 로 연관관계의 주인이 아님을 명시한다.

### 대상 테이블에 외래 키

#### 대상 테이블에 외래 키 > 단방향

일대일 관계 중 대상 테이블에 외래 키가 있는 단방향 관계는 JPA 에서 지원하지 않는다.

#### 대상 테이블에 외래 키 > 양방향

```kotlin
@Entity
class Member(
    
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    val id: Long,

    val username: String,

    @OneToOne(mappedBy = "member")
    val locker: Locker
)

@Entity
class Locker(
    
    @Id
    @GeneratedValue
    @Column(name = "LOCKER_ID")
    val id: Long,

    val name: String,

    @OneToOne
    @JoinColumn(name = "MEMBER_ID")
    val member: Member
)
```

## 다대다

관계형 데이터베이스는 정규화된 테이블 두 개로 다대다 관계를 표현할 수 없다.
대신 일대다, 다대일 관계로 풀 수 있는 연결 테이블을 사용한다.

예를 들어, 회원은 상품들을 주문할 수 있고 상품은 회원들에 의해 주문될 수 있다.
테이블 관계는 아래 처럼 구성될 수 있다.

- Member
  - MEMBER_ID (PK)
  - USERNAME
- Product
  - PRODUCT_ID
  - NAME
- Member_Product
  - MEMBER_ID (PK, FK)
  - PRODUCT_ID (PK, FK)

객체는 테이블과 다르게 객체 두 개로 다대다 관계를 표헌할 수 있다.

### 다대다 단방향

```kotlin
@Entity
class Member(

    @Id
    @Column(name = "MEMBER_ID")
    val id: Long? = null,

    val username: String,

    @ManyToMany
    @JoinTable(
        name = "MEMBER_PRODUCT",
        joinColumns = [JoinColumn(name = "MEMBER_ID")],        // 멤버와 매핑할 조인 칼럼 정보
        inverseJoinColumns = [JoinColumn(name = "PRODUCT_ID")] // 반대 방향인 상품과 매핑할 조인 칼럼 정보 
    )
    val products: MutableList<Product>
)

@Entity
class Product(

    @Id
    @Column(name = "PRODUCT_ID")
    val id: String,

    val name: String
)
```

### 다대다 양방향

```kotlin
@Entity
class Member(

    @Id
    @Column(name = "MEMBER_ID")
    val id: Long? = null,

    val username: String,

    @ManyToMany
    @JoinTable(
        name = "MEMBER_PRODUCT",
        joinColumns = [JoinColumn(name = "MEMBER_ID")],
        inverseJoinColumns = [JoinColumn(name = "PRODUCT_ID")]
    )
    val products: MutableList<Product>
)

@Entity
class Product(

    @Id
    @Column(name = "PRODUCT_ID")
    val id: String,

    val name: String,
    
    @ManyToMany(mappedBy = "products") // 연관관계의 주인이 아님
    val members: MutableList<Member>
)
```

### 다대다 > 연결 엔티티 사용

멤버가 상품을 주문하면, 연결 테이블에 단순히 주문한 회원 아이디, 상품 아이디 이외에 주문 수량 칼럼이나 주문한 날짜와 같은 칼럼을 사용하고 싶을 수 있다.
이를 위해 추가적인 연결 엔티티를 사용할 수 있다.

```kotlin
@Entity
class Member(

    @Id
    @Column(name = "MEMBER_ID")
    val id: Long? = null,

    val username: String,

    @OneToMany(mappedBy = "member")
    val memberProducts: MutableList<MemberProduct>
)

@Entity
class Product(

    @Id
    @Column(name = "PRODUCT_ID")
    val id: String,

    val name: String,
)

@Entity
@IdClass(MemberProductId::class) // 복합 기본키
class MemberProduct(
    
    @Id
    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    val member: Member,

    @Id
    @ManyToOne
    @JoinColumn(name = "PRODUCT_ID")
    val product: Product,

    val orderAmount: Int
)

data class MemberProductId(
    val member: String,
    val product: String
) : Serializable
```

회원 상품은 회원의 기본키를 받아서 자신의 기본키로 사용함과 동시에 회원 과의 관계를 위한 외래키로 사용한다.
회원 상품은 상품의 기본키를 받아서 자신의 기본키로 사용함과 동시에 상품 과의 관계를 위한 외래키로 사용한다.
그리고, MemberProductId 식별자 클래스로 두 기본 키를 묶어서 복합 기본키로 사용한다.

JPA 에서는 복합 기본키를 사용하려면 별도의 식별자 클래스를 만들어야한다. 
그리고 엔티티에 @IdClass 로 식별자 클래스를 지정해야한다.

### 다대다 > 새로운 기본 키 사용

복합 키를 사용하면 복잡하다. 
복합 키를 위한 식별자 클래스를 만들어야하고, @IdClass 를 사용해야하고, 식별자 클래스에 equals, hashCode 등도 구현해야한다.
복합 키를 사용하지 않고 간단히 다대다 관계를 만들 수 있다.

기본 키 생성 전략으로, 데이터베이스에서 자동으로 생성해주는 대리 키를 Long 값으로 사용하는 것이다.
복합 키를 따로 만들지 않아도 되므로, 간단히 매핑 가능하다.

```kotlin
@Entity
class Order(

    @Id @GeneratedValue
    @Column(name = "ORDER_ID")
    val id: Long,
    
    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    val member: Member,

    @ManyToOne
    @JoinColumn(name = "PRODUCT_ID")
    val product: Product,

    val orderAmount: Int
)

@Entity
class Member(

  @Id
  @Column(name = "MEMBER_ID")
  val id: Long? = null,

  val username: String,

  @OneToMany(mappedBy = "member")
  val memberProducts: MutableList<Order>
)

@Entity
class Product(

  @Id
  @Column(name = "PRODUCT_ID")
  val id: String,

  val name: String,
)
```

---

자바 ORM 표준 프로그래밍 <김영한>
