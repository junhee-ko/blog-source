---
layout: post
title:  "[오브젝트] 14장_일관성 있는 협력"
date:   2020-01-20
categories: Object
---

## 01 핸드폰 과금 시스템 변경하기

![](/image/object_cowork_chapter14.png)

##### 기본 정책 확장

1. 고정 요금 방식

   ex ) 10초당 18원

2. 시간대별 방식

   ex ) 00시-19시 : 10초당 19월, 19시-24시 : 10초당 15원

3. 요일별 방식

   ex ) 평일 : 10초당 38원, 공휴일 : 10초당 19원

4. 구간별 방식

   ex) 초기 1분 : 10초당 50원, 초기 1분 이후 : 10초당 20원

##### 고정 요금 방식 구현하기

```java
public class FixedFeePolicy extends BasicRatePolicy {
    private Money amount;
    private Duration duration;

    public FixedFeePolicy(Money amount, Duration duration) {
        this.amount = amount;
        this.duration = duration;
    }

    @Override
    protected Money calculateCallFee(Call call) {
        return amount.times(call.getDuration().getSeconds() / seconds.getSeconds());
    }
}
```

##### 시간대별 방식 구현하기

TimeOfDayDiscountPolicy 클래스에서 가장 중요한 것은, 시간에 따라 서로 다른 요금 규칙을 정의하는 방법을 결정하는 것이다. 

이를 위해 서로 다른 List 를 가질 수 있다. 같은 규칙에 포함된 요소들은 List 의 동일한 인덱스에 위치한다.

```java
public class TimeOfDayDiscountPolicy extends BasicRatePolicy {
    private List<LocalTime> starts = new ArrayList<>();
    private List<LocalTime> ends = new ArrayList<>();
    private List<LocalTime> duration = new ArrayList<>();
    private List<Money> amounts = new ArrayList<>();

    @Override
    protected Money calculateCallFee(Call call) {
        ...
    }
    
    ...
}
```

##### 요일별 방식 구현하기

시간대별 방식의 4개 List 와 다르게, 규칙을 DayOfWeekDiscountRule 이라는 하나의 클래스로 구현해보자.

```java
public class DayOfWeekDiscountRule {
    private List<DayOfWeek> dayOfWeeks = new ArrayList<>();
    private Duration duration = Duration.ZERO;
    private Money amount = Money.ZERO;

    public DayOfDiscountRule(List<DayOfWeek> dayOfWeeks, Duration duration, Money amount) {
        this.dayOfWeeks = dayOfWeeks;
        this.duration = duration;
        this.amount = amount;
    }
    
    public Money caculate(DateTimeInterval interval){
        if(dayOfWeeks.contains(interval.getFrom().getDayOfWeek())){
            return ...
        }
        ...
    }
}
```

```java
public class DayOfWeekDiscountPolicy extends BasicRatePolicy {
    private List<DayOfWeekDiscountRule> rules = new ArrayList<();

    public DayOfWeekDiscountPolicy(List<DayOfWeekDiscountPolicy> rules) {
        this.rules = rules;
    }

    @Override
    protected Money calculateCallFee(Call call) {
        ...
    }   
}
```

##### 구간별 방식 구현하기

(구간별 방식 구현 코드는 생략)

지금까지의 구현의 가장 큰 문제는, 이 클래스들이 `유사한 문제를 해결하고 있음에도 불구하고 설계에 일관성이 없다`는 것이다.

## 02 설계에 일관성 부여하기

`협력을 일관성 있게` 만들기 위해서는,

1. 변하는 개념을 변하지 않는 개념으로 분리하라.
2. 변하는 개념을 캡슐하하라.

##### 조건 로직 객체 탐색

4장의 절차적인 방식으로 구현했던 ReservationAgency 코드를 보자.

```java
public class ReservationAgency {
    public Reservation reservation(Screening screening, Customer customer, int audienceCount){
        for(DiscountCondition condition : movie.getDiscountConditions(){
            if(condition.getType() == DiscountCondition.PERIOD){
                //기간조건
            }else {
                //회차 조건
            }
        }
            
        if(discountable){
            switch (movie.getMovieType()){
                case AMOUNT_DISCOUNT:
                    //금액할인 정책
                case PERCENT_DISCOUNT:
                    // 비율 할인 정책
                ...
            }
        } else{
            ...
        }
    }
}
```

객체지향에서는, 변경을 다루는 전통적인 방법은 조건 로직을 객체 사이의 이동으로 바꾸는 것이다.

![](/image/object_movie_chapter14.png)

```java
public class Movie {
    private DiscountPolicy discountPolicy;

    public Money calculateMovieFee(Screening screening){
        return fee.minus(discountPolicy.calculateDiscountAmount(screening));
    }
}
```

```java
public abstract class DiscountPolicy {
    private List<DiscountCondition> conditions = new ArrayList<>();

    public Money calculateDiscountAmount(Screening screening){
        for(DiscountCondition each : conditions){
            if(each.isSatisfiedBy(screening)){
                return getDiscountAmount(screening);
            }
        }

        return screening.getMovieFee();
    }
}
```

객체지향적인 코드는 조건을 판단하지 않는다. 단지 다음 객체로 이동할 뿐이다.

1. Movie 는 현재의 할인 정책이 어떤 종류인지 판단하지 않는다. 단지 DiscountPolicy 로 향하는 참조를 통해 메세지를 전달할 뿐이다. 

2. DiscountPolicy 역시 할인 조건의 종류를 판단하지 않는다. 단지 DiscountCondition 으로 향하는 참조를 통해 메세지를 전단할 뿐이다.

`협력을 일관성 있게` 만들기 위한 방법을 다시 정리하면,

1. 변하는 개념을 변하지 않는 개념으로 분리하라.

   각 조건문을 개별적인 객체로 분리했고 이 객체들을 일관성 있게 협력하기 위해 타입 계층을 구성했다.

2. 변하는 개념을 캡슐하하라.

   Movie 가 알고 있는 사실은 협력하는 객체가 단지 DiscountPolicy 클래스의 인터페이스에 정의된 calculateDiscountAmount 메세지를 이해할 수 있다는 것 뿐이다. 

   메세지 수신자 타입은 Movie 에 대해 완벽히 캡슐화되었다.

##### 캡슐화 다시 살펴보기

캡슐화란 변하는 어떤 것이든 감추는 것이다.

다음 그림에는 다양한 종류의 캡슐화가 공존한다.

![](/image/object_encapsulation_chapter14.png)

1. 데이터 캡슐화

   클래스는 내부에 관리하는 데이터를 캡슐화한다.

2. 메서드 캡슐화

   DiscountPolicy 클래스에 정의된 getDiscountAmount 메서드의 가시성은 protected 이다.

   즉, 클래스의 외부에서는 이 메서드에 접근하지 못하고 클래스 내부와 서브 클래스에서만 접근이 가능하다.

3. 객체 캡슐화

   Movie 클래스는 DiscountPolicy 타입의 인스턴스 변수를 포함한다. 

   이 인스턴스 변수는 private 가시성을 가지므로 Movie 와 DiscountPolicy 사이의 관계를 변경해도 외부에는 영향을 미치지 않는다.

   즉, 합성이다.

4. 서브타입 캡슐화

   Movie 는 DiscountPolicy 에 대해서 알고 있지만, AmountDiscountPolicy 에 대해서는 모른다. 

   그러나 실행 시점에 협력할 수 있다.

   서트타입의 종류를 캡슐화하고 있기 때문에, 다형성의 기반이 된다.

## 3 일관성 있는 기본 정책 구현하기

전체 설계는 다음과 같다.

![](/image/object_all_chapter14.png)

##### 변경 분리하기

시간대별, 요일별, 구간별 방식의 공통점은 각 기본 정책을 구성하는 방식이 유사하는 것이다.

1. 기본 정책은 한 개 이상의 규칙으로 구성된다.
2. 하나의 규칙은 적용 조건과 단위요금을 조합이다.

![](/image/object_rule_chapter14.png)

모든 규칙에 적용 조건이 포함된다는 사실은 변하지 않지만 실제 조건의 세부 내용은 다르다. 

즉, 조건의 세부 내용이 바로 변화에 해당하는 것이다. 

변하지 않는 '규칙' 으로부터 변하는 '적용 조건' 을 분리해야한다.

##### 변경 캡슐화하기

![](/image/object_fee_chapter14.png)

변하는 FeeCondition 의 서브 타입은 변하지 않는 FeeRule 로부터 캡슐화된다. 

##### 협력 패턴 설계하기

![](/image/object_basic_chapter14.png)

1. BasicRatePolicy 의 calculateFee 메서드는 인자로 전달받은 통화 목록의 전체 요금을 계산한다.

2. BasicRatePolicy 는 목록에 포함된 각 Call 별로 FeeRule 의 calculateFee 메서드를 실행한다.
3. 하나의 BasicRatePolicy 는 하나 이상의 FeeRule 로 구성되어서, Call 하나당 FeeRule 에 다수의 calculateFee 메세지가 전송된다.

##### 추상화 수준에서 협력 패턴 구현하기

변하지 않는 요소와 추상적인 요소만으로 요금 계산에 필요한 전체적인 협력 구조를 설명할 수 있다.

##### 구체적인 협력 구현하기

code : 505 Page

유사한 기능에 대해 유사한 협력 패턴을 적용하는 것은 객체지향 시스템에서 개념적 무결성을 유지할 수 있는 방법이다. 

개념적 무결성이란, 일관성이다.

---

오브젝트 <조영호>