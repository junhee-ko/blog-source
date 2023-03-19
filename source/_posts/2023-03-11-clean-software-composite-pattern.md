---
layout: post
title: Composite Pattern
date: 2023-03-11
categories: Agile
---

Composite Pattern 을 정리한다.

## Sensor-Command 예제

Sensor 는 무엇을 감지하면 Command 의 do() 를 호출한다.
Command 를 하나 이상 실행해야하는 경우도 있다.
하나 이상 실행해야할 때는, Sensor 가 목록을 순회하면서 각 Command 의 do() 를 호출한다.

![](/image/clean-software-composite-pattern-sensor-command-ex.png)

## Composite Pattern 적용

Command 에 복수성 개념을 추가해서 다음과 같이 Composite 패턴을 사용할 수 있다.

![](/image/clean-software-composite-pattern-sensor-command-with-composite.png)

## 다수성

composite 패턴을 사용하면 일대다 관계 없이도 일대다의 행위를 할 수 있다.
클래스의 클라이언트마다 목록 관리와 순환 코드를 중복해서 사용하는 대신, 그 코드가 컴포지트 클래스에서만 사용하게 된다.

---

클린 소프트웨어 <로버트 C.마틴>
