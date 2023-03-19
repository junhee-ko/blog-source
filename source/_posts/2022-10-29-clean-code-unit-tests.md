---
layout: post
title: Unit Tests
date: 2022-10-29
categories: Clean Code
---

테스트 코드가 방치되어 망가지면, 실제 코드도 망가진다.
테스트 코드를 깨끗하게 유지하는 방벙을 정리하자.

## TDD 법칙 세 가지

1. 실패하는 단위 테스트를 작성하기 전에, 실제 코드를 작성하지 않는다.
2. 컴파일은 실패하지 않으면서 실행이 실패하는 정도로 단위 테스트를 작성한다.
3. 현재 실패하는 테스트를 통과할 정도로 실제 코드를 작성한다.

위와 같이 하면, 실제 코드를 전부 테스트하는 테스트 케이스가 나온다.
하지만 방대한 테스트 코드는 심각한 관리 문제를 유발하기도 한다.

## 깨끗한 테스트 코드 유지하기

테스트 코드는 실제 코드 못지 않게 중요하다.
테스트 코드가 복잡할 수록,

1. 실제 코드를 작성하는 시간보다, 테스트 케이스를 추가하는 시간이 더 걸린다.
2. 실제 코드를 변경해 기존 테스트 케이스가 실패하면, 실패 테스트 케이스를 더 통과하기 어려워 진다.
3. 팀이 테스트 케이스를 유지 보수하는 비용도 늘어나게 된다.

하지만 테스트 슈트가 없으면 수정한 코드가 제대로 동작하는지 확인할 방법이 없다.
테스트 케이스가 있으면 변경이 두렵지 않다.
테스트 케이스가 없다면 모든 변경이 잠적정인 버그이다.

## 깨끗한 테스트 코드

깨끗한 테스트 코드를 만들려면, 가독성이 중요햐다.
가독성을 높이려면 명료성, 단순성, 풍부한 표현력이 필요하다.

실제 코드만큼 효율적일 필요는 없다.
실제 환경이 아니라 테스트 환경에서 동작하기 때문이다.

## 테스트 당 assert 하나

JUnit 으로 테스트 코드를 작성할 때, 함수마다 assert 문 단 하나만을 사용해야한다는 주장이 있다.
결론이 하나라서, 코드를 이해하기 쉽고 빠르다.

하지만 테스트를 분리하기 때문에 중복 코드가 많아진다.
중복 코드를 아래 방법으로 해결할 수도 있다.

1. Template Method 패턴으로 given/when 부분을 부모클래스에, then 부분을 자식클래스에 두어서
2. 독자적인 테스트 클래스의 @Before 에 given/when 부분을, @Test 함수에 then 부분을 두어서

하지만 과하다.

## 테스트 당 개념 하나

테스트 함수마다 하나의 개념만 테스트하고, 개념당 assert 문 수를 최소로 하자.
아래 처럼, 여러 개념을 테스트하는 긴 함수는 피하자.

```java
public void testAddMonths() {
    SerialDate d1 = SerialDate.createInstance(31, 5, 2004);

    SerialDate d2 = SerialDate.addMonths(1, d1);
    assertEquals(...);
    assertEquals(...);
    assertEquals(...);

    SerialDate d3 = SerialDate.addMonths(2, d1);
    assertEquals(...);
    assertEquals(...);
    assertEquals(...);
}
```

위 테스트 코드는 assert 문이 여럿이여서 문제가 아니다.
한 테스트 함수에 여러 개념을 테스트한다는 것이 문제다.

##  FIRST

깨긋한 테스트는 다음 다섯 가지 규칙을 따른다.

1. Fast: 테스트는 빨라야한다. 느리면 자주 돌릴 염두가 나지 않는다.
2. Independent: 각 테스트는 독립적이어야한다. 테스트가 서로 의존하면 실패할 때 원인을 찾기 어렵다.
3. Repeatable: 어떤 환경에서도 반복 가능해야한다. 실제 환경, QA 환경, 네트웍이 단절된 환경..
4. Self-Validating: 테스트는 bool 값 결과를 내야한다. 통과 여부를 로그 파일로 확인하면 안된다.
5. Timely: 적시에 작성해야한다. 테스트 하려는 실제 코드 구현 직전에 테스트를 작성해야한다.

---

클린코드 <로버트 C.마틴>