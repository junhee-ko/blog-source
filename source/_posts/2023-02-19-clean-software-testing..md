---
layout: post
title: Testing
date: 2023-02-19
categories: Agile
---

단위 테스트를 작성하는 것은 단순한 검증이 아니라 설계의 문제이고 문서화의 문제이다.

## 테스트 주도 개발

테스트 코드를 먼저 작성하면,

1. 프로그램의 모든 단일 함수의 동작을 검증하는 테스트를 갖게 된다.
2. 프로그래머가 다른 관점에서 문제를 해결할 수 있다. (프로그램의 호출자 관점에서, 편리하게 호출할 수 있는 소프트웨어 설계)
3. 테스트 가능한 프로그램을 설계하도록 강제할 수 있다. (소프트웨어를 다른 환경과 분리하도록 강제)
4. 테스트가 문서화의 형태로 기능한다.

### 테스트 우선 방식 설계의 예

```java
public void testMove() {
  WumpusGame g = new WumpusGame();
  g.conect(4, 5, "E");
  g.setPlayerRoom(4);
  g.east();
  assertEquals(5, g.getPlayerRoom());
}
```

위 코드는 WumpusGame 의 어떤 부분보다 먼저 작성되었다.
자신의 의도를 구현하기 전에, 먼저 그 의도를 단순하고 읽기 편하게 만들어 테스트로 제시한다.

### 테스트 분리

운영 코드를 만들기 전에, 테스트를 먼저 작성하면 소프트웨어에서 분리해야할 부분이 드러나곤한다.
예를 들어,

1. Payroll 클래스는 EmployeeDatabase 클래스를 이용해 Employee 객체를 꺼낸다.
2. Employee 에 임금을 계산하도록 요청하고 CheckWriter 객체에 그 임금을 넘겨 수표를 만든다.
3. Employee 객체에 임금을 지급하고 그 객체를 다시 DB 에 기록한다.

이제 Payroll 객체의 행위를 명시하는 테스트를 작성해야한다.
그런데,

1. 어떤 DB 를 사용한 것인가 ?
2. 적절한 수표가 출려되는지 어떻게 검증할 수 있는가 ?

이 문제의 해결책은 Mock Object 패턴을 이용하는 것이다.
Payroll 의 모든 관련 요소 사이에 인터페이스를 추가하고, 인터페이스를 구현하는 test stub 을 생성한다.

```java
public void testPayroll(){
  MockEmployeeDatabase db = new MockEmployeeDatabase()
  MockCheckWriter w = new MockCheckWriter()
  Payroll p = new Payroll(db, w)

  p.payEmployees();

  assertTrue(w.checksWereWrittenCorrectly());
  assertTrue(db.paymentWerePostedCorrectly());
}
```

### 코드보다 테스트를 먼저 작성하면 설계가 개선된다

Payroll 을 다른 객체와 분리하였다.
이로써, 애플리케이션의 확장이란 측면에서 다른 DB 와 수표 기록기로 교체할 수 있게 되었다.
이 분리 작업이 테스트에 대한 필요에서 촉발되었다.

## 인수 테스트

단위 테스트는 시스템의 작은 구성 요소가 기대한 대로 동작하는지 검증한다.
하지만, 시스템이 전체로서 제대로 동작하는지 검증하지 않는다.
인수 테스트는 고객의 요구 사항을 충족하는지 검증한다.

```javascript
AddEmp 1429 "jko" 1243.12
Payday
Verify Paycheck EmpId 1429 GrossPay 1243.12
```
---

클린 소프트웨어 <로버트 C.마틴>
