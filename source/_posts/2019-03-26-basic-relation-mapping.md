---
layout: post
title:  "[자바 ORM 표준 JPA 프로그래밍] 5장_연관관계 매핑 기초"
date:   2019-03-26
categories: JPA
---

이번 포스팅에서는 객체의 참조와 테이블의 외래키를 매핑하는 것을 다룹니다. 

#### 1. 단방향 연관관계

회원과 팀의 다대일 단방향 관계 (맴버->팀) 를 생각해봅시다.

- 객체 연관 관계
  - 회원은 Member.team 필드를 통해서 팀을 알 수 있습니다.
  - 팀은 회원을 알 수 없습니다.
- 테이블 연관 관계
  - 회원 테이블은 TEAM_ID 외래 키로 팀 테이블과 연관관계를 맺습니다.
  - 회원 테이블과 팀 테이블은 양향관 관계입니다. 회원 테이블은 TEAM_ID 외래 키로 MEMBER JOIN TEAM 과 TEAM JOIN MEMBER 둘 다 가능합니다.
- 개체 연관관계와 테이블 연관관계의 가장 큰 차이
  - 참조를 사용하는 객체의 연관관계는 언제나 단방향 입니다. 객체를 양방향으로 참조하려면 단방향 연관관계 2개를 만들어야합니다.
  - 테이블은 외래 키 하나로 양방향으로 조인할 수 있습니다.

##### 1.1 객체 관계 매핑

```java
@Entity
public class Member {

    @Id
    @Column(name = "MEMBER_ID")
    private String id;

    private String username;

    // 연관관계 매핑
    @ManyToOne // 회원관 팀은 다대일 관계
    @JoinColumn(name = "TEAM_ID") //외래 키를 매핑할 때 사용. name 속성에는 매핑할 외래키 이름.
    private Team team;
    
    // 연관관계 설정
    public void setTeam(Team team){
        this.team = team;
    }

    // Getter, Setter ..

}
```

```java
public class Team {

    @Id
    @Column (name = "TEAM_ID")
    private String id;

    private String name;

    // Getter, Setter ..

}
```

#### 2. 연관관계 사용

연관 관계를 등록, 수정, 삭제, 조회하는 예제를 통해 연관관계를 어떻게 사용하는지 확인합니다.

##### 2.1 저장

```java
  public void save(){
        // 팀 1 저장
        Team team1 = new Team();
        team1.setId("team1");
        team1.setName("팀1");

        // 회원 1 저장
        Member member1 = new Member();
        member1.setTeam(team1);
        em.persist(member1); // 연관관계 설정 member1 -> team1

        // 회원 2 저장
        Member member2 = new Member();
        member2.setTeam(team1);
        em.persist(member2); // 연관관계 설정 member2 -> team1
    }
```

##### 2.2 조회

조회 방법은 두 가지 입니다.

- 객체 그래프 탐색
  - memebr.getTeam();
- JPQL
  - select m from Member m join m.team t where t.name=:teamName
  - :로 시작하는 것은 파라미터를 바인딩하는 문법입니다.

##### 2.3 수정

수정은 update() 가 없습니다. 불러온 엔티티의 값만 변경하면 트렌잭션을 커밋할 때, flush 가 일어나면서 변경 감지 기능이 작동합니다. 

```java
public static void update(){
        // 새로운 팀2
        Team team2 = new Team();
        em.persist(team2);
        
        // 회원1에 새로운 팀2 설정
        Member member = em.find(Member.class, "member1");
        member.setTeam(team2);
    }
```

##### 2.4 제거

```java
public static void delete(){
        Member member = em.find(Member.class, "member1");
        member.setTeam(null);
    }
```

##### 2.5 삭제

기존에 있던 연관관계르ㅓㄹ 먼저 제거하여 삭제해야합니다.

```java
member1.setTeam(null); // 회원1 연관관계 제거
member2.setTeam(null); // 회원2 연관관계 제거
em.remov(team); // 팀삭제
```

#### 3. 양방향 연관관계

회원에서 팀으로 접근하고, 반대 방향인 팀에서도 회원으로 접근하도록 합니다.

##### 3.1 양방향 연관관계 매핑

회원 엔티티는 변경이 없습니다.

```java
@Entity
public class Member {

    @Id
    @Column(name = "MEMBER_ID")
    private String id;

    private String username;

    // 연관관계 매핑
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    // 연관관계 설정
    public void setTeam(Team team){
        this.team = team;
    }

    // Getter, Setter ..

}
```

팀 엔티티에 Member list 를 추가했습니다. 그리고 일대다 관계를 매핑하기 위해 @OneToMany 를 설정했습니다.

```java
@Entity
public class Team {

    @Id
    @Column (name = "TEAM_ID")
    private String id;

    private String name;

    @OneToMany (mappedBy = "team")
    private List<Member> memberList = new ArrayList<Member>();

    // Getter, Setter ..
    
}
```

#### 4. 연관관계의 주인

엔티티를 양방향 연관관계로 설정하면 객체의 참조는 둘인데, 외래 키는 하나입니다. 이런 차이로 인해 JPA 는 두 객체 연관관계 중 하나를 정해서 테이블의 외래키를 관리해야 하는데 이것을 연관관계의 주인이라고 합니다.

##### 4.1 양방향 매칭의 규칙 : 연관관계의 주인

연관계의 주인만이 데이터베이스 연관관계와 매핑되고, 외래 키를 관리 할 수 있습니다. 주인이 아닌 쪽은 읽기만 가능합니다. 연관관계의 주인을 정한다는 것은 외래 키 관리자를 선택한다는 것입니다.

##### 4.2 연관 관계의 주인은 외래 키가 있는 곳

데이터베이스 테이블의 다대일, 일대다 관계에서는 항상 다 쪽이 외래키를 가집니다. 따라서, 다 쪽인 @ManyToOne 은 항상 연관계의 주인이므로 mappedBy 속성을 설정할 수 없습니다.

#### 5. 양방향 연관관계의 주의점

연관관계의 주인에는 값을 입력하지 않고, 주인이 아닌 곳에만 값을 입력하는 경우를 봅시다.

```java 
 public void save(){

        // 회원 1 저장
        Member member1 = new Member();
        em.persist(member1); // 연관관계 설정 member1 -> team1

        // 회원 2 저장
        Member member2 = new Member();
        em.persist(member2); // 연관관계 설정 member2 -> team1
        
        Team team1 = new Team();
        team1.setId("team1");
        team1.setName("팀1");
        
        //주인이 아닌 곳에만 연관관계 설정
        team1.getMemberList().add(member1)
        team1.getMemberList().add(member2)

    }
```

데이터베이스에서 회원테이블을 조회해 봅시다. 

```sql
SELECT * FROM MEMBER;

//결과
Memer_ID / USERNAME / TEAM_ID
member1 / 회원 1 / null
member2 / 회원 2 / null
```

##### 5.1 순수한 객체까지 고려한 양방향 연관관계

JPA를 사용하지 않고 순수한 객체로 테스트해 봅시다.

```java
 public void test(){

        Team team1 = new Team();
        team1.setId("team1");
        team1.setName("팀1");

        Member member1 = new Member();
        member1.setId("member1");
        member1.setUsername("회원1");

        Member member2 = new Member();
        member2.setId("member2");
        member2.setUsername("회원2");
        
        member1.setTeam(team1); // 연관 관계 설정 member1 -> team1
        member2.setTeam(team1); // 연관 관계 설정 member2 -> team1

        List<Member> memberList = team1.getMemberList();
        System.out.println("member size : " + memberList.size()); // 출력 결과 : 0
    }
```

회원 -> 팀 을 설정하면 다음 코드 처럼 반대 방향인 팀 -> 회원도 설정해야합니다.

```java
public void test(){

        Team team1 = new Team();
        team1.setId("team1");
        team1.setName("팀1");

        Member member1 = new Member();
        member1.setId("member1");
        member1.setUsername("회원1");

        Member member2 = new Member();
        member2.setId("member2");
        member2.setUsername("회원2");
        
        member1.setTeam(team1); // 연관 관계 설정 member1 -> team1
  			team1.getMemberList.add(member1); // 연관 관계 설정 team1 -> member1
  
        member2.setTeam(team1); // 연관 관계 설정 member2 -> team1
    		team1.getMemberList.add(member2); // 연관 관계 설정 team1 -> member2

        List<Member> memberList = team1.getMemberList();
        System.out.println("member size : " + memberList.size()); // 출력 결과 : 2
    }
```

JPA를 적용하면 다음과 같습니다.

```java
public void test(){

  			// 팀1 저장
        Team team1 = new Team();
        team1.setId("team1");
        team1.setName("팀1");
        em.persist(tema1);

        Member member1 = new Member();
        member1.setId("member1");
        member1.setUsername("회원1");
  
  			// 양방향 연관관계 설정
 		    member1.setTeam(team1); // 연관 관계 설정 member1 -> team1
  			team1.getMemberList.add(member1); // 연관 관계 설정 team1 -> member1
  			em.persist(member1);

        Member member2 = new Member();
        member2.setId("member2");
        member2.setUsername("회원2");
        
        // 양방향 연관관계 설정
 		    member2.setTeam(team1); // 연관 관계 설정 member2 -> team1
  			team1.getMemberList.add(member2); // 연관 관계 설정 team1 -> member2
  			em.persist(member2);
    }
```

##### 5.2 연관관계 편의 메서드

```java
member1.setTeam(team1); // 연관 관계 설정 member1 -> team1
team1.getMemberList.add(member1); // 연관 관계 설정 team1 -> member1
```

양방향 관계에서는 위 두 코드를, 다음과 같이 하나인 것처럼 사용하는 것이 안전합니다.

```java
public class Member{
  
  private Team team;
  
  public void setTeam(Team team){
    this.team = team;
    team.getMemberList().add(this);
  }
}
```

따라서 다음과 같이 양방향 관계를 설정할 수 있습니다.

```java
public void test(){

  			// 팀1 저장
        Team team1 = new Team();
        team1.setId("team1");
        team1.setName("팀1");
        em.persist(tema1);

        Member member1 = new Member();
        member1.setId("member1");
        member1.setUsername("회원1");
  
  			// 양방향 연관관계 설정
 		    member1.setTeam(team1); 
  			em.persist(member1);

        Member member2 = new Member();
        member2.setId("member2");
        member2.setUsername("회원2");
        
        // 양방향 연관관계 설정
 		    member2.setTeam(team1); // 연관 관계 설정 member2 -> team1
  			em.persist(member2);
    }
```

##### 5.3 연관관계 편의 메서드 작성시 주의 사항

```java
member.setTeam(teamA);
member.setTeam(teamB);
Member findMember = teaA.getMember(); //member1이 여전히 조회
```

이는, teamB로 변경할 때, teamA -> member 관계를 제거 하지 않았기 때문입니다. 따라서 편의 메서드를 다음과 같이 수정해야 합니다.

```java
public void setTeam(Team team){
  
  // 기존 팀과 관계를 제거
  if(this.team != null){
    this.team.getMemberList.remove(this);
  }
  
  this.team = team1; // 연관 관계 설정 member -> team
  team.getMemberList.add(this); // 연관 관계 설정 team -> member
}
```

#### Reference

자바 ORM 표준 프로그래밍 <김영한>