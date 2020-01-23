---
layout: post
title:  "[오브젝트] 13장_서브클래싱과 서브타이핑"
date:   2020-01-13
categories: Object
---

상속의 두가지 용도는 다음과 같다.

1. 타입 계층 구현

   동일한 메세지에 대해 서로 다르게 행동할 수 있는 다형적인 객체를 구현하기 위해서는 객체의 행동을 기반으로 타입 계층을 구성해야한다.

2. 코드 재사용

   부모 클래스와 자식 클래스가 강하게 결합되기 때문에 변경하기 어려운 코드를 얻게 된다.

## 01 타입

타입을 세 가지 관점으로 정리하자.

##### 개념 관점의 타입

1. 타입

   우리가 인지하는 세상의 사물의 종류를 의미한다.

   자바, 루비, C 를 프로그래밍 언어로가 부를 때, 이것들을 프로그래밍 언어라는 타입으로 분류하고 있는 것이다.

2. 인스턴스

   어떤 대상이 타입으로 분류될 때 그 대상을 타입의 인스턴스라고 한다. 

   자바, 루비, C 는 프로그래밍 언어의 인스턴스이다.

##### 프로그래밍 언어 관점의 타입

하드웨어는 데이터를 0과 1로 구성된 일련의 비트 조합으로 취급한다. 

프로그래밍 언어 관점의 타입은, 비트 묶음에 의미를 부여하기 위해 정의된 제약과 규칙을 의미한다.

타입은 두가지 목적으로 사용된다.

1. 타입에 수행될 수 있는 유효한 오퍼레이션의 집합을 정의한다.

   자바의 '+' 연산자는 원시형 숫자 타입이나 문자열 타입의 객체에는 사용할 수 있지만 다른 클래스의 인스턴스에 대해서는 사용할 수 없다.

2. 타입에 수행되는 오퍼레이션에 대해 미리 약속된 문맥을 제공한다.

   자바에서 a + b 라는 연산이 있을 때 a, b 의 타입이 int 라면 두 수를 더한다. a, b 의 타입이 String 이면 두 문자열을 하나의 문자열로 합친다.

##### 객체 지향 패러다임 관점의 타입

프로그래밍 언어 관점에서 타입은 호출 가능한 오퍼레이션의 집합이다. 

객체지향 프로그래밍에서 오퍼레이션은 객체가 수신할 수 있는 메세지이다. 

객체지향 프로그래밍에서 타입을 정의하는 것은 객체가 수신할 수 있는 객체의 퍼블릭 인터페이스를 정의하는 것이다.

동일한 퍼블릭 인터페이스를 제공하는 객체들은 동일한 타입으로 분류된다. 

## 02 타입 계층

##### 타입 사이의 포함관계

타입 계층을 구성하는 두 타입 간의 관계에서,

1. 슈퍼타입

   더 일반적인 타입

2. 서브타입

   더 특수한 타입

##### 객체지향 프로그래밍과 타입 계층

핵심은, 서브타입의 인스턴스는 슈퍼타입의 인스턴스로 간주 될 수 있다는 것이다.

퍼블릭 인터페이스의 관점에서,

1. 슈퍼타입

   서브타입이 정의한 퍼블릭 인터페이스를 일반화시켜 상대적으로 범용적이고 넓은 의미로 정의한 것

2. 서브타입

   슈퍼타입이 정의한 퍼블릭 인터페이스를 특수화시켜 상대적으로 구체적이고 좁은 의미로 정의한 것

## 3 서브클래싱과 서브타이핑

##### 언제 상속을 사용해야 하는가 ?

다음 주 질문의 답이 '예' 이면 상속을 사용해라.

1. 상속 관계가 is-a 관계를 모델링 하는가 ?
2. 클라이언트 입장에서 부모 클래스의 타입으로 자식 클래스를 사용해도 되는가 ? (행동 호환성)

##### is-a 관계

타입 S 가 타입 T 의 일종이라면 "타입 S 는 타입 T 다"

하지만 is-a 관계가 생각처럼 직관적이고 명쾌한 것은 아니다. 다음 예를 보자.

1. 팽귄은 새다.
2. 새는 날 수 있다.

이를 코드로 옮기면 다음과 같다.

```java
public class Bird {
    public void fly() {
        ...
    }
}

public class Penguin extends Bird {
    ...
}

```

그런데, 팽귄은 새는 맞지만 날 수 없다. 위 코드에서는, 팽귄은 새이고 날 수 있다는 것을 주장한다.

이 예는, 어휘적인 정의가 아니라 기대되는 행동에 따라 타입 계층을 구성해야한다는 사실을 보여준다.

즉,`어떤 두 대상을 언어적으로 is-a 라고 표현할 수 있어도 일단은 상속을 사용할 예비 후보 정도로 생각`해야한다.

##### 행동 호환성

두 타입 사이에 행동이 호환될 경우에만 타입 계층으로 묶어야 한다.

중요한 것은, `행동의 호환 여부를 판단하는 기준은 클라이언트의 관점`이다. 클라이언트가 두 타입이 동일하게 행동할 것이라고 기대하면 두 타입을 타입 계층으로 묶을 수 있다.

Penguin 이 Bird 의 서브 타입이 아닌 이유는, 클라이언트 입장에서 모든 새가 날 수 있다고 가정하기 때문이다.

다음과 같이 클라이언트가 날 수 있는 새만을 원한다고 해보자.

```java
public void flyBird(Bird bird){
		bird.fly(); // 인자로 전달된 모든 bird 는 날 수 있어야한다.
}
```

Penguin 은 날 수 없고 클라이언트 입장에서 모든 bird 가 날 수 있기를 기대하기 때문에 flyBird 의 메서드로 전달되어서는 안된다.

상속 관계를 유지하면서 문제를 해결하기 위해 시도할 수 있는 세가지 방법이 있다.

1. Penguin 이 fly 메서드를 오버라이딩해서 내부 구현을 비워두는 것이다.

   ```java
   public class Penguin extends Bird {
       @Override
       public void fly() {
           
       }
   }
   ```

   하지만, 이 방법은 어떤 행동도 수행하지 않기 때문에 모든 bird 가 날 수 있다는 클라이언트의 기대를 만족하지 않는다. 

2. Penguin 의 fly 메서드를 오버라이딩한 후 예외를 던지는 것이다.

   ```java
   public class Penguin extends Bird {
       @Override
       public void fly() {
           throw new UnsupportedOperationException();
       }
   }
   ```

   flyBird 메서드는 모든 bird 가 날 수 있다고 가정한다. 

   flyBird 메서드 fly 메시지를 전송한 결과로 UnsupportedOperationException 예외가 던져질 것이라고 기대하지 않을 것이다.

3. flyBird 메서드를 수정해서 인자로 전달된 bird 타입이 팽귄이 아닐 경우에만 fly 메세지를 전송하는 것이다.

   ```java
    public void flyBird(Bird bird){
   		if (!(bird instanceof Penguin)){
   				bird.fly();    
   		}
    }
   ```

   만약 팽귄이 이외에 날 수 없는 또 다른 새가 상속 계층에 추가되면, flyBird 메서드 안에 새로운 타입을 체크하는 코드가 추가된다. 이것은 구체적인 클래스에 대한 결합도를 높여, 개방-폐쇄 원칙을 위반한다.

##### 클라이언트의 기대에 따라 계층 분리하기

문제 해결을 위해서는, 위 세가지 방법 말고 클라이언트의 기대에 따라 계층을 분리해야한다.

날 수 있는 새와 날 수 없는 새를 명확하게 구분할 수 있게 상속 계층을 분리하면 서로 다른 요구사항을 가진 클라이언트를 만족 시킬 수 있다.

```java
public class Bird {
 ...    
}

public class FlyingBird extends Bird{
    public void fly(){
        ...
    }
}

public class Penguin extends Bird {
   ...
}
```

```java
public void flyBird(FlyingBird bird){
		bird.fly();
}
```

또 다른 방법으로는, 다음 그림과 같이 클라이언트에 따라 인터페이스를 분리하는 것이다.

![](/image/object_subclass_interface.png)

더 좋은 방법은, 합성을 사용하는 것이다.

![](/image/object_subclass_composition.png)

설계가 꼭 현실 세계를 반영할 필요는 없다. `자연어에 현혹되지 말고 요구사항 속에서 클라이언트가 기대하는 행동에 집중해라.`

##### 서브클래싱과 서브타이핑

상속을 사용하는 목적에 따라 다음과 같이 나눌 수 있다.

1. 서브클래싱

   다른 클래스의 코드를 재사용할 목적으로 상속을 사용하는 경우이다.

   구현상속, 클래스 상속이라고도 부른다. 왜냐하면, 내부 구현 자체를 상속하는 것에 초점을 맞추기 때문이다.

2. 서브타이핑

   타입 계층을 구성하기 위해 상속을 사용하는 것이다.

   인터페이스 상속이라고도 부른다. 왜냐하면, 서브타입이 슈퍼타입의 퍼블릭 인터페이스를 상속하는 것 처럼 보이기 때문이다.

## 04 리스코프 치환 법칙

리스코프 치환 법칙이란, 클라이언트가 차이점을 인식하지 못한 채 파생 클래스의 인터페이스를 통해 서브 클래스를 사용할 수 있어야한다는 것이다.

10장의 Stack 과 Vector 는 리스코프 치환 법칙을 위반하는 전형적인 예이다. 클라이언트가 부모 클래스인 Vector 에 대해 기대하는 행동을 Stack 에 대해서는 기대할 수 없기 때문에 행동 호환성을 만족하지 못하기 때문이다.

다른 예로, "직사각형은 사격형이다 " 가 있다. 하지만, 직사각형은 사각형이 아닐 수 있다.

다음은 사각형이다.

```java
public class Rectangle { 
    private int x, y, width, height;

    public Rectangle(int x, int y, int width, int height) {
        this.x = x;
        this.y = y;
        this.width = width;
        this.height = height;
    }

    public int getWidth() {
        return width;
    }

    public void setWidth(int width) {
        this.width = width;
    }

    public int getHeight() {
        return height;
    }

    public void setHeight(int height) {
        this.height = height;
    }

    public int getArea() {
        return width * height;
    }
}
```

다음은 정사각형이다.

```java
public class Square  extends Rectangle{

    public Square(int x, int y, int size) {
        super(x, y, size, size);
    }

    @Override
    public void setWidth(int width) {
        super.setWidth(width);
        super.setHeight(width);
    }

    @Override
    public void setHeight(int height) {
        super.setWidth(height);
        super.setHeight(height);
    }
}
```

문제는 다음과 같이, Rectangle 과 협력하는 클라이언트는 사각형의 너비와 높이가 다르다고 가정한다.

```java
public void resize(Rectangle rectangle, int width, int height){
		rectangle.setWidth(width);
		rectangle.setHeight(height);
		assert rectangle.getWidth() == width && rectangle.getHeight() == height;
}
```

다음과 같이, 위 코드에서 resize 메서드 인자로 Rectangle 대신, Square 를 전달한다고 해보자. 

```java
Square square = new Square(10,10,10);
resize(square, 50, 100);
```

메서드의 실행이 실패하고 만다.

중요한 것은, 클라이언트 입장에서 행동이 호환되는지의 여부이다. 행동이 호환될 경우, 자신 클래스가 부모 클래스 대신 사용될 수 있다.

##### 클라이언트와 대체 가능성

Square 가 Rectangle 을 대체할 수 없는 이유는, 클라이언트 관점에서 Square와 Rectangle 이 다르기 때문이다.

대체 가능성을 결정하는 것은 클라이언트이다.

##### is-a 관계 다시 살펴보기

상속이 서브타이핑을 위해 사용될 경우에만 is-a 관계이다. `서브 클래싱을 구현하기 위해 상속을 사용했다면 is-a 관계가 아니다.`

##### 리스코프 치환 원칙은 유연한 설계의 기반이다.

클라이언트 입장에서, 퍼블릭 인터페이스의 행동방식이 변경되지 않는다면 클라이언트의 코드를 변경하지 않고도 새로운 자식 클래스와 협력할 수 있다.

8장에서 중복 할인 정책을 구현하기 위해 기존의 DiscountPolicy 상속 계층에 새로운 자식 클래스인 OverlappedDiscountPolicy 를 추가하더라도 클라이언트를 수정할 필요가 없었다.

![](/image/object_subclass_chapter8.png)

위 설계는 다음 원칙을 조합한 유연할 설계이다.

1. 의존성 역전 원칙

   상위 수준 모듈인 Movie 와 하위 수준 모듈인 OverlappedDiscountPolicy 모두 추상 클래스인 DiscountPolicy 에 의존한다.

2. 리스코프 치환 원칙

   OverlappedDiscountPolicy 는 클라이언트에 대한 영향 없이도 DiscountPolicy 를 대체할 수 있다.

3. 개방-폐쇄 원칙

   중복할인이라는 새로운 기능을 추가하기 위해 OverlappedDiscountPolicy 를 추가하더라도, Movie 에는 영향이 없다.

## 5. 계약에 의한 설계와 서브타이핑

'계약에 의한 설계' 란, 클라이언트와 서버 사이의 협력을 의무와 이익으로 구성된 계약의 관점에서 표현하는 것이다.

서브타입이 리스코프 치환 원칙을 만족시키기 위해서는, 클라이언트와 슈퍼타입 간에 체결된 '계약' 을 준수해야한다.

즉, 서브타입이 슈퍼타입처럼 보일 수 있는 유일한 방법은, 클라이언트가 슈퍼타입과 맺은 계약을 서브 타입이 준수하는 것이다.

---

오브젝트 <조영호>