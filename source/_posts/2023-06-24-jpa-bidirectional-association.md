---
layout: post
title: JPA 양방향 연관관계
date: 2023-06-24
categories: JPA
---

JPA 의 양방향 연관관계에 대해 정리해보자.

## 예시: 회원, 팀

회원에서 팀으로 접근할 수 있고 팀에서 회원으로 접근할 수 있도록 양방향 연관관계로 매핑해보자.
객체 연관 관계는 다음과 같다.

1. 회원 -> 팀: Member.team
2. 팀 -> 회원: Team.members

데이터베이스 테이블의 관계는, 외래 키 하나로 양방향으로 조회할 수 있다.
team_id 외래 키를 사용해서 member join team 이 가능하고 반대로, team join member 도 가능하다.

## 양방향 연관관계 매핑

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
)
```

@OneToMany 의 mappedBy 속성은 양방향 매핑일 때 사용한다.
반대쪽 매핑의 필드 이름을 값으로 설절하면 된다.

## 일대다 컬렉션 조회

```kotlin
fun biDirection() {
    val team: Team = entityManager.find(Team::class.java, "team1")
    val members: List<Member> = team.members
    
    members.forEach{ member ->
        println(member)
    }
}
```

## 연과관계 주인

엔티티를 양방향 연관관계로 설정하면 객체의 참조는 둘이지만, 외래키는 하나이다.
그래서, 둘 사이에 차이가 발생한다.

이런 차이로 인해, JPA 에서는 두 객체 연관 관계 중 하나를 정해서 테이블의 외래키를 관리해야한다.
이것을 연관관계의 주인이라고 한다.

주인은 mappeddBy 속성을 사용하지 않는다. 
주인이 아니면, mappeddBy 속성을 사용해서 속성의 값으로 연관관계의 주인을 지정해야한다.

즉, 연관관계의 주인만 데이터베이스 연관관계와 매핑되고 외래 키를 관리할 수 있다.
주인이 아닌 반대편은 읽기만 가능하고 외래키를 변경하지 못한다.

## 양방향 연관관계 저장

```kotlin
fun main() {
    // 팀1 저장
    val team1 = Team(
        id = "team1",
        name = "팀1"
    )
    entityManager.persist(team1)

    // 회원1 저장
    val member1 = Member(
        id = "member1",
        username = "회원1"
    )
    member1.setTeam(team1)
    entityManager.persist(member1)

    // 회원2 저장
    val member2 = Member(
        id = "member2",
        username = "회원2"
    )
    member2.setTeam(team1)
    entityManager.persist(member2)
}
```

양방향 연관관계는 연관관계의 주인이 외래 키를 관리하기 때문에, 주인이 아닌 방향은 값을 설정하지 않아도 데이터베이스에 외래 키 값이 정상 입력된다.
즉, 아래 코드는 데이터베이스 저장할 때 무시된다.

```kotlin
team1.members.add(member1)
team1.members.add(member2)
```

## 순수한 객체까지 고려한 양방향 연과관계

객체 관점에서 양쪽 방향 모두에 값을 입력해주는 것이 안전하다.
양쪽 방향 모두에 입력하지 않으면, JPA 를 사용하지 않는 순수한 객체 상태에 문제가 발생할 수 있다.

```kotlin
fun 순수한_객체_양방향(){
    val team1 = Team(
        id = "team1",
        name = "팀1"
    )
    
    val member1 = Member(
        id = "member1",
        username = "회원1"
    )
    member1.setTeam(team1)
    
    val member2 = Member(
        id = "member2",
        username = "회원2"
    )
    member2.setTeam(team1)

    val members = team1.members
    println(members.size) // 0
}
```

출력 결과는 0 이 나오는데, 이것은 기대하는 결과가 아니다.
따라서, 회원->팀을 설정했으면 반대방향인 팀->회원도 설정해야한다.

```kotlin
team1.members.add(member1)
team1.members.add(member2)
```

## 연관관계 편의 메서드

양방향 연관관계는 결국 양쪽 다 신경써야한다. 하지만, 실수로 둘 중 하나만 호출해서 양방향이 깨질 수 있다.

```kotlin
member.setTeam(team)
team.members.add(member)
```

그래서, 다음처럼 위 코드를 하나인 것 처럼 사용가능하다.

```kotlin
@Entity
class Member(
    @Id
    @Column(name = "MEMBER_ID")
    val id: String,

    val username: String,

    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    var team: Team?= null
) {

    fun setTeam(team: Team) {
        this.team = team
        team.members.add(this) // HERE
    }
}
```

## 연관관계 편의 메서드 주의사항

위 setTeam() 은 버그가 있다.

```kotlin
member1.setTeam(teamA)
member1.setTeam(teamB)

teamA.members // 여전히 member1 이 조회된다
```

그래서, 연관관계를 변경할 때는 기존 팀이 있으면 기존 팀과 회원의 연관관계를 삭제하는 코드가 추가되어야한다.

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

        // HERE
        if (this.team != null) {
            this.team!!.members.remove(this)
        }

        this.team = team
        team.members.add(this)
    }
}
```

---

자바 ORM 표준 프로그래밍 <김영한>한
