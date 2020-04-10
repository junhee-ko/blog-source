---
layout: post
title:  "[오브젝트] 9장_유연한 설계"
date:   2019-12-15
categories: OOP
---

이번 장은, 8장에서 설명한 의존성 관리 기법들을 원칙이라는 관점에서 정리한다.

## 01 개팡-폐쇄 원칙

소프트웨어 개체(클래스, 모듈, 함수 ...) 는 확장에 열려있고, 수정에 닫혀 있어야한다.

1. 확장에 열려 있다

   요구사항이 변경될 때, 변경에 맞게 새로운 동작을 추가해서 애플리케이션의 기능을 확장할 수 있다.

2. 수정에 닫혀 있다

   기존의 코드를 수정하지 않고 애플리케이션의 동작을 추가하거나 변경할 수 있다.

##### 컴파일타임 의존성을 고정시키고 런타임 의존성을 변경하라

1. 런타임 의존성

   실행 시에 협력에 참여하는 객체들 사이의 관계

2. 컴파일 타임 의존성

   코드에서 드러나는 클래스들 사이의 관계

중복 할인 정책을 추가하기 위해 한 일은, DiscountPolicy 의 자식 클래스로 OverlappedDiscountPolicy 클래스를 추가한 것이다. 단순히 새로운 크래스를 추가하는 것만으로 Movie 를 새로운 컨텍스트에 사용되도록 확장할 수 있었다.

이 설계 방식은,

1. 새로운 할인 정책을 추가해서 새로운 기능을 확장할 수 있도록 허용한다. 즉, 확장에 열려 있다.
2. 기존의 코드를 수정할 필요 없이 새로운 클래스 추가만으로 새로운 할인 정책을 확장한다. 즉, 수정에 닫혀 있다.

의존성 관점에서 개방-폐쇄 원칙을 따르는 설계란, 컴파일 타임 의존성은 유지하면서 런타임 의존성의 가능성을 확장하고 수정할 수 있는 구조이다.

##### 추상화가 핵심이다

개방-폐쇄 원칙의 핵심은 추상화에 의존하는 것이다.

추상화를 통해 생략된 부분은 확장의 여지를 남긴다.

```java
publc abstract classs DiscoutPolicy{
  ...
   
  public Money calculateDiscountAmount(Screening screening){
    for(DiscountCondition each : conditions){
      if(each.isSatisfiedBy(screening)){ // 할인 여부를 판단하는 로직
        return getDiscountFee(screenig);
      }
    }
    
    return screeing.getMovieFee();
  }
  
  ...
}
```

위 코드에서 변하지않는 부분은, 할인 여부를 판단하는 로직이다.

변하는 부분은, 할인 요금을 계산하는 방법이다. 이 부분이 추상화를 통해 생략된 부분이다. 상속을 통해 생략된 부분을 구체화함으로써 할인 정책을 확장할 수 있다.

변하는 부분을 고정하고 변하지 않는 부분을 생략하는 추상화 메커니즘이 개방-폐쇄 원칙의 기반이 된다.

## 02 생성 사용 분리

```java
public class Movie {
  ...
    private DiscountPolicy discountPolicy;
  	
  	public Movie(String title, ...){
      ...
      this.discountPolicy = new AmountDiscountPolicy(...); // 객체 생성
    }
  
  	public Money cacluateMovieFee(Screening screening){
      return fee.minus(discountPolicy.calculateDiscountAmount(screenig)); // 객체 사용
    }
}
```

위 코드에서, Movie 의 할인 정책을 비율 할인 정책으로 변경할 수 있는 방법은, 직접 코드를 수정하는 것이다. 이것은 동작을 추가하거나 변경하기 위해 기존 코드를 수정하도록 만들어서, 개방-폐쇄 원칙을 위반한다.

문제는, 부적절한 곳에서 객체를 생성한다는 것이다. 동일한 클래스 안에서 객체 생성과 사용이라는 두 가지 다른 목적을 가진 코드가 공존한다는 것이다.

객체의 관련된 두 가지 책임을 서로 다른 객체로 분리해야한다. 즉, 객체애 대한 생성과 사용을 분리해야한다. 현재의 컨텍스트에 관한 결정권을 가지고 있는 클라이언트로 컨텍스트에 대한 지식을 옮겨 Movie 는 특적한 클라이언트에 결합되지 않고 독립될 수 있다.

##### FACTORY 추가하기

Movie 를 사용하는 Client 역시도, 특정한 컨텍스트에 묶이지 않기를 바란다고 가정하자.

객체 생성에 특화된 객체인, FACTORY 를 사용하면 된다.

```java
public class Factory {
  public Movie cacluateAvatarMovie() {
    return new Movie("아바타", ....);
  }
}

public class Client {
  private Factory factory;
  
  ...
}
```

##### 순수한 가공물에게 책임 할당하기

위의 FACTORY 는 도메인 모델에 속하지 않는다. FACTORY는 순수하게 기술적인 결정이다. 즉, 객체 생성 책임을 할당할 만한 도메인 객체가 존재하지 않을 때 선택한 PURE FABRICATION  이다.

시스템을 객체로 분해하는 방법으로 표현적 분해와 행위적 분해가 있다. 

1. 표현적 분해

   도메인에 존재하는 사물 또는 개념을 표현하는 객체들을 이용해 시스템을 분해하는 것이다.

2. 행위적 분해

   그런데, 실제로 동작하는 애플리케이션은 데이터베이스에 접근 위한 객체와 같이 도메인 개념을 초월하는 기계적인 개념도 필요하다. 이렇게 도메인과 무관한 인공적인 객체를 PURE FABRICATION (순수한 가공물) 이라고 한다. PURE FABRICATION은 특정한 행동을 표현하는 것이 일반적이다. 

## 03 의존성 주입

의존성 주입이란, 사용하는 객체가 아닌 외부의 독립적인 객체가 인스턴스를 생성한 후 이를 전달해서 의존성을 해결 하는 방법이다.

의존성 주입에서 의존성 해결 방법은 ,

1. 생성자 주입 : 객체 생성 시점에 생성자 통해 의존성 해결
2. setter 주입 : 객체 생성 후 세터 메서드로 의존성 해결
3. 메서드 주입 : 메서드 실행 시 인자로 의존성 해결

##### 숨겨진 의존성을 나쁘다

의존성 주입 외에, 의존성 해결 방법으로 SERVICE LOCATOR 패턴이 있다.

SERVICE LOCATOR 는, 의존성 해결할 객체들을 보관하는 일종의 저장소이다. 외부에서 객체에게 의존성을 전달하는 의존성 주입과 달리, 객체가 직접 SERVICE LOCATOR 에게 의존성을 해결해달라고 요청한다.

```java
public class Movie {
  ...
    private DiscountPolicy discountPolicy;
  	
  	public Movie(String title, ...){
      ...
      this.discountPolicy = ServiceLocator.discountPolicy();
    }
}

public class ServiceLocator {
  private static ServiceLocator soleInstance = new ServiceLocator();
  private DiscountPolicy discountPolicy;
  
  public static DiscountPolicy discountPolicy(){
    return soleInstance.discountPolicy;
  }
  
  public static void provide(DiscountPolicy discountPolicy){
    soleInstance.discountPolicy = discountPolicy;
  }
  
  private ServiceLocator {
    
  }
}

serviceLocator.provide(new AmountDiscountPolicy(...));
Movie avater = new Movie("아바타", ... );

```

Movie 는 DiscountPolicy 에 의존하고 있지만, Movie 의 퍼블릭 인터페이스 어디에도 의존성에 대한 정보가 표시되어 있지 않다. 의존성이 암시적이며 코드 깊숙히 숨겨져 있다. 

숨겨진 의존성은,

1. 이해하기 어렵고 디버깅이 어렵다. 문제점을 발견할 수 있는 시점이 코드작성 시점이 아니라 실행시점으로 미뤄지기 때문이다. 
2. 또한, 클래스 사용법을 익히기 위해서 구현 내부를 샅샅이 뒤져야한다.

핵심은, 의존성 주입이 SERVICE LOCATOR 패턴보다 좋다는 것이 아니다. 명시적인 의존성이 숨겨진 의존성보다 좋다는 것이다.

## 04 의존성 역전 원칙

##### 추상화와 의존성 역전

 ```java
public class Movie {
  private AmountDiscountPolicy discountPolicy; // 구체적인 방법, 하위 수준
}
 ```

위 설계가 변경에 취약한 이유는, 요금을 계산하는 상위 정책이 요금을 계산하는데 필요한 구체적인 방법에 의존하기 때문이다.

객체 사이의 협력이 존재하면, 그 협력의 본질을 담고 있는 것은 상위 수준의 정책이다. 상위 수준의 클래스가 하위 수준의 클래스에 의존하면, 

1. 하위 수준의 변경에 의해 상위 수준의 클래스가 영향을 받게 된다. 
2. 재사용이 어렵다. 상위 수준의 클래스를 재사용할 때 하위 수준의 클래스도 필요하기 때문이다.

가장 중요한 핵심은, 추상화에 의존해야한다는 것이다. 모든 의존성의 방향이 추상클래스나 인터페이스와 같은 추상화를 따라야한다. 즉,

1. 상위 수준의 모듈은 하위 수준의 모듈에 의존해서는 안된다. 둘 모두 추상화에 의존해야한다.
2. 추상화는 구체적인 사항에 의존해서는 안된다. 구체적인 사항은 추상화에 의존해야한다.

##### 의존성 역전 원칙과 패키지

![](/image/object_chapter9_01.png)

위 그림은, 인터페이스의 소유권을 클아이언트 모듈이 아닌 서버 모듈에 위치시켰다. Movie 를 다양한 컨텍스트에서 재사용하기 위해서는 불필요한 클래스들이 Movie 와 함께 배포되어야한다. 즉, DiscountPolicy 클래스에 의존하기 위해서는, 같은 패키지안에 있는 AmountDiscountPolicy 와 PercentDiscountPolicy 클래스도 함께 존재해야한다.

![](/image/object_chapter9_02.png)

위 그림은 인터페이스 소유권을 클라이언트에 위치시켰다. Movie 를 다른 컨텍스트에서 재사용하기 위해서는, 단지 Movie 와 DiscountPolicy 가 포함된 패키지만 재사용하면 된다.

## 05 유연성에 대한 조언

##### 유연한 설계는 유연성이 필요할 때만 옳다

유연한 설계는 복잡하다. 유연하지 않은 설계는 단순하다. 

유연성을 코드를 읽는 사람들이 복잡함을 수용할 때만 가치가 있다.

##### 협력과 책임이 중요하다

역할, 책임, 협력에 집중하라.

객체를 생성하는 방법은 모든 책임이 자리를 잡은 후 마지막 시점에 내리는 것이 적절하다.

---

오브젝트 <조영호>
