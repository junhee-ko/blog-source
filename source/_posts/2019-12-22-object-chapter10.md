---
layout: post
title:  "[오브젝트] 10장_상속과 코드 재사용"
date:   2019-12-22
categories: OOP
---

이번 장은, 클래스 재사용을 위해 새로운 클래스를 추가하는 가장 대표적 기법인 상속에 대해 정리한다.

## 01 상속과 중복 코드

##### DRY 원칙 (Don't Repeat Yourself)

"동일한 지식을 중복하지 마라"

중복 여부를 판단하는 기준은 변경이다. 중복 코드는 변경을 방해한다. 이것이 중복 코드를 제거해야하는 핵심적 이유이다. 

중복 코드는 코드를 수정하는데 노력을 몇배로 증가시킨다. 왜냐하면,

1. 어떤 코드가 중복인지 찾아야하고,
2. 찾아낸 모든 코드를 일관되게 수정해야하고,
3. 개별적으로 테스트를 다 해야하기 때문이다. 

##### 중복과 변경

한 달에 한 번씩 가입자별로 전화 요금 계산하는 간단한 애플리케이션을 개발해보자.

개별 통화 기간을 저장하는 클래스가 필요하다.

```java
public class Call {
    private LocalDateTime from; //통화 시작 시간
    private LocalDateTime to; // 통화 종료 시간

    public Call(LocalDateTime from, LocalDateTime to) {
        this.from = from;
        this.to = to;
    }
    
    public Duration getDuration(){
        return Duration.between(from, to);
    }
    
    public LocalDateTime getFrom(){
        return from;
    }
}
```

전체 통화 목록에 대해 알고 있는 정보 전문가에게 `요금 계산 책임` 을 할당해야한다. 

```java
public class Phone {
    private Money amount;
    private Duration seconds;
    private List<Call> calls = new ArrayList<>();

    public Phone(Money amount, Duration seconds) {
        this.amount = amount;
        this.seconds = seconds;
    }
    
    public void call(Call call){
        calls.add(call);
    }

    public List<Call> getCalls() {
        return calls;
    }

    public Money getAmount() {
        return amount;
    }

    public Duration getSeconds() {
        return seconds;
    }
    
    public Money calculateFee(){
        Money result = Money.ZERO;
        
        for (Call call : calls){
            result = result.plus(amount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
        }
        
        return result;
    }
}
```

그런데, `심야 할인 요금제` 라는 새로운 요금 방식을 추가해야하는 요구사항이 접수됐다. 

Phone 코드를 복사해서 새로운 클래스를 만들어보자.

```java
public class NightlyDiscountPhone {
    private static final int LATE_NITGHT_HOUR = 22;
    
    private Money nightAmount;
    private Money regularAmount;
    private Duration seconds;
    private List<Call> calls = new ArrayList<>();

    public NightlyDiscountPhone(Money nightAmount, Money regularAmount, Duration seconds) {
        this.nightAmount = nightAmount;
        this.regularAmount = regularAmount;
        this.seconds = seconds;
    }

    public Money calculateFee(){
        Money result = Money.ZERO;

        for (Call call : calls){
            if(call.getFrom().getHour() >= LATE_NITGHT_HOUR){
                result = result.plus(
                    nightAmount.times(call.getDuration().getSeconds() / seconds.getSeconds())
                );
            }else {
                result = result.plus(
                    regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds())
                );
            }
        }

        return result;
    }
}
```

`통화 요금에 부과할 세금`을 계산하는 요구사항을 추가해보자.

```java
public class Phone {
    ...
    private double taxRate;

    ...
      
    public Money calculateFee(){
        Money result = Money.ZERO;

        for (Call call : calls){
            result = result.plus(amount.times(call.getDuration().getSeconds() / seconds.getSeconds()));
        }

        return result.plus(result.times(taxRate);
    }
}
```

```java
public class NightlyDiscountPhone {
	  ...
    private double taxRate;
	  ...

    public Money calculateFee(){
        Money result = Money.ZERO;

        for (Call call : calls){
            if(call.getFrom().getHour() >= LATE_NITGHT_HOUR){
                result = result.plus(
                    nightAmount.times(call.getDuration().getSeconds() / seconds.getSeconds())
                );
            }else {
                result = result.plus(
                    regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds())
                );
            }
        }

        return result.minus(result.times(taxRate));
    }
}
```

이처럼, 많은 코드 속에서 어떤 코드가 중복인지리 파악하는 것은 쉬운 일이 아니다.

두 클래스 사이의 중복 제거 방법으로, 클래스를 하나로 합치고 요금제를 구분하는 `타입 코드`를 추가하는 방법이 있다. 타입 코드 값에 따라 로직을 분기시키는 것이다. 하지만 이 방법은 낮은 응집도와 높은 결합도의 문제가 있다.

##### 상속을 이용해 중복 코드 제거하기

```java
public class NightlyDiscountPhone extends Phone{

    @Override
    public Money calculateFee(){
      ...
    }
```

10시 이전의 통화요금 계산하는 경우는 Phone 에 구현된 로직을 재사용하고 10시 이후의 통화 요금 계산은 NightlyDiscountPhone 에서 구현하기로 결정한다.

이 예제처럼, 상속을 이용해 코드 재사용 하기 위해서는 부모 클래스의 개발자가 세웠던 가정이나 추론 가정을 정확하게 이해해야한다. 이것은 자식 클래스의 작성자가 부모 클래스의 구현 방법에 대한 정확한 지식을 가져야한다는 것을 의미한다.

따라서 상속은 결합도를 높인다.

##### 강하게 결합된 Phone 과 NightlyDiscountPhone

자식 클래스가 부모 클래스의 구현에 강하게 결합되면 부모 클래스의 변경에 의해 자식 클래스가 영향을 받는다. 

## 02 취약한 기반 클래스 문제

부모 클래스의 변경에 의해 자식 클래스가 영향을 받는 현상을 취약한 기반 클래스 문제라고 한다.

취약한 기반 클래스 문제는, 자식 클래스가 부모 클래스의 구현 세부 사항에 의존하도록 만들기 때문에

1. 결합도를 높인다. 
2. 캡슐화를 약화 시킨다. 

##### 불필요한 인터페이스 상속 문제

1. 다음 코드를 보자.

```java
Stack<String> stack = new Stack<>();

stack.push("1st");
stack.push("2nd");
stack.push("3rd");

stack.add(0, "4th");

assertEquals("4th", stack.pop()); // 에러 
```

반환값은 3rd 이다. 왜냐하면, Vector 의 add 메서드를 이용해서 스택의 맨 앞에 "4th" 를 추가했기 때문이다.

문제 원인은, Stack 이 규칙을 무너뜨릴 여지가 있는 위험한 Vector 의 퍼블릭 인터페이스까지도 함께 상속 받았기 때문이다.

2. 다음 코드를 보자.

```java
Properties properties = new Properties();
properties.setProperty("aaa", "C++");
properties.setProperty("bbb", "Java");

properties.put("ccc", 67);

assertEquals("C", properties.getProperty("ccc")); // 에러 
```

"ccc" 를 키로 검색하면 null 이 반환된다. Properties 의 getProperty 메서드가 반환할 값이 String 이 아니면 null 을 반환하도록 구현되어 있기 때문이다. 

Hashtable 의 인터페이스에 포함되어 있는 put 메서드를 이용해서 String 타입 이외의 키 값이라도 Properties 에 저장을 한 것이다.

##### 메서드 오버라이딩의 오작용 문제

상속은 코드 재사용을 위해 캡슐화를 희생한다.

##### 부모 클래스와 자식 클래스의 동시 수정 문제

## 03 Phone 다시 살펴보기

##### 추상화에 의존하자

부모 클래스와 자식 클래스 모두 추상화에 의존하도록 수정하자

##### 차이를 메서드로 추출하자

변하는 것으로부터 변하지 않는 걸을 분리하자. 변하는 부분을 찾고 이를 캡슐화하자.

##### 중복 코드를 부모 클래스로 올려라

```java
public abstract class AbstractPhone {
    private List<Call> calls = new ArrayList<>();
    
    public Money calculateFee(){
        Money result = Money.ZERO;
        
        for(Call calls : calls){
            result = result.plus(calculateCallFee(call));
        }
        
        return result;
    }
  
   	abstract protected Money calculateCallFee(Call call);
}
```

```java
public class Phone extends AbstractPhone {
    private Money amount;
    private Duration seconds;

    public Phone(Money amount, Duration seconds) {
        this.amount = amount;
        this.seconds = seconds;
    }

    @Override
    protected Money calculateCallFee(Call call) {
        return amount.times(call.getDuration().getSeconds() / seconds.getSeconds());
    }
}
```

```java
public class NightlyDiscountPhone extends AbstractPhone{
    private static final int LATE_NITGHT_HOUR = 22;

    private Money nightlyAmount;
    private Money regularAmount;
    private Duration seconds;

    public NightlyDiscountPhone(Money nightlyAmount, Money regularAmount, Duration seconds) {
        this.nightlyAmount = nightlyAmount;
        this.regularAmount = regularAmount;
        this.seconds = seconds;
    }

    @Override
    public Money calculateCallFee(Call call) {
        if(call.getFrom().getHour() >= LATE_NITGHT_HOUR){
            return nightlyAmount.times(call.getDuration().getSeconds() / seconds.getSeconds());
        }
        
        return regularAmount.times(call.getDuration().getSeconds() / seconds.getSeconds());
    }
}

```

##### 추상화가 핵심이다

공통 코드를 이동시킨 후에, 각 클래스는 서로 다른 변경 이유를 가진다.

1. AbstractPhone

   전체 통화 목록을 계산하는 방법이 바뀔 경우에만 변경

2. Phone

   일반 요금제의 통화 한건을 계산하는 방법이 바뀔 경우에만 변경

3. NightlyDisoucoutPhone

   심야 할인 요금제의 통화 한건을 계산하는 방법이 바뀔 경우에만 변경

그리고, 추상화에 의존함으로써

1. 부모 클래스는 자신의 내부에 구현된 추상 메서드를 호출함으로써 추상화에 의존하고,
2. 요금 계산과 상위 수준의 정책을 구현하는 AbstractPhone 이 세부적인 요금 계산 로직을 구현하는 Phone 과 NightlyDisoucoutPhone 에 의존하지 않고 그 반대로 의존하기 때문에 의존성 역전 원칙을 지키고,
3. 새로운 요금제룰 추가하기도 쉽다. AbstractPhone 을 상속받는 클래스를 추가하고 calculateCallFee 메서드만 오버라이딩 하면 된다.
4. 개방 폐쇄 원칙을 준수한다. 확장에 열려있고 수정에 닫혀 있기 때문이다.

##### 의도를 드러내는 이름 선택하기

```java
public abstract class Phone { ... }
public class RegularPhone extends Phone { ... }
public class NightlyDiscountPhone extends Phone { ... }
```

##### 세금 추가하기

통화 요금에 세금 부과하는 요구사항 추가해보자.

```java
public abstract class Phone {
    private double taxRate;
    private List<Call> calls = new ArrayList<>();

    public Money calculateFee(){
        Money result = Money.ZERO;

        for(Call calls : calls){
            result = result.plus(calculateCallFee(call));
        }

        return result.plus(result.times(taxRate));
    }

    abstract protected Money calculateCallFee(Call call);
}

```

## 04 차이에 의한 프로그래밍

기존 코드와 다른부분을 추가해서 애플리케이션의 기능을 확장하는 방법을 차이에 의한 프로그래밍이라고 한다. 

차이에 의한 프로그래밍의 목표는 중복 코드를 제거하고 코드를 재사용 하는 것이다.

객체지향 세계에서 중복 코드 제거하고 코드 재사용하는 가장 유명한 방법은 상속이다. 상속의 오용과 남용은 애플리케이션을 이해하고 확장하기 어렵게 만든다. 

"정말로 필요한 경우에만 상속을 사용해라"

---

오브젝트 <조영호>
