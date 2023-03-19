---
layout: post
title: Factory Pattern
date: 2023-03-10
categories: Agile
---

Factory Pattern 을 정리한다.

## new Circle 예시

DIP 에 따르면, 구체 클래스에 의존하지 말고 추상 클래스에 의존해야한다.
다음 코드는 DIP 를 위반한다.

```java
Circle c = new Circle(origin, 1)
```

Circle 은 구체 클래스이기 때문에, Circle 인스턴스를 생성하는 모듈은 DIP 를 어긴다.

다만, DIP 위반이 해롭지 않은 경우가 있다.
구체 클래스가 쉽게 변경되는 종류의 클래스가 아닌 경우이다. ex) String

## 팩토리 패턴 적용

팩토리 패턴을 사용하면 추상 인터페이스에만 의존하면서 구체적인 인스턴스를 만들 수 있다.

```java
public interface Shape

public interface ShapeFactory {
  public Shape makeCircle();

  public Shape makeSquare();
}

public class ShapeFactoryImplementation implements ShapeFactory {

  public Shape makeCircle() {
    return new Circle();
  }

  public Shape makeSquare() {
    return new Square();
  }
}
```

## trade-off

DIP 를 엄격히 적용하면, 쉽게 변경되는 종류의 모든 클래스마다 팩토리를 사용해야한다.
하지만, 항상 팩토리를 사용하면 팩토리를 사용해서 새로운 클래스를 만들기 위해 필요한 인터페이스/클래스가 많아진다.
팩토리의 필요성이 충분히 커지면 그 때 도입하자.

---

클린 소프트웨어 <로버트 C.마틴>
