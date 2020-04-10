---
layout: post
title:  "[오브젝트] 부록 B 타입 계층의 구현"
date:   2020-02-10
categories: OOP
---

타입과 타입 계층을 구현할 수 있는 방법들을 정리하자.

##### 클래스를 이용한 타입 계층 구현

타입은 객체의 퍼블릭 인터페이스이다. 클래스는 객체의 타입과 구현을 동시에 정의한 것이다.

Phone 클래스가 있다. Phone 의 인스턴스는 calculateFee 메시지를 수신할 수 있는 퍼블릭 메서드를 구현한다. 타입은 퍼블릭 인터페이스이기 때문에, Phone 클래스는 calculateFee 메세지에 응답할 수 있는 타입을 선언한 동시에 객체 구현을 정의한 것이다.

상속은, 퍼블릭 인터페이스는 유지하면서 새로운 구현을 가진 객체를 추가할 수 있는 간단한 방법이다. 하지만, 상속은 자식 클래스를 부모 클래스의 구현에 강하게 결합시킨다.

##### 인터페이스를 이용한 타입 계층 구현

인터페이스는, 상속으로 인한 결합도 문제를 피하고 다중 상속이라는 구현 제약을 해결할 수 있는 방법이다. 

##### 추상 클래스를 이용한 타입 계층 구현

추상 클래스는, 클래스 상속을 이용해 구현을 공유하면서도 결합도로 인한 부작용을 피하는 방법이다.

##### 추상 클래스와 인터페이스 결합하기

인터페이스를 이용해 타입을 정의하고 특정 상속 계층에 국한된 코드를 공유할 필요가 있으면 추상 클래스를 이용해 코드 중복을 방지 할 수 있다. 이것이 골격 구현 추상 클래스이다.

```java
public interface DiscountPolicy {
  Money calculateDiscountAmount(Screening screeing);
}

public abstract class DefaultDiscountPolicy implements DiscountPolicy {
  ...
   
  @Override
  public Money calculateDiscountAmount(Screening screeing){
    ...
  }
  
  abstract protected Money getDiscountAmount(Screeing screening);
} 
```

##### 덕 타이핑 사용하기

덕 테스트는 어떤 대상의 행동이 오리와 같다면 그것을 오리라는 타입으로 취급해도 무방하다는 것이다. 즉, 객체가 어떤 인터페이스에 정의된 행동을 수행할 수 있다면 그 객체를 해당 타입으로 분류해도 문제가 없다.

자바 같은 대부분의 정적 타입 언어에서는 두 클래스를 동일한 타입으로 취급하기 위해서는, 코드 상의 타입이 동일하게 선언되어 있어야한다. 

반면, 런타입에 타입을 결정하는 동적 타입 언어에서는 특정한 클래스를 상속받거나 인터페이스를 구현하지 않고도 객체가 수신할 수 있는 메세지의 지합으로 객체의 타입을 결정할 수 있다.

##### 믹스인과 타입 계층

믹스인은, 객체를 생성할 때 코드 일부를 섞어 넣을 수 있도록 만들어진 일종의 추상 서브클래스이다. 사용하는 목적은, 다양한 객체의 구현 안에서 동일한 행동을 중복 코드 없이 재사용할 수 있게 만드는 것이다. 즉, 공통의 행동이 믹스인된 객체들은 동일한 메세지를 수신할 수 있는 퍼블릭 인터페이스를 공유하는 것이다. 예를 들어, 스칼라의 trait 로 구현할 수 있다.

스칼라의 trait 와 유사하게, 자바 8에 추가된 default method 는 인터페스에 메서드의 기본 구현을 추가하는 것을 허용한다. 디폴트 메서드가 제공하는 혜택을 누리기 위해서는 한계를 명확히 인식해야한다.

```java
public interface DiscountPolicy {
  default Money calculateDiscountAmount(Screening screeing){
    for(DiscountCondition each : getConditions()){
      if(each.isSatisfiedBy(screeing)){
        return getDiscountAmount(screening);
      }
    }
    
    return screening.getMovieFee();
  }
  
  List<DiscountCondition> getConditions();
  Money getDiscountAmount(Screeing screening);
}
```

calculateDiscountAmount 가 내부적으로 두개의 메서드를 사용한다. 그래서 이 인터페이스를 구현하는 모든 클래스들은 해당 메서드의 구현을 제공해야한다는 것을 명시한 것이다.

추상 클래스를 사용했을 경우에는, getDiscountAmount 메서드의 가시성이 protected 였다. 하지만 이제 디폴트 메서드안에서 사용된다는 이유만으로 public 메서드가 되어야한다. 이것은 외부에 노출할 필요가 없는 메서드를 불필요하게 퍼블릭 인터페이스에 추가한 결과를 낳는다.

디폴트 메서드가 추가된 이유는, 인터페이스에 새로운 오퍼레이션을 추가할 경우에 발생하는 하위 호환성 문제를 해결하기 위해서이다. 추상 클래스를 제거하기 위해서가 아니다.

---

오브젝트 <조영호>
