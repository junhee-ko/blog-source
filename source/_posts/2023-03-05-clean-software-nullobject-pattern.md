---
layout: post
title: Null Object Pattern
date: 2023-03-05
categories: Agile
---

널 오브젝트 패턴을 정리한다.

## 기존의 not null check

```java
Employee e = DB.getEmployee("Bob");
if (e != null && e.isTimeToPay(today)) {
    e.pay();
}
```

not null 확인은 관용적인 표현이지만 보기 싫고, 에러가 발생하기 쉽다.
DB.getEmployee 가 null 대신 예외를 발생시키면 에러가 발생할 위험을 감소시킬 수 있다.
하지만, try/catch 블록이 추가되어야한다.

## Null Object Pattern 적용

널 오브젝트 패턴을 사용하면 null 검사 코드가 제거되고 코드를 단순화시킨다.

```java
public class DB {
  public static Employee getEmployee(String name) {
    return Employee.NULL;
  }
}

public interface Employee {
  public boolean isTimeToPay(Date payDae);

  public void pay();

  public static final Employee NULL = new Employee() {

    public boolean isTimeToPay(Date payDae) {
        return false;
    }

    public void pay() {

    }
  }
}

Employee e = DB.getEmployee("Bob");
if (e.isTimeToPay(today)) {
  e.pay();
}
```

없는 직원을 익명 내부 클래스로 만들어서, 인스턴스가 오직 하나임을 보장한다.
없는 직원 내부에서는, isTimeToPay 는 false 를 반환하고 아무 임금도 지급하지 않는다.

---

클린 소프트웨어 <로버트 C.마틴>
