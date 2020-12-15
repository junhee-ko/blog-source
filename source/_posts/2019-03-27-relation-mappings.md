---
layout: post
title:  "[자바 ORM 표준 JPA 프로그래밍] 6장_다양한 연관관계 매핑"
date:   2019-03-27
categories: JPA
---

이번 포스팅에서는 다양한 연관관계를 다룹니다. 엔티티의 연관 관계를 매핑 할 때는 다음 세 가지를 고려해야합니다.

- 다중성
  - 다대일
  - 일대다
  - 일대일
  - 다대다
- 단방향, 양방향
  - 테이블에는 방향이라는 개념이 없습니다.
  - 객체는 참조용 필드를 가지고 있는 객체만 연관된 객체를 조회할 수 있습니다.
- 연관관계의 주인
  - 데이터베이스는 외래 키 하나로 두 데이블이 연관관계 맺습니다.
  - 엔티티를 양방향으로 매핑하면 A->B, B->A 2곳에서 서로를 참조하기 때문에, 객체의 연관관계를 관리하는 포인트는 2곳입니다. 따라서, 두 객체 연관관계 중 하나를 정해서 데이터베이스 외래 키를 관리해야합니다.

#### 1. 다대일

데이터베이스 테이블의 일, 다 관계에서 외래 키는 항상 다쪽에 있습니다. 따라서, 객체 양방향 관계에서 연관관계의 주인은 항상 다쪽입니다.

##### 1.1 다대일 단방향

회원은 팀 엔티티를 참조 가능합니다. 팀은 회원 참조하는 필드가 없습니다.

```java
@Entity
public class Member{
  
  @Id @GeneratedValue
  @Column (name = "MEMBER_ID")
  private Long id;
  
  private String username;
  
  @ManyToOne
  @JoinColum (name = "TEAM_ID")
  private Team team;	
  
  //Getter, Setter ...
}
```

```java
@Entity
public class Team{

  @Id @GeneratedValue
  @Column (name = "TEAM_ID")
  private Long id;
  
  private String name;
  
  //Getter, Setter ...
}
```

##### 1.1 다대일 양방향

양방향 연관관계는 항상 서로 참조해야합니다. 서로 참조하게 하려면 연관관계 편의 메소드를 작성하는게 좋습니다.

```java
@Entity
public class Member{
  
  @Id @GeneratedValue
  @Column (name = "MEMBER_ID")
  private Long id;
  
  private String username;
  
  @ManyToOne
  @JoinColum (name = "TEAM_ID")
  private Team team;
  
  public void setTeam(Team team){
    this.team = team;
    
    //무한 루프에 빠지지 않도록 체크
    if(!team.getMembers().contains(this)){
      team.getMembers().add(this);
    }
  }
}
```

```java
@Entity
public class Team{
 
  @Id @GeneratedValue
  @Column (name = "TEAM_ID")
  private Long id;
  
  private String name;
  
  @OneToMany (mappedBy = "team")
  private List<Member> members = new ArrayList<Member>();
  
  public void addMember(Member member){
    this.members.add(member);
    
    //무한 루프에 빠지지 않도록 체크
    if(member.getTeam() != this){
      member.setTeam(this);
    }
  }
  
  //Getter, Setter ...
}
```

#### 2. 일대다

##### 일대다 단방향

일대다 단방향은 약간 특이합니다. 일대다 관계에서, 외래 키는 항상 다쪽 테이블에 있습니다. 하지만 다 쪽인 Memebr 엔티티에는 외래 키를 매핑할 수 있는 참조 필드가 없습니다. 대신에, Team 엔티티에는 참조 필드인 members 가 있습니다. 따라서 반대편 테이블의 외래키를 관리하는 특이한 모습이 나타납니다.

```java
@Entity
public class Team{
 
  @Id @GeneratedValue
  @Column (name = "TEAM_ID")
  private Long id;
  
  private String name;
  
  @OneToMany
  @JoinColum (name = "TEAM_ID") //MEMBER 테이블의 TEAM_ID (FK)
  private List<Member> members = new ArrayList<Member>();
   
  //Getter, Setter ...
}
```

```java
@Entity
public class Member{
  
  @Id @GeneratedValue
  @Column (name = "MEMBER_ID")
  private Long id;
  
  private String username;
  
  //Getter, Setter
}
```

- 일대다 단방향 매핑의 단점
  - 매핑한 객체가 관리하는 외래 키가 다른 테이블에 있다는 점입니다.
  - 연관관계 처리를 위한 UPDATE SQL 을 추가로 실행해야합니다.
  - 다음 코드에서, Member 엔티티는 Team 엔티티를 모릅니다. 따라서 Member 엔티티를 저장할 때는 Member 테이블의 TEAM_ID 외래키에 아무 값도 저장되지 않습니다. 대신, Team 엔티티를 저장할 때 Team.members 의 참조 값을 확인해서 회원 테이블에 있는 TEAM_ID 외래 키를 업데이트 합니다.

```java
public void testSave(){
  
  Member member1 = new Member("member1");
  Member member2 = new Member("member2");
  
  Team team1 = new Team("team1");
  team1.getMembers().add(member1);
  team1.getMembers().add(member2);
  
  em.persist(member1); //INSERT-member1
  em.persist(member2); //INSERT-member2
  em.persist(team1);   //INSERT-team1, UPDATE-member.fk, UPDATE-member2.fk
  
  transaction.commit();
}
```

따라서, 일대다 단방향 매핑보다는 다대일 양방향 매핑을 권장합니다.

##### 일대다 양방향

둘 다 같은 키를 관리하므로 문제가 발생할 수 있습니다. 따라서, 반대편인 다대일 쪽은 insertable =false, update = false 를 설정해서 읽기만 가능하게 합니다. 되록, 다대일 양방향 매핑을 사용해야합니다.

```java
@Entity
public class Team{
 
  @Id @GeneratedValue
  @Column (name = "TEAM_ID")
  private Long id;
  
  private String name;
  
  @OneToMany
  @JoinColum (name = "TEAM_ID") 
  private List<Member> members = new ArrayList<Member>();
   
  //Getter, Setter ...
}
```

```java
@Entity
public class Member{
  
  @Id @GeneratedValue
  @Column (name = "MEMBER_ID")
  private Long id;
  
  private String username;
  
  @ManyToOne
  @JoinColumn( name = "TEAM_ID", insertable =false, update = false)
  private Team team;
  
  //Getter, Setter ...
}
```

#### 3. 일대일

일대일 관계는 주 테이블이나 대상 테이블 중에 누가 외래키를 가질지 선택해야합니다.

##### 3.1 주 테이블에 외래 키

객체지향 개발자들은 주 테이블에 외래 키가 있는 것을 선호합니다. 

- 단방향

```java
@Entity
public class Member{
  
  @Id @GeneratedValue
  @Column (name = "MEMBER_ID")
  private Long id;
  
  private String username;
  
  @OneToOne
  @JoinColumn( name = "LOCKER_ID")
  private Locker locker;
  
}
```

```java
@Entity
public class Locker{
  
  @Id @GeneratedValue
  @Column (name = "LOCKER_ID")
  private Long id;
  
  private String name;
  
}
```

- 양방향

```java
@Entity
public class Member{
  
  @Id @GeneratedValue
  @Column (name = "MEMBER_ID")
  private Long id;
  
  private String username;
  
  @OneToOne
  @JoinColumn( name = "LOCKER_ID")
  private Locker locker;
  
}
```

```java
@Entity
public class Locker{
  
  @Id @GeneratedValue
  @Column (name = "LOCKER_ID")
  private Long id;
  
  private String name;
  
  @OneToOne (mappedBy ="locker")
  private Member member;
  
}
```

##### 3.2 대상 테이블에 외래 키

- 단방향
  - 일대일 관계 중 대상 테이블에 외래 키가 있는 단방향 관계는 JPA 에서 지원하지 않습니다.
- 양방향

```java
@Entity
public class Member{
  
  @Id @GeneratedValue
  @Column (name = "MEMBER_ID")
  private Long id;
  
  private String username;
  
  @OneToOne (mappedBy ="member")
  private Locker locker;
  
}
```

```java
@Entity
public class Locker{
  
  @Id @GeneratedValue
  @Column (name = "LOCKER_ID")
  private Long id;
  
  private String name;
  
  @OneToOne 
  @JoinColumn(name ="MEMBER_ID")
  private Member member;
  
}
```

#### 4. 다대다

관계형 데이터베이스는 정규화된 테이블 2개를 다대다 관계로 표현할 수 없습니다. 그래서 일대다, 다대일 관계로 풀어내는 연결 테이블을 사용합니다. 그런데 객체는 테이블과 다르게 객체 2개로 다대다 관계를 만들 수 있습니다.

##### 4.1 다대다 단방향

```java
@Entity
public class Member{
  
  @Id @Column (name = "MEMBER_ID")
  private Long id;
  
  private String username;
  
  @ManyToMany
  @JoinTable(name = "MEMBER_PRODUCT", joinColums = @JoinColumn (name = "MEMBER_ID"),
            inverseJoinColums = @JoinColumn(name = "PRODUCT_ID"))
  private List<Product> products = new Arraylist<List>();
  
}
```

```java
@Entity
public class Product{
  
  @Id @Column (name = "PRODUCT_ID")
  private Long id;
  
  private String name;
  
}
```

다음은 다대다 관계를 저장하는 예제입니다.

```java
public void save(){
 
  Product productA = new Product();
  productA.setId("productA");
  productA.setName("상품A");
  em.persist(productA);
  
  Member member1 = new Member();
  member1.setId("member1");
  member1.setUsername("회원1");
  member.getProducts.add(productA);	//연관관계 설정
  em.persist(member1);
  
}
```

이 코드를 실행하면 다음 SQL이 실행됩니다.

```sql
INSERT INTO PRODUCT...
INSERT INTO MEMBER...
INSERT INTO MEMBER_PRODUCT...
```

다음은 다대다 관계를 탐색하는 예제입니다.

```java
public void find(){
  
  Member member = em.find(Member.class, "member1");
 	List<Product> products = member.getProducts(); //객체 그래프 탐색
  for (Product product : products){
    System.out.println("product.name = " + product.getName());
  }
}
```

member.getProducts() 를 호출해서 상품 이름을 출력하면 다음 SQL이 실행됩니다.

```sql
SELECT * FROM MEMBER_PRODUCT MP
INNER JOIN PRODUCT P ON MP>PRODUCT_ID = P.PRODUCT_ID
WHER MP.MEMBER_ID = ? 
```

##### 4.2 다대다 양방향

양쪽 중 원하는 곳에 mappedBy 로 연관관계의 주인을 지정합니다.

```java
@Entity
public class Product{
  
  @Id 
  private Long id;
  
  @ManyTomany (mappedBy = "products") //역방향 추가
  private List<Member> members;
  
}
```

다대다의 양방향 연관관계는 다음처럼 설정합니다.

```java
member.getProducts().add(product);
product.getMembers().add(member);
```

회원 엔티티에 다음 편의 메소드를 추가합니다.

```java
public void addProduct(Product product){
  ...
  products.add(product);
  product.getMembers.add(this);
}
```

역방향 탐색은 다음과 같습니다.

```java
public void findInverse(){
  Product product = em.find(Product.class, "productA");
  List <Member> members = product.getMembers();
  for (Member member : members){
    System.out.println("member : " + member.getUsername);
  }
}
```

##### 4.3 다대다: 매핑의 한계와 극복, 연결 엔티티 사용

연결테이블에 컬럼을 추가하면, 더이상 @ManyToMany를 사용할 수 없습니다. 주문 엔티티나 상품 엔티티에는 추가한 칼럼들을 매핑할 수 없기 때문입니다. 따라서, 연결 엔티티를 만들고 이곳에 추가한 칼럼들을 매핑해야합니다.

```java
@Entity
public class Member{
  
  @Id @Column (name = "MEMBER_ID")
  private Long id;
    
  //역방향
  @OneToMany (mappedBy = "member")
  private List<MemberProduct> memberProducts;
  
}
```

```java
@Entity
public class Product{
  
  @Id @Column(name = "PRODUCT_ID")
  private String id;
 
  private String name;
  
}
```

```java
@Entity
@IdClass (MemberProductId.class) //복합 기본키를 매핑
public class MemberProduct{
  
  // 기본키 + 외래키
  @Id
  @ManyToOne
  @JoinColumn (name = "MEMBER_ID")
  private Member member; //MemberProductId.product 와 연결
  
  @Id
  @ManyToOne
  @JoinColumn (name = "PRODUCT_ID")
  private Product product; //MemberProductId.product 와 연결
  
  private int orderAmout;
}
```

```java
// 복합 키를 위한 식별자 클래스
public class MemberProductId implements Serializable{
  
  private String member; //MemberProduct.member 와 연결
  private String product; //MemberProduct.product 와 연결
  
  //hashCode cand equals ...
}
```

##### 4.4 다대다: 새로운 키본 키 사용

추천하는 기본 키 생성 전략은 데이터베이스에서 자동으로 생성해주는 대리 키를 Long 값으로 사용하는 것입니다.
다음은, 연결 테이블에 새로운 기본키를 사용합니다. MemberProduct 보다 Order 이라는 이름으로 합니다.

```java
@Entity
public class Order{
  
  @Id @GeneratedValue
  @Column (name = "ORDER_ID")
  private Long id; 
  
  @ManyToOne
  @JoinColumn (name = "MEMBER_ID")
  private Member member;
  
  @ManyToOne
  @JoinColumn (name = "PRODUCT_ID")
  private Product product;
  
  private int orderAmout;
}
```

##### 4.5 다대다 연관관계 정리

다대다 관계를 일대다, 다대일 관계로 풀어내기 위해 연결 테이블을 만들 때, 식별자 구성 방법은 다음 두 가지가 있습니다.

- 식별관계
  - 받아온 식별자를 기본키 + 외래키로 사용
  - 데이터베이스에서는 이를 식별 관계라고 합니다.
- 비식별 관계
  - 받아온 식별자는 외래키로만 사용하고 새로운 식별자를 추가
  - 데이터베이스에서는 이를 비식별 관계라고 합니다.
  - 이걸 추천합니다.

