---
layout: post
title: Classes
date: 2022-10-30
categories: Clean Code
---

깨끗한 클래스를 만드는 방법을 정리한다.

## 클래스는 작아야한다

```java
public class SuperDashboard extends JFrame implements MetaDataUser {
    public Component getLastFocusedComponent()
    public void setLastFocused(Component lastFocused)
    public int getMajorVersionNumber()
    public int getMinorVersionNumber()
    public int getBuildNumber()
}
```

위 SuperDashboard 는 책임이 너무 많다.

클래스 이름은 클래스 책임을 기술해야하는데, 클래스 이름이 모호하면 책임이 많아서이다.
예를 들면, Processor, Manager, Super

그리고 클래스 설명은 if, and, or, but 을 사용하지 않고 가능해야한다.
SuperDashboard 클래스는 다음 처럼 설명할 수 있다:
"마지막으로 포커스를 얻은 컴포넌트에 접근하는 방법을 제공하며, 버젼과 빌드 번호를 추적하는 메커니즘을 제공한다"

"~하며" 는 SuperDashboard 에 책임이 많다는 증거다.

## 단일 책임 원칙

SRP 는 클래스나 모듈을 변경할 이유 (책임) 가 단 하나뿐이어야한다는 원칙이다.
SuperDashboard 는 변경할 이유가 두 가지다.

1. 소프트웨어 버젼 정보를 추적하는데, 소프트웨어를 출시할 때마다 달라진다.
2. 스윙 컴포넌트를 관리하는데, 스윙 코드가 변경되면 버젼 번호가 달라진다.

그래서, SuperDashboard 에서 버젼 정보를 다루는 메서드를 독자적인 클래스로 만들자.

```java
public class Version {
    public int getMajorVersionNumber()
    public int getMinorVersionNumber()
    public int getBuildNumebr()
}
```

## 응집도

응집도가 높다는 것은, 클레스에 속한 메서드와 변수가 서로 의존하며 논리적인 단위로 묶인다는 것이다.
클래스는 인스턴스 변수가 작아야하고, 각 클래스 메서드는 클래스 인스턴스 변수를 하나 이상 사용해야한다.
메서드가 변수를 더 많이 사용할 수록, 메서드와 클래스의 응집도가 높다.

## 여러 작은 클래스

큰 함수를 작은 함수로 여럿으로 나누기만 해도 클래스 수가 많아진다.

변수가 아주 많은 큰 함수에서, 큰 함수 일부를 작은 함수 하나로 빼자.
그런데, 추출하려는 코드에서 큰 함수에 정의된 변수 넷을 사용한다.
만약, 네 변수를 클래스 인스턴스 변수로 승격하면 새 함수는 인수가 필요없다.

이렇게 되면 몇몇 함수가 사용하는 인스턴스 변수가 많아져 클래스의 응집도가 떨어지게 된다.
이 때 몇몇 함수가 몇몇 변수를 사용한다면 독자적인 클래스로 분리하자.

## 변경하기 쉬운 클래스

```java
public class Sql {
    public Sql(String table, Column[] columns)
    public String create()
    public String insert(Object[] fields)
    public String selectAll()
    public String findByKey(String keyColumn, String keyValue)
    ...
}
```

Sql 클래스는 변경할 이유가 두 가지라, SRP 를 위반한다.

1. 새로운 SQL 문을 지원하려면 Sql 클래스를 수정해야한다.
2. 또한, 기존 SQL 문을 수정하려면 Sql 클래스를 수정해야한다.

개선하자.

```java
abstract public class Sql {
    public Sql(String table, Column[] columns)
    abstract public String generate()
}

public class CreateSql extends Sql {
    public CreateSql(String table, Column[] columns)
    @Overrdie public String generate()
}

public class SelectSql extends Sql {
  public SelectSql(String table, Column[] columns)
  @Overrdie public String generate()
}

...
```

이제, 함수 하나를 수정해도 다른 함수가 망가질 위험이 없다.
update 문을 추가하고 싶으면, 기존 클래스 변경 없이 UpdateSql 클래스를 추가하면 된다.
즉, 확장에 개방적이고 수정에 폐쇄적이어야 한다는 OCP 를 지원한다.

## 변경으로부터 격리

요구사항은 변하기 때문에, 코드도 변한다.
상세한 구현에 의존하는 클라이언트 클래스는 구현이 바뀌면 위험해진다.
그래서 인터페이스와 추상 클래스를 통해 구현이 미치는 영향을 격리해야한다.

예를 들어보자.
Portfolio 클래스에서, 외부 TokyoStockExchange API 로 포트포리오 값을 계산한다.
Portfolio 클래스에서 TokyoStockExchange API 를 직접 호출하지 말자.

```java
public interface StockExchange {
    Money currentPrice(String symbol);
}

public Portfolio {
    private StockExchange exchange;

    public Portfolio(StockExchange exhange) {
        this.exhange = exhange;
  }
}
```

이제, TokyoStockExchange 는 StockExchange 를 구현한다.
그리고 TokyoStockExchange 클래스를 흉내내는 테스트용 클래스를 만들 수도 있다.

이처럼 테스트가 가능할 정도로 시스템 결합도를 낮추면 유연성과 재사용성도 높아진다.
결합도가 낮으면 각 시스템 요소가 다른 요소와 변경으로부터 격리된다.
결합도를 줄이면 클래스 상세 구현이 아니라 추상화에 의존해야 하는 규칙 (DIP) 을 따르는 클래스가 나온다.

---

클린코드 <로버트 C.마틴>
