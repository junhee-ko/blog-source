---
layout: post
title: 행위
date: 2022-11-27
categories: Implementation Patterns
---

## 주요 흐름

주요 제어 흐름을 명확하게 표현한다.
프로그램의 주요 흐름에는 예외도 발생할 수 있지만, 연산에는 큰 흐름이 있다.
프로그래밍 언어를 통해 주요 흐름을 명확히 표현하자.

## 메세지

메세지를 보내서 제어흐름을 표헌한다.

```java
compute() {
    input();
    process();
    output();
}
```

"이 연산을 이해하기 위해 세 단계를 알면 되고 당장 자세한 내용은 알 필요 없다" 라는 뜻을 가지고 있다.

## 선택 메세지

여러 선택 사항을 나타내기 위해 메세지 구현자를 다양화한다.
여러 방법 중 하나로 그래픽을 표시해야할 때, 런타임에 선택이 일어남을 나타내기 위해 다형적 메세지를 사용할 수 있다.

```java
public void displayShape(Shape subject, Brush brush) {
    brush.display(subject)
}
```

## 더블 디스패치

두 가지 축으로 메세지 구현자를 다양화해서 중첩된 선택을 표현한다.

```java
public void displayShape(Shape subject, Brush brush) {
  subject.displayWith(brush)
}

Oval.displayWith(Brush brush) {
    brush.displayOval(this);
}

Rectangle.displayWith(Brush brush) {
  brush.displayRectangle(this);
}
```

## 되돌림 메세지

메세지를 같은 수신자에게 보내서 제어흐름에 대칭성을 부여한다.

```java
void compuute() {
    input();
    helper.process(this);
    output:
}
```

대칭성이 떨어진다.
도우미 메서드를 사용해서 메서드 가독성을 높이자.

```java
void compuute() {
    input();
    process(this);
    output:
}

void process(Helper helper){
    helper.process(this)
}
```

## 설명 메세지

로직을 설명하기 위해 메세지를 보낸다.

```java
highlight(Rectangle area){
    reverse(area);
}
```

reverse 를 직접 호출하지 않고, highlight 를 호출하고 있다.
highlight 는 연산의 목적이 아니라, 프로그래머의 의도를 전달하기 위한 메서드이다.

## 예외 흐름

주요 흐름에 대한 표현을 방해하지 않으면서, 명확하게 예외적 제어흐름을 표헌한다.

## 보호 구문

지역적 예외 흐름은 이른 반환을 통해 표현한다.

```java
void compute() {
    Server server = getServer();
    if (server == null) return;

    // ...
}
```

## 예외

비지역적 예외 흐름은 예외로 표헌한다.

## 체크 예외

명시적 선언으로 예외를 처리한다.

## 예외 전달

예외를 전달할 때는 예외 처리자에게 적합한 정보를 전달할 수 있도록 필요에 따라 예외의 형태를 변화한다.

---

켄트 벡의 구현 패턴 <켄트 벡>
