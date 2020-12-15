---
layout: post
title:  "[자바 ORM 표준 JPA 프로그래밍] 9장_값 타입"
date:   2019-04-11
categories: JPA
---

JPA의 데이터 타입은 엔티티 타입과 값 타입으로 나눌 수 있습니다. 값 타입은 다음 세 가지로 나눌 수 있습니다.

- 기본값 타입
  - 자바 기본 타입 : int, double ...
  - 래퍼 클래스 : Integer ...
  - String
- 임베디드 타입
- 컬렉션 값 타입

#### 9.1 기본값 타입

```java
@Entity
public class Member{
  @Id @GeneratedValue
  private Long id; // 기본값 타입
  private String name; // 기본값 타입
  private int age; // 기본값 타입
}
```

#### 9.2 임베디드 타입 (복합 값 타입)

새로운 값 타입을 직접 지정해서 사용할 수 있습니다. 이것을 JPA 에서는 임베디드 타입이라고 합니다.

```java
@Entity
public abstract class Member{ 
  
  @Id @GeneratedValue
  private Long id;
  private String name;
  
  @Embedded  // 값 타입을 사용하는 곳에 표시
  Period workPeriod;
  
  @Embedded 
  Address homeAddress;
}

@Embeddable // 값 타입을 정의하는 곳에 표시
public class Period{
  @Temporal (TemporalType.DATE) java.util.Date startDate;
  @Temporal (TemporalType.DATE) java.util.Date endDate;
  
  public boolean isWork(Date date){
    ...
  }
}

@Embeddable // 값 타입을 정의하는 곳에 표시
public class Address{

}
```

##### 9.2.1 임베디드 타입과 테이블 매핑

임베디드 타입은 엔티티의 값일 뿐입니다. 따라서, 값이 속한 엔티티의 테이블에 매핑합니다.

##### 9.2.2 임베디드 타입과 연관관계

엔티티는 공유 될수 있으므로 참조한다고 표현하고, 값 타입은 특정 주인에게 소속되고 개념상 공유되지 않으므로 포함한다고 합니다.

##### 9.2.3 @AttributeOverride : 속성 재정의

```java
@Entity
public abstract class Member{ 
  
  @Id @GeneratedValue
  private Long id;
  private String name;
  
  @Embedded 
  Address homeAddress;
  
  @Embedded 
  Address companyAddress;
}
```

위 코드는 칼럼명이 중복된다는 문제가 있습니다. 이를 다음과 같이 해결합니다.

```java
@Entity
public abstract class Member{ 
  
  @Id @GeneratedValue
  private Long id;
  private String name;
  
  @Embedded 
  Address homeAddress;
  
  @Embedded
  @AttributeOverrides({
    @AttributeOverride(name = "city", column = @Column(name = "COMPANY_CITY")),
    @AttributeOverride(name = "street", column = @Column(name = "COMPANY_STREET")),
    @AttributeOverride(name = "zipcode", column = @Column(name = "COMPANY_ZIPCODE")),
  })
  Address companyAddress;
}
```

보다시피 코드의 중복도가 높아집니다. 한 엔티티에 같은 임베디드 타입을 중복해서 사용하는 일은 많지 않습니다.

##### 9.2.4 임베디드 타입과 NULL

임베디드 타입이 null 이면 매핑한 칼럼 값은 모두 null 이 됩니다.

#### 9.3 값 타입과 불변 객체

##### 9.3.1 값 타입 공유 참조

```java
member1.setHomeAddress(new Address("city1"));
Address address = member1.getHomeAddress();

address.setCity("city2"); //공유해서 사용
member2.setHomeAddress(address);
```

위 코드는 회원2의 주소만 바뀌길 기대하지만, 회원1의 주소도 바뀌게 됩니다. 이렇듯, 뭔가를 수정했는데 전혀 예상하지 못한 곳에서 문제가 발생하는 것을 Side Effect 라고 합니다. 이를 막기 위해, 값을 복사 해서 사용해야합니다.

##### 9.3.2 값 타입 복사

```java
member1.setHomeAddress(new Address("city1"));
Address address = member1.getHomeAddress();

//복사
Address newAddress = address.clone();

newAddress.setCity("city2"); //공유해서 사용
member2.setHomeAddress(newAddress);
```

위의 코드 처럼 복사해서 사용하면 Side Effect 문제는 해결하지만, 임베디드 타입 처럼 직접 정의한 값 타입은 자바의 기본 타입이 아니라 객체 타입이란 것입니다.

```java
int a = 10;
int b = a; //기본 타입은 항상 값을 복사
b = 4;
```

```java
Address a = new Address("old");
Address b = a; //a와 b는 같은 인스턴스를 공유 참조
b.setCity("New");
```

복사하지 않고 원본의 참조 값을 직접 넘기는 것을 막을 방법은 없습니다. 자바는 기본 타입이면 값을 복사해서 넘기고, 객체는 참졸르 넘길 뿐이기 때문입니다. 따라서, 객체의 공유 참조는 피할 수 없습니다. 이를 위해, 객체의 값을 수정하지 못하게 막아야합니다.

##### 9.3.3 불변 객체

한 번 만들면 절대 수정할수 없는 객체를 불변 객체라고 합니다.

```java
@Embeddable
public abstract class Address{ 
  
	private String city;
	
	protected Address() {}
	
	public Address(String city){
    this.city = city;
	}
	
	public String getCity(){
    return city;
	}
	
	//수정자 없음
}
```

#### 9.4 값 타입의 비교

- 동일성 비교
  - 인스턴스의 참조 값을 비교
  - == 사용
- 동등성 비교
  - 인스턴스의 값을 비교
  - equlas() method 사용

값 타입은 그 안에 값이 같으면 같은 것으로 봐야합니다. 따라서 동등성 비교를 해아합니다. 

#### 9.5 값 타입 컬렉션

```java
@Entity
public class Member{
  
  ...
  
  @ElementCollection
  @CollectionTable(name = "ADDRESS", joinColums = @JoinColumn(name = "MEMBER_ID"))
  @Column(name = "FOOD_NAME")
  private List<Address> addressHistory = new ArrayList<Address>();
  
  ...
}
```

값 타입은 식별자가 없는 단순한 값들의 모임이기 때문에, 값을 변경하면 데이터베이스에 저장된 원본 데이터를 찾기 어렵습니다. 값 타입 컬렉션에 보관된 값 타입들은 별도의 테이블에 보관됩니다. 따라서 여기에 보관된 값 타입의 값이 변경되면 데이터베이스에 있는 원본 데이터를 찾기 어렵습니다.
이런 문제로, JPA구현체들은 값 타입 컬레션에 변경 사항이 발생하면, 값 타입 컬렙션이 매핑된 테이블의 연관된 모든 데이터를 삭제하고 현재 값 타입 컬렉션 객체에 있는 모든 값을 데이터베이스에 다시 저장합니다. 