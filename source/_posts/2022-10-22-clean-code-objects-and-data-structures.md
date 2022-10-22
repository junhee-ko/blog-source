---
layout: post
title: Objects and Data Structures
date: 2022-10-22
categories: Clean Code
---

객체와 자료구조의 차이를 정리하자.

## 자료 추상화

자료를 세세하게 공게하기 보다, 추상적인 개념으로 표현하자.
아무 생각 없이 조회/설정 함수를 추가하지 말자.

2차원 점을 표현하는 아래 두 코드의 차이를 보자.

```java
public class Point {
    public double x;
    public double y;
}
```

위 코드는 구현을 외부에 노출한다.
변수를 private 으로 선언하더라도, 각 값마다 get, set 함수를 제공하면 구현을 외부로 노출하는 셈이다.
확실히 직교좌표계를 사용하고, 개별적으로 좌표값을 읽고 설정하게 강제한다.

```java
public interface Point {
    doube getX();
    double getY();
    void setCartesian(double x, doube y);
    double getR();
    double getTheta();
    void setPolar(double r, doube theta);
}
```

위 코드는 구현을 숨긴다.
직교좌표계를 사용하는지 극좌표계를 사용하는지 알 수 없다.
그리고 자료구조 이상을 표현한다. 클래스 메서드가 접근 정책을 강제한다. 좌표를 읽을 때는 개별적으로 읽고, 설정할 때는 두 값을 한꺼번에 설정해야한다.

아래 두 코드를 보자.

```java
public interface Vehicle {
    double getFuelTankCapacityInGallons();
    double getGallonsOfGasoline();
}
```

위 코드는, 자동차 연료 상태를 구체적인 숫자 값으로 알려준다.
두 함수가 변수값을 읽어 반환할 뿐이라는 사실이 확실하다.

```java
public interface Vehicle {
    double getPercentFuelRemaining();
}
```


위 코드는, 자동차 연료 상태를 백분율이라는 추상적인 개념으로 알려준다.
정보가 어디서 오는지 드려나지 않는다.

## 자료, 객체 비대칭

아래 두 개념은 정반대의 개념이다.
객체는, 추상화 뒤로 자료를 숨긴 채 자료를 다루는 함수만 공개한다.
자료 구조는, 자료를 그대로 공개하고 함수를 제공하지 않는다.

절차적인 도형 클래스를 보자.

```java
public class Square {
    public Point topLeft;
    public double side;
}

public class Circle {
    public Point center;
    public double radius;
}

public class Geometry {
    public final double PI = 3.1415;

    public double area(Object shape) throws NoSuchShapeException {
        if (shape instanceof Square) {
            Square s = (Square) shape;
            return s.side * s.side;
        }
        else if (shape instanceof Circle) {
            Circle c = (Circle) shape;
            return PI * c.radius * c.radius;
        }
        throw new NoSuchShapeException();
    }
}
```

다형적인 도형 클래스를 보자.

```java
public class Square implements Shape {
    private Point topLeft;
    private double side;

    public double area() {
        return side * side;
    }
}

public class Circle implements Shape {
    private Point center;
    private double radius;
    private final double PI = 3.1415;

    public double area() {
        return PI * radius * radius;
    }
}
```

절차적인 코드는 기존 자료구조를 변경하지 않고 새 함수를 추가하기 쉽다.
반면, 객체지향 코드는 기존 함수를 변경하지 않고 새 클래스를 추가하기 쉽다.

절차적인 코드는 새로운 자료구조를 추가하기 어렵다. 그러려면 모든 함수를 고쳐야한다.
객체 지향 코드는 새로운 함수를 추가하기 어렵다. 그러려면 모든 클래스를 고쳐야한다.

새로운 함수가 아니라, 새로운 자료타입이 필요한 경우가 있다. 이 때는, 클래스와 객체 지향 기법이 적합하다.
새로운 자료 타입이 아니라, 새로운 함수가 필요하면 절차저인 코드의 자료 구조가 적합하다.

## 디미터 법칙

디미터 법칙이란, 모듈은 자신이 조작하는 객체의 사정을 몰라야 한다는 법칙이다.
즉, 객체는 내부 구조를 공개하면 안된다.

아래 코드는, 디미터 법칙을 어기는 듯이 보인다.
이런 코드를 기차 충돌 ( Train Wreck ) 이라고 부른다.

```java
final String outputDir = ctxt.getOptions().getScratchDir().getAbsoltePath();
```

위 코드가 디미터 법칙을 어기는지는, ctxt, Options, ScratchDir 이 객체인지, 자료구조인지에 달렸다.
객체라면, 당연히 내부구조를 숨겨야하기 때문에  디미터 법칙을 위반한다.
자료구조라면, 당연히 내부구조를 노출하므로 디미터 법칙이 적용되지 않는다.

이렇게 구현했다면, 디미터 법칙을 거론할 필요가 없다.

```java
final String outputDir = ctxt.options.scratchDir.absoltePath;
```

만약 ctxt 가 객체라면, 뭔가를 하라고 말해야하지 속을 드러내라고 말하면 안된다.
우선 임시 디렉토리의 절대 경로를 얻어서 뭘 하려고 했는지부터 파악하자.
만약 임시 파일을 생성하기 위해서였다면, 임시 파일을 생성하라고 시키면 된다.

```java
BufferedOutpuStream bos = ctxt.createScratchFileSteam(classFileName);
```

## 잡종 구조

때로는 절반은 객체, 절반은 자료 구조인 잡종이 나온다.
중요한 기능을 수행하는 함수도 있고, 공개 변수나 공개 조회/설정 함수도 있다.
이런 구조는 새로운 함수는 물론이고, 새로운 자료 구조도 추가하기 어렵다.
되도록 피하자.

## Data Transfer Object

자료 구조체의 전형적인 형태는 공개 변수만 있고, 함수가 없는 클래스다.
이런 자료 구조체를 Data Transfer Object (DTO) 라고 한다.

---

클린코드 <로버트 C.마틴>
