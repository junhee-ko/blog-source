---
layout: post
title:  "[오브젝트] 11장_합성과 유연한 설계"
date:   2019-12-27
categories: OOP
---

코드 재사용 기법으로는,

1. 상속
   부모클래스와 자식클래스를 연결해서, 부모클래스의 코드 재사용

2. 합성
   전체를 표현하는 객체가 부분을 표현하는 객체를 포함해서 객체의 코드 재사용
   내부에 포함되는 객체의 구현이 아니라 `퍼블릭 인터페이스`에 의존

## 01 상속을 합성으로 변경하기

상속의 문제는,

1. 불필요한 인터페이스 상속
2. 메서드 오버라이딩 오작용
3. 부모 클래스와 자식 클래스 동시 수정

이 문제들을 합성으로 해결해보자.

##### 불필요한 인터페이스 상속 : java.utils.Properties, java.utils.Stack

1. 불필요한 Hasbtable 의 오퍼레이션들이 Properties 클래스의 퍼블릭 인터페이스를 오염시키지 않는다.

```java
public class Properties {
    private Hashtable<String, String> properties = new Hashtable<>();

    public String setProperty(String key, String value) {
        return properties.put(key, value);
    }

    public String getProperty(String key) {
        return properties.get(key);
    }
}
```

2. 불필요한 Vector 의 오퍼레이션들이 Stack 클래스의 퍼블릭 인터페이스를 오염시키지 않는다.

```java
public class Stack<E> {
    private Vector<E> elements = new Vector<>();

    public E push(E item) {
        elements.addElement(item);

        return item;
    }

    public E Pop() {
        if (elements.isEmpty()) {
            throw new EmptyStackException();
        }

        return elements.remove(elements.size() - 1);
    }
}
```

##### 메서드 오버라이딩 오작용 : InstrumentedHashSet

```java
public class InstrumentedHashSet<E> implements Set {
    private Set<E> set;
   	...   
    
    @Override ..
    @Override ..      
		@Override ..
}  
```

##### 부모 클래스와 자식 클래스의 동시 수정 : PersonalPlaylist

```java
public class PersonalPlaylis {
    private Playlist playlist = new Playlist();
    
    public void append(Song song){
        playlis.append(song);
    }
    
    public void remove(Song song){
        playlist.getTracks().remove(song);
        playlist.getSingers().remove(song.getSinger());
    }
}
 
```

## 02 상속으로 인한 조합의 폭발적인 증가

상속으로 작은 기능들을 조합해 더 큰 기능을 수행하는 객체를 만들 때의 문제는,

1. 하나의 기능을 추가하거나 수정을 위해 `불필요하게 많은 수의 클래스를 추가하거나 수정`해야한다.
2. 단일 상속만 지원하는 언어에서는 상속으로 `중복 코드`가 중가한다.

##### 기본 정책과 부가 정책 조합하기

1. 기본 정책 : 일반 요금제 / 심야 할인 요금제
   
2. 부가 정책 : 세금 정책 / 기본 요금 할인 정책
   기본 정책의 계산 결과에 적용, 선택적으로 적용 가능, 조합 가능, 임의의 순서로 적용 가능

##### 상속을 이용해서 기본 정책 구현하기

##### 기본 정책에 세금 정책 조합하기

##### 기본 정책에 기본 요금 할인 정책 조합하기

##### 중복 코드의 덫에 걸리다

![](/image/object_inheritance_problem.png)

상속의 남용으로 하나의 기능을 추가하기 위해 필요 이상으로 많은 수의 클래스를 추가해야하는 경우를 `클래스 폭발 문제`,  `조합의 폭발 문제` 라고 한다.
이 문제는, 자식 클래스가 부모 클래스의 구현에 강하게 결합되도록 강요하는 상속의 한계 때문에 발생하는 문제이다.

## 03 합성 관계로 변경하기

1. 상속
   조합의 결과를 개별 클래스 안으로 밀어 넣는 방법

2. 합성
   합성은 조합을 구성하는 요소들을 개별 클래스로 구현한 후 실행 시점에 인스턴스를 조립하는 방법

##### 기본 정책 합성하기

```java
public interface RatePolicy {
    Money calculateFee(Phone phone);
}

public abstract class BasicRatePolicy implements RatePolicy {
    @Override
    public Money calculateFee(Phone phone) {
        Money result = Money.ZERO;

        for(Call call phone.getCalls(){
            result.plus(calculateCallFee(call));
        }

        return result;
    }

    protected abstract Money calculateCallFee(Call call);
}

public class RegularPolicy extends BasicRatePolicy {
    private Money amnount;
    private Duration seconds;

    public RegularPolicy(Money amnount, Duration seconds) {
        this.amnount = amnount;
        this.seconds = seconds;
    }

    @Override
    protected Money calculateCallFee(Call call) {
        return amnount.times(call.getDuration().getSeconds() / seconds.getSeconds());
    }
}            

public class NightlyDiscountPolicy extends BasicRatePolicy {
    private static final int LATE_NIGHT_HOUR = 22;

    private Money nightlyAmnount;
    private Money regularAmnount;
    private Duration seconds;

    public NightlyDiscountPolicy(Money nightlyAmnount, Money regularAmnount, Duration seconds) {
        this.nightlyAmnount = nightlyAmnount;
        this.regularAmnount = regularAmnount;
        this.seconds = seconds;
    }

    @Override
    protected Money calculateCallFee(Call call) {
        if (call.getFrom.getHour() >= LATE_NIGHT_HOUR) {
            return nightlyAmnount.times(call.getDuration().getSeconds() / seconds.getSeconds());
        }

        return regularAmnount.times(call.getDuration().getSeconds() / seconds.getSeconds());
    }
}
            
public class Phone {
    private RatePolicy ratePolicy; // Phone 이 다양한 요금 정책과 협력할수 있어야 하므로 인터페이스
    private List<Call> calls = new ArrayList<>();

    public Phone(RatePolicy ratePolicy) {
        this.ratePolicy = ratePolicy;
    }

    public List<Call> getCalls() {
        return Collections.unmodifiableList(calls);
    }

    public Money calculateFee() {
        return ratePolicy.calculateFee(this);
    }
}
```

```java
// 일반 요금제
Phone phone = new Phone(new RegularPolicy(Money.wons(10), Duration.ofSeconds(10)));

// 심야 할인 요금제
Phone phone = new Phone(new NightlyDiscountPolicy(Money.wons(5), Money.wons(10), Duration.ofSeconds(10)));
```

##### 부가 정책 적용하기

```java
public abstract class AdditionalRatePolicy implements RatePolicy {
    private RatePolicy next;

    public AdditionalRatePolicy(RatePolicy next) {
        this.next = next;
    }


    @Override
    public Money calculateFee(Phone phone) {
        Money fee = next.calculateFee(phone);

        return afterCalculated(fee);
    }

    abstract protected Money afterCalculated(Money fee);
}

public class TaxablePolicy extends AdditionalRatePolicy {
    private double taxRatio;

    public TaxablePolicy(RatePolicy next, double taxRatio) {
        super(next);
        this.taxRatio = taxRatio;
    }

    @Override
    protected Money afterCalculated(Money fee) {
        return fee.plus(fee.times(taxRatio));
    }
}

public class RateDiscountPolicy extends AdditionalRatePolicy{
    private Money disoucntAmount;

    public RateDiscountPolicy(RatePolicy next, Money disoucntAmount) {
        super(next);
        this.disoucntAmount = disoucntAmount;
    }

    @Override
    protected Money afterCalculated(Money fee) {
        return fee.minus(disoucntAmount);
    }
}
```

##### 기본 정책과 부가 정책 합성하기

```java
// 일반 요금제에 세금 정책 조합
Phone phone = new Phone(
    new TaxablePolicy(0.05, 
        new RegularPolicy(...)));

// 일반 요금제에 기본 요금 할인 정책을 조합한 결과에 세금 정책 조합
Phone phone = new Phone(
    new TaxablePolicy(0.05, 
        new RateDiscountablePolicy(Money.wons(1000),
            new RegularPolicy(...))));
```

##### 새로운 정책 추가하기

![](/image/object_composition.png)

요구사항이 변경되면 오직 하나의 클래스만 수정하면 된다.
세금 정책을 변경한다고 할 때,

1. 상속
   여러 클래스를 수정

2. 합성
   오직 TaxablePolicy 클래스 하나만 변경

## 04 믹스인

객체를 생성할 때 코드 일부를 클래스 안에 섞어 넣어 재사용하는 기법이다.
합성처럼 유연하며 상속처럼 쉽게 코드 재사용할 수 있다.
스칼라의 trait 를 이용하면 믹스인 구현이 가능하다.
