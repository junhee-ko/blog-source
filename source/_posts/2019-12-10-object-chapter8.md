---
layout: post
title:  "[오브젝트] 8장_의존성 관리하기"
date:   2019-12-10
categories: OOP
---

이번장에서는 충분히 협력적이고 유연한 객체를 만들기 위해, 의존성을 관리하는 방법을 정리한다.

## 01 의존성 이해하기

##### 변경과 의존성

어떤 객체가 협력을 위해 다른 객체가 필요할 때, 두 객체 사이의 의존성이 존재한다.
아래 코드에서, 어떤 형태로든 DayOfWeek, LocalTime, Screening, DiscountCondition 이 변경되면 PeriodCondition 도 함께 변경될 수 있다. 

```java
public class PeriodCondition implements DiscountCondition {
  private DayOfWeek dayOfWeek;
  private LocalTime startTime;
  private LocalTime endTime;
  ...
    
  public boolean isSatisfiedBy(Screening screening){
    ...
  }
}
```

##### 의존성 전이

![](/image/object_dependency.png)

PeriodCondition 이 Screening 에 의존하면, PeriodCondition 은 Screening 이 의존하는 대상에 대해서도 의존하게 된다는 것이다.
의존성이 실제로 전이 될지 여부는 변경의 방향과 캡슐화의 정도에 따라 다르다. Screening 이 내부 구현을 효과적으로 캡슐화하면 Screening 에 의존하고 있는 PeriodCondition 까지는 변경이 전파되지 않는다.
의존성의 종류는,

1. 직접 의존성
   한 요소가 다른 요소에 직접 의존하는 경우. PeriodCondition 이 Screening 에 의존

2. 간접 의존성
   의존성 전이에 의해 영향이 전파되는 경우

##### 런타임 의존성과 컴파일타임 의존성

1. 런타임
   애플리케이션이 실행되는 시점

2. 컴파일 타임
   작성된 코드를 컴파일 하는 시점. 문맥에 따라서는, 코드 그 자체

```java
public class Movie {
  ...
    private DiscountPolicy discountPolicy;
  	
  	public Movie(String title, ... , DiscountPolicy discountPolicy){
      ...
      this.discountPolicy = discountPolicy;
    }
  
  	public Money calcuateMovieFee(Screening screening){
      return fee.minus(discountPolicy.caculateDiscountAmount(screening));
    }
}
```

코드를 작성하는 시점에서 Movie 클래스는 AmountDiscountPolicy 클래스와 PercentDiscountPolicy 의 존재를 모르지만, 실행 시점의 Movie 인스턴스는  AmountDiscountPolicy 인스턴스와 PercentDiscountPolicy 인스턴스와 협력할 수 있어야한다.
협력할 인스턴스의 구체적인 클래스를 알면 안된다. 실제로 협력할 객체가 어떤 것인지는 런타임에 해결해야한다.
핵심 : 유연하고 재사용 가능한 설계를 만들기 위해서는 동일한 소스 코드 구조로 다양한 실행 구조를 만들 수 있어야한다.

##### 컨텍스트 독립성

클래스가 특정 문맥에 강하게 결합되면 다른 문맥에서 사용하기 어렵다. 클래스가 사용될 특정 문맥에 대해 최소한의 가정으로만 이뤄져 있다면 다른 문맥에서 사용하기 쉽다. 이것이 컨텍스트 독립성이다.

##### 의존성 해결하기

의존성 해결이란, 컴파일 타임 의존성을 실행 컨텍스트에 맞는 적절한 런타임 의존성으로 교체하는 것이다.

해결 방법으로,

1. 객체 생성 시점에, 생성자로 의존성 해결

   ```java
   Movie avater = new Movie("아바타", new AmountDiscountPolicy(...), ....);
   ```

2. 객체 생성 후, setter 메서드를 통해 의존성 해결

   ```java
   Movie avater = new Movie(...);
   avater.setDiscountPolicy(new AmountDiscountPolicy(...));
   ```

   장점은, 실행 시점에 대상을 변경할수 있어서 설계가 유연하다.
   단점은, 객체를 생성하고 의존 대상 설정 전까지는 객체의 상태가 불완전할 수 있다.

   ```java
   Movie avater = new Movie(...);
   avatar.calculatFee(...); //NPE 예외 발생
   avater.setDiscountPolicy(new AmountDiscountPolicy(...));
   ```

3. 생성자 방식과 setter 방식 혼합

   ```java
   Movie avater = new Movie("아바타", new AmountDiscountPolicy(...), ....);
   avater.setDiscountPolicy(new PercentDiscountPolicy(...));
   ```

4. 메서드 실행 시 인자를 이용해 의존성 해결

   ```java
   public class Movie{
     public Money calulateMovieFee(Screening screening, DiscountPolicy discoutPolicy){
       ...
     }
   }
   ```

   협력 대상에 지속적으로 의존 관계 맺을 필요 없이, 메서드가 실행되는 동안 일시적으로 의존관계가 존재해도 되거나, 메서드 실행 시점에 매번 의존 대상이 변경되는 경우 유용하다.

## 02 유연한 설계

##### 의존성과 결합도

바람직한 의존성이란, 

1. 컨텍스트에 독립적이고 다양한 환경에서 재사용될 수 있는 가능성을 열어놓은 것이다.
2. loose coupling, weak coupling

##### 지식이 결합을 낳는다.

한 요소가 다른 요소에 대해 많은 정보를 알면 알수록, 두 요소는 강하게 결합된다.

1. Movie 클래스가 PercentDiscountPolicy 에 직접 의존하면, Movie 는 협력할 객체가 비율 할인 정책에 따라 할인 요금을 계산할 것이라는 걸 알아야 한다.
2. Movie 클래스가 추상 클래스인 DiscoutPolicy 에 의존하면, 구체적인 계산 방법은 알 필요 없이 할잉ㄴ 요금을 계산한다는 사실만 알면 된다.

결합도를 느슨하게 만들기 위해선, 협력 대상에 대해 필요한 정보 외에는 최대한 감추어야한다. 방법은 추상화이다.

##### 추상화에 의존하라

의존하는 대상이 추상적일 수록, 결합도는더 낮아진다.
아래로 갈수록, 클라이언트가 알아야하는 지식의 양이 적어져 결합도가 느슨해진다.

- 구체 클래스 의존성
- 추상 클래스 의존성
- 인터페이스 의존성

구체 클래스에 비해, 추상 클래스는 메서드의 내부 구현과 자식 클래스의 종류에 대한 지식을 클라이언트에게 숨길 수 있다.
추상 클래스에 비해, 인터페이스는 상속 계층을 모르더라도 협력이 가능하다.

##### 명시적인 의존성

"숨겨진 의존성을 밝은 곳으로 드러내서 널리 알리자"

1. 숨겨진 의존성

```java
public class Movie {
  ...
    private DiscountPolicy discountPolicy;
  	
  	public Movie(String title, ...){
      ...
      this.discountPolicy = new AmountDiscountPolicy(...);
    }
}
```

위 코드는, 생성자에서 구체 클래스인 AmountDiscountPolicy 의 인스턴스를 직접 생성해서 대입하고 있다. Movie 는 추상 클래스인 DiscountPolicy 뿐만 아니라, AmountDiscountPolicy 에도 의존하고 있다.
Movie 가 DiscountPolicy 에 의존하고 있다는 것을 감춘다. 이것이 숨겨진 의존성이다.
의존성 파악을 위해 내부 구현을 직접 살펴봐야한다. 그리고, 의존성이 명시적이지 않으면 클래스를 다른 컨텍스트에서 재사용하기 위해 내부 구현을 직접 변경해야한다.

2. 명시적인 의존성

```java
public class Movie {
  ...
    private DiscountPolicy discountPolicy;
  	
  	public Movie(String title, ... , DiscountPolicy discountPolicy){
      ...
      this.discountPolicy = discountPolicy;
    }
}
```

위 코드는, 생성자 안에서 직접 인스턴스를 생성하지 않고 생성자의 인자로 선언한다.
Movie 가 DiscountPolicy에 의존하고 있다는 것을 Movie 의 퍼블릭 인터페이스로 드러내고 있다. 이것이 명시적인 의존성이다.

##### new 는 해롭다

결합도 측면헤서 해로운 이유는,

1. new 연산자 사용 위해 구체 클래스의 이름을 직접 기술해야한다. 그래서, new 를 사용하는 클라이언트는 구체 클래스에 의존을 해서 결합도가 높아진다.
2. 구체 클래스의 어떤 인자를 사용해서 클래스의 생성자를 호출해야하는지도 알아야한다. 그래서, 클라이언트가 알아야하는 지식의 양이 늘어나 결합도가 높아진다.

```java
public class Movie {
  ...
    private DiscountPolicy discountPolicy;
  	
  	public Movie(String title, ... ){
      ...
      this.discountPolicy = new AmountDiscountPolicy(....)
    }
}
```

AmountDiscountPolicy 인스턴스 생성을 위해, 생성자에 전달해야하는 인자를 알아야한다.
해결방법은, 인스턴스 생성 로직과 사용 로직을 분리한다. 즉, Movie 는 외부로부터 이미 생성된 AmountDiscountPolicy 의 인스턴스를 전달받아야한다.
외부에서 인스턴스를 전달받는 방법은,

1. 생성자의 인자
2. setter 메서드
3. 메서드의 인자

```java
public class Movie {
  ...
    private DiscountPolicy discountPolicy;
  	
  	public Movie(String title, ... , DiscountPolicy discountPolicy){
      ...
      this.discountPolicy = discountPolicy;
    }
}
```

AmountDiscountPolicy 생성 책임은 Movie 의 클라이언트로 옮겨지고, Movie 는 AmountDiscountPolicy 인스턴스를 사용하는 책임만 남는다.

##### 가끔은 생성해도 무방하다

주로 협력하는 기본 객체를 설정하고 싶으면 가끔은 생성해도 무방하다.
설계는 트레이드오프다.

```java
public class Movie {
  ...
    private DiscountPolicy discountPolicy;
  
  	public Movie(String title, ... ){
			this(title, ..., new AmountDiscountPlicy(...));
    }
  	
  	public Movie(String title, ... , DiscountPolicy discountPolicy){
      ...
      this.discountPolicy = discountPolicy;
    }
}
```

##### 표준 클래스에 대한 의존은 해롭지 않다

변경될 확률이 거의 없으면, 의존성이 문제가 되지 않는다.
ex) JDK 에 포함된 표준 클래스

##### 컨텍스트 확장하기

1. NoneDiscountPolicy
   할인 정책이 존재 하지 않는다는 사실을 할인 정책의 한 종류로 간주

2. OverlappedDiscountPolicy
   중복 할인 정책을 표현하기 위함

설계를 유연하게 할 수 있었던 이유는,

1. DiscountPolicy 라는 추상화에 의존
2. 생성자를 통해 DiscountPolicy 에 대한 의존성을 명시적으로 드러냄
3. new 와 같이 구체 클래스를 직접 다뤄야하는 책임을 Movie 외부로 옮김

##### 조합 가능한 행동

훌륭한 객체 지향 설계는, 어떻게 하는지 표현하는 것이 아니라 객체들의 조합을 선언적으로 표현해서 객체들이 무엇을 하는지 표현하는 설계다.

---

오브젝트 <조영호>
