---
layout: post
title: 상태
date: 2022-11-26
categories: Implementation Patterns
---

## 직접 접근

객체 내의 상태를 직접 접근한다.
직접 접근의 장점은 표현의 명확성이다.

```java
x = 10;
```

## 간접 접근

좀더 나은 유연성을 위해 메서드를 통해 상태에 접근한다.

```java
Rectangle void setWitdh(int width) {
    this.width = width;
    area = width * height;
}
```

## 공용 상태

클래스의 모든 인스턴스에 적용되는 상태는 필드에 저장한다.

```java
class Point {
    int x;
    int y;
}
```

## 가변 상태

같은 클래스의 인스턴스마다 다른 상태를 유지해야 할 경우 상태를 맵에 저장한다.

```java
Map<String, Object> props = new HashMap<String, Object>();
```

## 지역변수

단일 범위 내에서만 유효한 상태를 저장한다.

## 필드

객체가 생성될 때부터 소멸될 때까지 상태를 저장한다.

## 파라미터

메서드가 활성화된 동안 상태를 저장한다.

## 수집 파라미터

여러 개의 메서드를 통해 복잡한 결과를 얻기 위해 파라미터를 전달한다.

```java
List resutls = new ArrayList();
addTo(results);
```

## 옵션 파라미터

어떤 메서드는 파라미터가 전달되지 않으면, 기본 파라미터를 사용한다.
이 경우, 반드시 필요한 파라미터를 앞에서 전달하고, 옵션 파라미터는 뒤에 전달할 수 있다.

```java
public ServerSocket()
public ServerSocket(int port)
public ServerSocket(int port, int backlog)
```

## 가변 인자

임의의 수의 인자를 사용해서 메서드를 호출 할 수 있다.

```java
method(Class.. classes)
```

## 파라미터 객체

자주 사용하는 긴 파라미터 목록은 객체로 만들어서 통합한다.

## 상수

변하지 않는 상태는 상수로 저장한다.

## 역할 제시형 작명

변수 이름은 연산에서의 역할을 반영하자.

## 초기화

변수 초기화는 선언적으로 하자.

## 열성적 초기화

인스턴스가 생성될 때 필드를 초기화한다.

```java
class Point {
    int x, y;

    Point(int x, int y){
        this.x = x;
        this.y = y;
    }
}
```

## 게으른 초기화

초기화 비용이 높은 객체이면, 실제 사용되기 직전에 초기화한다.

```java
Library.Collection<Person> getMembers() {
    if (members = null){
        members = new ArrayList<Person>();
    }

    return members;
}
```

---

켄트 벡의 구현 패턴 <켄트 벡>
