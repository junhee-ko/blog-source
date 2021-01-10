---
layout: post
title:  "[오브젝트] 2장_객체지향 프로그래밍"
date:   2019-11-06
categories: OOP
---

## 영화 예매 시스템

사용자가 영화 시스템을 이용해서 영화를 예매할 수 있는 애플리케이션을 구현한다.

## 협력, 객체, 클래스

객치지향 패러다임은, 클래스가 아닌 객체에 초점을 맞춰야한다. 다음 두가지에 집중해야 한다.

1. 어떤 클래스가 필요한지를 고민하기 전에, 어떤 객체들이 필요한지 고민하라. 클래스는 공통적인 상태와 행동을 공유하는 객체들을 추상화한 것이다.
2. 객체는 독립적인 존재가 아니라, 기능 구현을 위한 협력 공동체의 일원이다.

## 도메인의 구조를 따르는 프로그램 구조

도메인이란, 문제를 해결하기 위해 사용자가 프로그램을 사용하는 분야이다.
객체지향 패러다임에서는, 도메인을 구성하는 개념들이 프로그램의 객체와 클래스로 매끄럽게 연결된다. 
일반적으로 클래스의 이름은, 대응되는 도메인의 개념의 이름과 동일하거나 유사하게 지어야한다. 

## 클래스 구현하기

클래스를 구현하거나 다른 개발자에 의해 개발된 클래스를 사용할 때 가장 중요한 것은, 클래스의 경계를 구분짓는 것이다.
외부에서는 객체의 속성에 직접 접근할 수 없도록 막고 적절한 public 메서드를 통해서만 내부 상태를 변경할 수 있게해야한다. 
경계의 명확성이 객체의 자율성을 보장하기 때문이다. 또, 프로그래머에게 구현의 자유를 제공하기 때문이다.

### 자율적인 객체

객체는 스스로 판단하고 행동하는 자율적인 존재다. 
객체가 자율적인 존재가 되기 위해서는, 외부의 간섭을 최소화해야한다.
캡슐화와 접근 제어는 객체를 두 부분으로 나눈다. (인터베이스와 구현의 분리)

1. Public Interface : 외부에서 접근 가능한 부분
2. Implementation : 외부에서 접근 불가능한 부분

### 프로그래머의 자유

프로그래머의 역할은, 클래스 작성자와 클라이언트 프로그래머로 구분할 수 있다.

1. 클래스 작성자 : 새로운 데이터 타입을 프로그램에 추가
2. 클라이언트 프로그래머 : 클래스 작성자가 추가한 데이터 타입을 사용

클래스 작성자는 클라이언트 프로그래머에게 필요한 부분만 공개하고 나머지는 숨겨야한다. 
클라이언트 프로그래머가 숨겨 놓여진 부분에 접근하는것을 막아 내부 구현을 마음대로 변경할 수 있다.
객체의 외부와 내부를 구분하면 클라이언트 프로그래머가 알아야할 지식 양이 줄고 클래스 작성자가 자유롭게 구현을 변경할 수 있다. 
변경될 가능성이 있는 세부적인 내용은 private 영역으로 감춰 변경으로 인한 혼란을 막을 수 있다.

Screening 클래스는 사용자들이 예매하는 대상인 '상영' 도메인이다.

```java
import java.time.LocalDateTime;

public class Screening {

  private Movie movie;
  private int sequence;
  private LocalDateTime whenScreened;

  public Screening(Movie movie, int sequence, LocalDateTime whenScreened) {
    this.movie = movie;
    this.sequence = sequence;
    this.whenScreened = whenScreened;
  }

  public LocalDateTime getStartTime() {
    return whenScreened;
  }

  public boolean isSequence(int sequence) {
    return this.sequence == sequence;
  }

  public Money getMovieFee() {
    return movie.getFee();
  }

  public Reservation reserve(Customer customer, int audienceCount) {
    return new Reservation(customer, this, calculateFee(audienceCount), audienceCount);
  }

  private Money calculateFee(int audienceCount) {
    return movie.calculateMovieFee(this).times(audienceCount);
  }
}
```

Money 는 금액과 관련된 여러 계산을 구현하는 클래스이다.
금액을 Money 클래스로 정의하면, 저장하는 값이 금액과 관련돼 있다는 의미를 전달 할 수 있다. (Long Type 은 그렇지 못함) 
객체지향의 장점은 객체를 이용해 도메인의 의미를 풍부하게 표현할 수 있다.

```java
import java.math.BigDecimal;

public class Money {

  public static final Money ZERO = Money.wons(0);
  private final BigDecimal amount;

  public Money(BigDecimal amount) {
    this.amount = amount;
  }

  public static Money wons(long amount) {
    return new Money(BigDecimal.valueOf(amount));
  }

  public static Money wons(double amount) {
    return new Money(BigDecimal.valueOf(amount));
  }

  public Money plus(Money amount) {
    return new Money(this.amount.add(amount.amount));
  }

  public Money minus(Money amount) {
    return new Money(this.amount.subtract(amount.amount));
  }

  public Money times(double percent) {
    return new Money(this.amount.multiply(BigDecimal.valueOf(percent)));
  }

  public boolean isLessThan(Money other) {
    return amount.compareTo(other.amount) < 0;
  }

  public boolean isGreaterThanOrEqual(Money other) {
    return amount.compareTo(other.amount) >= 0;
  }
}
```

Reservation 클래스는 영화 예매 도메인이다.

```java
public class Reservation {

  private Customer customer;
  private Screening screening;
  private Money money;
  private int audienceCount;

  public Reservation(Customer customer, Screening screening, Money money, int audienceCount) {
    this.customer = customer;
    this.screening = screening;
    this.money = money;
    this.audienceCount = audienceCount;
  }
}
```

## 협력

객체는 다른 객체의 인터페이스에 공개된 행동을 수행하도록 '요청' 할 수 있다. 요청 받은 객체는 요청 처리후에 '응답' 한다.
객체가 다른 객체와 상호작용하는 방법은, '메세지' 를 전송하는 것뿐이다. 메세지를 받는 객체는 메세지를 '수신했다' 고 한다. 
수신된 메세지를 처리하기 위한 자신만의 방법을 '메서드' 라고 한다. 메시지를 수신한 Movie 는 스스로 자신의 적절한 메서드를 선택한다. 

## 할인 요금 계산을 위한 협력 시작하기

Movie 클래스를 구현하자.
calculateMovieFee 메서드에는 어떤 할인 정책을 사용할 것인지 결정하는 코드가 안보인다. 
여기에는 상속과 다형성 개념이 숨겨져있다.

```java
import java.time.Duration;

public class Movie {

  private String title;
  private Duration runningTime;
  private Money fee;
  private DiscountPolicy discountPolicy;

  public Movie(String title, Duration runningTime, Money fee, DiscountPolicy discountPolicy) {
    this.title = title;
    this.runningTime = runningTime;
    this.fee = fee;
    this.discountPolicy = discountPolicy;
  }

  public Money getFee() {
    return fee;
  }

  public Money calculateMovieFee(Screening screening) {
    return fee.minus(discountPolicy.calculateDiscountAmount(screening));
  }
}
```

## 할인 정책과 할인 조건

할인 정책은, 금액 할인 정책과 비율 할인 정책으로 나뉜다. 
두 클래스는 대부분의 코드가 유사하고 할인 요금 계산하는 방식만 조금 다르기 때문에, 중복 코드 제거를 위해 추상 클래스를 사용한다.
부모 클래스에 기본적인 알고리즘의 흐름을 구현하고 중간에 필요한 처리를 자식 클래스에 위임하는 `Template Method Pattern` 을 이용했다.

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public abstract class DiscountPolicy {

  private List<DiscountCondition> conditions = new ArrayList<>();

  public DiscountPolicy(DiscountCondition... conditions) {
    this.conditions = Arrays.asList(conditions);
  }


  public Money calculateDiscountAmount(Screening screening) {
    for (DiscountCondition condition : conditions) {
      if (condition.isSatisfiedBy(screening)) {
        return getDiscountAmount(screening);
      }
    }

    return Money.ZERO;
  }

  protected abstract Money getDiscountAmount(Screening screening);
}
```

```java
public class AmountDiscountPolicy extends DiscountPolicy {

  private Money discountAmount;

  public AmountDiscountPolicy(Money discountAmount, DiscountCondition... conditions) {
    super(conditions);
    this.discountAmount = discountAmount;
  }

  @Override
  protected Money getDiscountAmount(Screening screening) {
    return discountAmount;
  }
}
```

```java
public class PercentDiscountPolicy extends DiscountPolicy {

  private double percent;

  public PercentDiscountPolicy(double percent, DiscountCondition... conditions) {
    super(conditions);
    this.percent = percent;
  }

  @Override
  protected Money getDiscountAmount(Screening screening) {
    return screening.getMovieFee().times(percent);
  }
}
```

할인 조건은, 순번 조건과 기간 조건으로 나뉜다.

```java
public interface DiscountCondition {

  boolean isSatisfiedBy(Screening screening);
}
```

```java
public class SequenceCondition implements DiscountCondition {

  private int sequence;

  public SequenceCondition(int sequence) {
    this.sequence = sequence;
  }

  @Override
  public boolean isSatisfiedBy(Screening screening) {
    return screening.isSequence(sequence);
  }
}
```

```java
import java.time.DayOfWeek;
import java.time.LocalTime;

public class PeriodCondition implements DiscountCondition {

  private DayOfWeek dayOfWeek;
  private LocalTime startTime;
  private LocalTime endTime;

  public PeriodCondition(DayOfWeek dayOfWeek, LocalTime startTime, LocalTime endTime) {
    this.dayOfWeek = dayOfWeek;
    this.startTime = startTime;
    this.endTime = endTime;
  }

  @Override
  public boolean isSatisfiedBy(Screening screening) {
    return screening.getStartTime().getDayOfWeek().equals(dayOfWeek) &&
        startTime.compareTo(screening.getStartTime().toLocalTime()) <= 0 &&
        endTime.compareTo(screening.getStartTime().toLocalTime()) >= 0;
  }
}
```

## 할인 정책 구성하기

생성자의 파라미터 목록을 이용해 초기화에 필요한 정보를 전달하도록 강제하면, 올바른 상태를 가진 객체의 생성을 보장할 수 있다.
위에서 정의한 Movie 의 생성자는 오직 하나의 DiscountPolicy 인스턴스만 받을 수 있도록 선언되어 있다.
반면, DiscountPolicy 의 생성자는 여러 개의 DiscountCondition 인스턴스를 허용한다.

## 컴파일 시간 의존성, 실행 시간 의존성

코드의 의존성과 실행시점의 의존성이 서로 다를 수 있다. 
Movie 인스턴스는 실행 시에 AmountDiscountPolicy 나 PercentDiscountPolicy 의 인스턴스에 의존해야한다.
하지만, 코드 수준에서 Movie 클래스는 두 클래스중 어떤 것에도 의존하지 않는다. 오직 추상 클래스인 DiscountPolicy 에만 의존한다.

## 차이에 의한 프로그래밍

부모 클래스와 다른 부분만을 추가해서 새로운 클래스를 쉽고 빠르게 만드는 방법을, 차이에 의한 프로그래밍이라고 한다.
AmountDiscountPolicy 와 PercentDiscountPolicy 는 DiscountPolicy 에서 정의한 추상 메서드를 오버라이딩해서 DiscountPolicy 의 행동을 수정한다.  

## 상속과 인터페이스

자식 클래스는 부모 클래스가 수신할 수 있는 모든 메세지를 수신할 수 있기 때문에, 외부 객체는 자식 클래스를 부모 클래스와 동일한 타입으로 간주할 수 있다.
자식 클래스가 부모 클래스를 대신하는 것을 `업캐스팅`이라고 한다.
Movie 는 협력 객체가 calculateMovieFee 라는 메세지를 이해할 수 있으면, 그 객체가 어떤 인스턴스인지 상관하지 않는다.
즉, 자식 클래스인 AmountDiscountPolicy 와 PercentDiscountPolicy 는 부모 클래스의 인터페이스를 물려받아서 부모 클래스 대신 사용될 수 있다.

```java
import java.time.Duration;

public class Movie {

  public Money calculateMovieFee(Screening screening) {
    return fee.minus(discountPolicy.calculateDiscountAmount(screening));
  }
}
```

## 다형성

Movie 는 동일한 메세지를 전송하지만, 실제로 어떤 메서드가 실행될 것인지는 메시지를 수신하는 객체의 클래스가 무엇이냐에 따라 다르다. 이것이 다형성이다.
다형성은 컴파일 시간 의존성과 실행 시간 의존성을 다르게 만들 수 있는 객체지향 특성을 이용해 서로 다른 메서드를 실행할 수 있게 한다.
다형성을 구현하는 방법은 다양하지만, 공통점은 메세지에 응답하기 위해 실행될 메서드를 컴파일 시점이 아닌 실행시점에 결정한다는 것이다. 
이를 지연바인딩, 동적바인딩 이라고 한다.

## 추상화

같은 계층에 속하는 클래스들이 공통으로 가질 수 있는 인턴페이스를 구현하며 구현의 일부 (추상클래스) 또는 전체 (인터페이스) 를 자식 클래스가 결정할 수 있도록 결정권을 위임한다.
추상화의 장점은,

1. 세부사항에 억눌리지 않고 상위 개념만으로 도메인의 중요한 개념을 설명할 수 있다
   ( Movie - DiscountPolicy - DiscountCondition )
2. 기존 구조를 수정하지 않고, 새로운 기능을 쉽게 추가하고 확장할 수 있다.

## 유연한 설계

책임의 위치를 결정하기 위해, 조건문을 사용하는 것은 협력의 설계 측면에서 좋지 않은 선택이다. 
0 원 이라는 할인 요금을 계산할 책임을 그대로 DiscountPolicy 계층에 유지하기 위해, NoneDiscountPolicy 클래스를 추가할 수 있다.
그럼, 기존의 Movie 와 DiscountPolicy 는 수정하지 않고, 새로운 클래스를 추가하는 것만으로 애플리케이션 환장이 가능하다.

```java
public class NoneDiscountPolicy extends DiscountPolicy {

  @Override
  protected Money getDiscountAmount(Screening screening) {
    return Money.ZERO;
  }
}
```

## 상속

상속의 단점은, 

1. 캡슐화 위반이다. 부모 클래스가 변경되면, 자식 클래스도 함께 변경될 확률이 높다.
2. 설계가 유연하지 않다. 부모클래스와 자식클래스의 관계를 컴파일 시점에 결정해서 실행시점에 객체의 종류를 변경할 수 없다.

인스턴스 변수로 연결하면, 실행 시점에 할인 정책을 간단히 변경할 수 있다.

## 합성

인터페이스에 정의된 메세지를 통해서만 코드를 재사용하는 방법이 합성이다.
합성은 상속의 두가지 문제를 해결한다.

1. 인터페이스에 정의된 메세지를 통해서만 재사용하기 때문에 구현을 효과적으로 캡슐화한다.
2. 의존하는 인스턴스를 교체하는 것이 쉽기 때문에 유연하다. 상속은 클래스를 통해 강하게 결합되지만, 합성은 메세지를 통해 느슨하게 결합된다.

---

오브젝트 <조영호>
