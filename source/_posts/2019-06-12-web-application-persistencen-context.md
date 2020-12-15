---
layout: post
title:  "[자바 ORM 표준 JPA 프로그래밍] 13장_웹 어플리케이션과 영속성 관리"
date:   2019-06-12
categories: JPA
---

이번 장에서는 컨테이너 환경에서 JPA 가 동작하는 내부 동작 방식을 이해하고, 발생할 수 있는 문제점과 해결방안을 정리합니다.

#### 13.1 트렌젝션 범위의 영속성 컨텍스트

스프링이나 J2EE 컨테이너 환경에서 JPA 를 사용하면 컨테이너가 제공하는 전략을 따라야합니다.

##### 13.1.1 스프링 컨테이너의 기본 전략

스프링 컨테이너는 트렌젝션 범위의 영속성 컨텐스트 전략을 기본으로 합니다. 즉, 트렌젝션을 시작할 때 영속성 컨텍스트를 생성하고 끝날 때 영속성 컨텍스트를 종료합니다. 그리고, 같은 트렌젝션 안에서는 항상 같은 영속성 컨텍스트에 접근합니다. 

```java
@Controller
class HelloController{
  
  @Autowired HelloService helloService;
  
  public void hello(){
    //반환된 member 엔티티는 준영속 상태
    Member member = helloService.logic();
  }
}

@Service
class HelloService{
  
  // 엔티티 메니저 주입
  @PersistenceContext
  EntityManager em;
  
  @Autowired Repository1 repository1;
  @Autowired Repository2 repository2;
  
  //트랜잭션 시작
  @Transactional
  public void logic(){
    repository1.hello();
    
    //Member 는 영속상태
    Member member = repository2.findMember();
    
    return member;
  }
  //트렌젝션 종료
}

@Repository
class Repository1 {
  
  @PersistenceContext
  EntityManager em;
  
  public void hello(){
    em.xxx(); //영속성 컨텍스트 접근
  }
}

@Repository
class Repository2 {
  
  @PersistenceContext
  EntityManager em;
  
  public Member findMember() {
    return em.find(Member.class, "id1"); //영속성 컨텍스트 접근
  }
}
```

#### 13.2 준영속 상태와 지연 로딩

조회한 엔티티가 서비스와 리포지토리 계층에서는 영속성 컨텍스트에 관리되면서 영속 상태를 유지하지만, 컨트롤러나 뷰 같은 프리젠테이션 계층에서는 준영속 상태가 됩니다. 따라서, 변경감지와 지연로딩이 동작하지 않습니다.

```java
@Entity
public class Order{
  @Id @GeneratedValue
  private Long id;
  
  @ManyToOne(fetch = FetchType.LAZY) //지연로딩
  private Member member; //주문 회원
}

class OrderController {
  public String view(Long orderId){
    Order order = orderService.findOne(orderId);
    Member member = order.getMember();
    member.getName(); //지연로딩 시 예외 발생
  }
}
```

- 준영속 상태와 변경 감지

변경감지 기능이 프리젠테이션 계층에서 동작하지 않는것은 문제가 되지 않습니다. 변경 감지 기능이 프리젠테이션 계층에서도 동작하면 애플리케이션 계층이 가지는 책임이 모호해지고, 데이터를 어디서 어떻게 변경했는지 프리젠테이션 계층까지 다 찾아야 하므로 유지보수하기 어렵습니다. 비즈니스 로직은 서비스 계층에서 끝내야합니다.

- 준영속 상태와 지연 로딩

준영속 상태의 지연 로딩을 해결하는 방법은 두 가지 입니다. 

1. 뷰가 필요한 엔티티를 미리 로딩
2. OSIV

뷰가 필요한 엔티티를 미리 로딩하는 방법은 어디서 미리 로딩 하느냐에 따라 세가지가 있습니다.

1. 글로벌 페치 전략 수정
2. JPQL Fetch Join
3. 강제 초기화

##### 13.2.1 글로벌 페치 전략 수정

```java
@Entity
public class Order{
  @Id @GeneratedValue
  private Long id;
  
  @ManyToOne(fetch = FetchType.EAGER) //즉시 로딩 전략
  private Member member; //주문 회원
}

//Presentation Logic
class OrderController {
  public String view(Long orderId){
    Order order = orderService.findOne(orderId);
    Member member = order.getMember();
    member.getName(); //이미 로딩된 엔티티
  }
}
```

글로벌 페치 전략에 즉시 로딩 사용시 단점은 두가지가 있습니다.

1. 사용하지 않는 엔티티를 로딩
order 를 조회하면서 사용하지 않는 member 도 함께 조회

2. N+1 문제

```java
Order order = em.find(Order.class, 1L);

//실행된 SQL
select o.*, m.*
from Order o
left outer join Member m on o.MEMBER_ID = m.MEMBER_ID
where o.id = 1
```

여기까지 보면 글로벌 즉시 로딩 전략이 좋아보이지만, 문제는 JPQL 을 사용할 때 발생합니다.

```java
List <Order> orders = em.createQuery("select o from Order o", Order.class).getResultList();

//실행된 SQL
select * from Order //JPQL 로 실행된 SQL
select * from Member where id = ? //EAGER 로 실행된 SQL
select * from Member where id = ? //EAGER 로 실행된 SQL
select * from Member where id = ? //EAGER 로 실행된 SQL
select * from Member where id = ? //EAGER 로 실행된 SQL
...
```

JPA 가 JPQL 을 분석해서 SQL 을 생성할 때, 글로벌 패치 전략을 참고하지 않고 오직 JPQL 자체만 사용합니다. 따라서, 즉시로딩이든 지연 로딩이등 구분하지 않고 JPQL 쿼리 자체에 충신한 SQL 을 만듦니다.
이런 N+1 문제는 이어서 소개할 JPQL Fetch Join 으로 해결할 수 있습니다.

##### 13.2.2 JPQL Fetch Join

```java
//Fetch Join 사용 전
JPQL : select o from Order o
SQL : select * from Order

//Fetch Join 사용 후
JPQL :
	select o 
	from Order o
	join fetch o.member
	
SQL :
	select o.*, m.*
	from Order o
	join Member m on o.MEMBER_ID = m.MEMBER_ID
```

Fetch Join 을 사용하면, SQL JOIN 을 사용해서 페치 조인 대상까지 함께 조회합니다. N+1 문제가 발생하지 않습니다.

- JPQL Fetch Join 의 단점
무분별하게 사용하면 화면에 맞춘 리포지토리 메소드가 증가할 수 있습니다. 결국 프리젠테이션 계층이 데이터 접근 계층을 침범하는 것입니다.

##### 13.2.3 강제로 초기화

영속성 컨테스트가 살아있을 때 프리젠테이션 계층이 필요한 엔티티를 강제로 초기화해서 반환하는 방법입니다.

```java
class OrderService{
  
  @Transactional
  public Order findOrder(id){
    Order order = oderRepository.findOrder(id);
    order.getMember().getName(); //프록시 객체를 강제로 초기화
    return order;
  }
}
```

글로벌 페치 전략을 지연로딩으로 설정하면, 연관된 엔티리를 실제 엔티티가 아닌 프록시 객체로 조회합니다. 프록시 객체는 실제 사용하는 시점에 초기화됩니다. order.getMember() 까지만 호출하면. 단순히 프록시 객체만 반환합니다. 아직 초기화 하지 않습니다. member.getName() 처럼 실제 값을 사용할 때 초기화됩니다.
프록시 초기화 하는 역하을 서비스 계층이 담당하면, 뷰가 필요한 엔티티에 따라 서비스 계층의 로직을 변경해야 합니다. 비즈니스 로직을 담당하는 서비스 계층에서 프리젠테이션 계층을 위한 프록시 초기화 역할을 하는 FACADE 계층이 그 역할을 담당해줍니다.

##### 13.2.4 FACADE 계층 추가

- 프리젠테이션 계층과 도메인 모델 계층간의 논리적 의존성을 분리합니다.
- 프리젠테이션 계층에서 필요한 프록시 객체를 초기화합니다.
- 서비스 계층을 호출해서 비즈니스 로직을 실행합니다.
- 리포지토리를 직접 호출해서 뷰가 요구하는 엔티티를 찾습니다.

```java
class OrderFacade{
	@Autowired OrderService orderService;
  
  public Order findOrder(id){
    Order order = orderService.findOrder(id);
    
    //프리젠테이션 계층이 필요한 프록시 객체를 강제 초기화
    order.getMember().getName();
    return order;
  }
}

class OrderService{
  public Order findOrder(id){
    return orderRepository.findOrder(id);
  }
}
```

#### 13.3 OSIV (Open Session in View)

영속성 컨텍스트를 뷰까지 열어둔다는 뜻입니다. 

##### 13.3.1 과거 OSIV : 요청 당 트렌젝션

OSIV 의 핵심은 뷰에서도 지연로딩이 가능하도록 하는 것입니다. 가장 단순한 구현은 클라이언트의 요청이 들어오자마자 서플릿 필터나 스프링 인터셉터에서 트렌젝션을 시작하고 요청을 끝날 때 트렌젝션도 끝내는 것입니다. 이것을 요청 당 트렌젝션 방식의 OSIV 라고 합니다.
문제는, 프레센테이션 계층이 엔티티를 변경할 수 있다는 것입니다. 

```java
class MemberControlelr{
	public String viewMember(Long id){
    Member member = memberService.getMember(id);
    member.setName("XXX");
    model.addAttribute("member", member);
  }
}
```

개발자의 의도는 단순히 뷰에 노출할 때만 고객이름을 XXX 로 변경하고 싶은 것이지, 데이터베이스에 있는 고객 이름까지 변경하고자 하는 것이 아니었습니다. 하지만 요청당 트렌젝션 방식은 뷰 렌더링 이후에 트렌젝션으 커밋합니다. 커밋을 하면 영속성 컨텍스트를 플러쉬합니다. 영속성 컨텍스트의 변경 감지 기능이 동작해서 변경된 엔티티를 데이터베이스에 반영해버립니다.
따라서, 프레젠테이션 계층에서 엔티티를 수정하지 못하게 해야합니다.

- 엔티티를 읽기 전용 인터페이스로 제공
- 엔티티 레핑
- DTO 만 반환

###### 엔티티를 읽기 전용 인터페이스로 제공

```java
interface MemberView{
  public String getName();
}

@Entity
class Member implements MemberView{
  ...
}

class MemberService {
  public MemberView getMember(id){
    return memberRepository.findById(id);
  }
}
```

###### 엔티티 레핑

엔티티의 읽기 전용 메소드만 가지고 있는 엔티티를 감싼 객체를 만들고, 이것을 프리젠테이션 계층에 반환하는 방법입니다.

``` java
class MemberWrapper{
  private Member member;
  
  public MemberWrapper(member){
    this.member = member;
  }
  
  // 읽기 전용 메소드만 제공
  public String getName(){
    member.getName;
  }
}
```

###### DTO 만 반환

```java
class MemberDTO {
	private STring name;
  
  //GETTER, SETTER
}

...
MemberDTO memberDTO = new MemberDTO();
memberDTO.setName(member.getName());
return memberDTO;
```

최근에는 비즈니스 계층에서만 트렌젝션을 유지하는 방식의 OSIV 를 사용합니다. 스프링 프레임워크가 제공하는 OSIV 방식입니다.

##### 13.3.2 스프링 OSIV : 비즈니스 계층 트렌젝션

클라이언트의 요청이 들어오면 영속성 컨텍스트를 생성합니다. 이 때, 트렌젝션을 시작하지 않습니다. 서비스 계층에서 트렌젝션을 시작하면 앞에서 생성해둔 영속성 켄텍스트에 트렌젝션을 시작합니다. 비즈니스 로직을 실행하고 서비스 계층이 끝나면 트렌젝셩르 커밋하면서 영속성 컨텍스트를 플러쉬합니다. 이때, 트렌젝션만 종료하고 영속성 컨텍스트를 살려둡니다. 클라이언트의 요청이 끝날 때 영속성 컨텍스트를 종료합니다.
엔티티를 변경하지 않고 단순히 조회만 할 때는 트렌젝션이 없어도 되는데, 이것을 트렌젝션 없이 읽기라고 합니다. 영속성 컨텍스트는 트렌젝션 범위 안에서 엔티티를 조회하고 수정할 수 있습니다. 영속성 컨텍스트는 트렌젝션 범위 밖에서 엔티리를 조회만 할 수 있습니다.