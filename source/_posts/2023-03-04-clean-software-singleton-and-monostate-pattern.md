---
layout: post
title: Singleton && MonoState Pattern
date: 2023-03-04
categories: Agile
---

싱글톤 패턴과 모노스테이트 패턴을 정리한다.

## Singleton Pattern

단 하나의 인스턴스를 가져야하는 클래스가 있을 수 있다.

```java
public class Singleton {
    private static Singleton theInstance = null;

    private Singleton() {

    }

    public static Singleton Instance() {
        if (theInstance == null) {
            theInstance = new Singleton();
        }

        return theInstance;
    }
}
```

## Monostate Pattern

단일성을 이루기 위한 또 다른 방법이다.

```java
public class Monostate {
    private static int itsX = 0;

    public Monostate() {

    }

    public void setX(int x) {
        itsX = x;
    }

    public int getX() {
        return itsX;
    }
}

public class Test {

    public void testInstanceBehavesAsOne() {
        Monostate m1 = new  Monostate();
        Monostate m2 = new  Monostate();

        for (int x = 0; x < 10; x++) {
            m1.setX(x);
            assetEquals(x, m2.getX());
        }
    }
}
```

싱글톤 패턴은 단일성 구조를 강제하여 둘 이상의 인스턴스가 생성되는 것을 막는다.
모노스테이트 패턴은 구조적인 제약은 없지만, 단일성이 있는 행위를 강제한다.

---

클린 소프트웨어 <로버트 C.마틴>
