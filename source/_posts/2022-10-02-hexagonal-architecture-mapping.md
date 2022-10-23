---
layout: post
title: Hexagonal Architecture - Mapping
date: 2022-10-02
categories: Architecture
---

웹, 애플리케이션, 도메인, 영속성 각 계층의 모델을 매핑하는 전략들을 정리하자.

- 매핑하지 않기
- 양방향 매핑
- 완전 매핑
- 단방향 매핑

## 매핑하지 않기

![](/image/not-mapping-between-layers.png)

포트 인터페이스가 도메인 모델을 입출력 모델로 사용하면, 계층 간 매핑이 필요없다.

하지만, 웹 계층과 영속성 계층의 모델에서는 특별한 요구사항이 있을 수 있다.
웹 계층에서는 JSON 으로 직렬화하기 위한 annotation 을 모델 클래스의 특정 필드에 붙여야할 수 있다.
영속성 계층에서는, DB 매핑을 위해 특정 annotation 이 필요할 수 있다.

또한, Account 클래스는 각 계층과 관련된 이유로 변경되어야해서 단일 책임 원칙을 위반한다.
각 계층이 Account 클래스에 특정 커스텀 필드를 두도록 요구할 수도 있다.

## 양방향 매핑

![](/image/two-way-mapping-between-layers.png)

각 어뎁터가 전용 모델을 가지고 있다.
그래서, 해당 모델을 도메인 모델로, 도메인 모델을 해당 모델로 매핑할 책임이 있다.
안쪽 계층은 해당 계층의 모델만 알면되고 도메인 로직에만 집중한다.

하지만, 도메인 모델이 계층 경계를 넘어서 통신하는데 사용되는 것이 단점이다.
도메인 모델은 도메인 모델의 필요에 의해서만 변경되는 것이 이상적인데, 바깥쪽 계층의 요구에 따라 변경에 취약하다.

## 완전 매핑

![](/image/full-mapping-between-layers.png)

각 연산이 전용 모델을 필요해서, 웹 어뎁터와 애플리케이션 계층 각각이 자신의 전용 모델을 각 연산을 실행하는데 필요한 모델로 매핑한다.
이러한 커멘드 객체는 애플리케이션 계층의 인터페이스를 해석할 여지 없이 명확하게 만들어준다.
각 유스케이스는 전용 필드와 유효성 검증 로직을 가진 전용 커멘드를 가진다.

## 단방향 매핑

![](/image/one-way-mapping-between-layers.png)

한 계층이 다른 계층으로부터 객체를 받으면 해당 계층에서 이용하기 위해 한 방향으로 매핑한다.
모든 계층의 모델들이 같은 인터페이스를 구현한다.
이 인터페이스는 관련 있는 attribute 에 대한 getter 메서드를 제공해서 도메인 모델의 상태를 캡슐화한다.
행동을 변경하는 것이 상태 인터페이스에 노출되어 있지 않아서, 실수로 도메인 객체의 상태를 변경하는 경우가 없다.

---

만들면서 배우는 클린 아키텍처 <톰 홈버그>