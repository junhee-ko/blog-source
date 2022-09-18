---
layout: post
title: Hexagonal Architecture
date: 2022-09-18
categories: Architecture
---

육각형 아키텍처의 패키지 구조를 정리한다.
본인의 계좌에서 다른 계좌로 송금을 하는 "송금하기" usecase 를 예로 들어보자.

## 계층으로 구성한 패키지 구조

우선, 계층으로 코드를 구성해보자.

- buckpal
  - domain에
    - Account
    - Activity
    - AccountRepository
    - AccountService
  - persistence
    - AccountRepositoryImpl
  - web
    - AccountController

웹, 도메인, 영속성 계층에 대해 전용 패키지를 구성했다.
그리고, 의존성 역전 원칙을 이용해서 의존성이 domain package 에 있는 도메인 코드만을 향한다.
domain package 에 AccountRepository 를, persistence package 에 AccountRepositoryImpl 를 두어서 의존성을 역전시켰다.

### 계층으로 구성한 패키지 구조의 문제

첫 번째로, application 의 기능 조각을 구분 짓는 package 경계가 없다.
사용자를 관리하는 기능이 추가되면, 서로 연관되지 않은 기능들이 부수효과를 일으킬 수 있다.

- buckpal
  - domain
    - Account
    - Activity
    - AccountRepository
    - AccountService
    - UserService <---- HERE
    - UserRepository <---- HERE
    - User <---- HERE
  - persistence
    - AccountRepositoryImpl
    - UserRepositoryImpl <---- HERE
  - web
    - AccountController
    - UserController <---- HERE

두 번째로, 애플리케이션이 어떤 usecase 를 제공하는지 파악할 수 없다.
AccountService, AccountController 가 어떤 usecase 를 구현하는지 파악할 수 없다.

세 번째로, 패키지 구조를 통해서 아키텍처를 파악할 수 없다.

## 기능으로 구성한 패키기 구조

위 계층으로 구성한 구조를 기능으로 구성해서 문제를 해결해보자.

- buckpal
  - account
    - Account
    - AccountController
    - AccountRepository
    - AccountRepositoryImpl
    - SendMoneyService

계좌와 관련된 코드를 account package 에 넣었고, 계층 패키지를 없앴다.
또한, package-private 접근 수준으로 외부에서 접근하면 안되는 클래스들에 대해 경계를 강화할 수 있다.
그리고 AccountService 의 책임을 좁히기 위해, SendMoneyService 로 클래스명을 변경했다.

### 기능으로 구성한 패키기 구조의 문제

첫 번째로, 여전히 기반 아키텍처가 명확히 보이지않는다.
두 번째로, SendMoneyService 가 AccountRepository 인터페이스만 알고 있다고 해도 도메인 코드가 실수로 영속성 코드에 의존하는 것을 막을 수 없다.

## 육각형 아키텍처 패키지 구조

육각형 아키텍처에서 구조적으로 핵심적인 요소는 다음과 같다.

- entity
- usecase
- incoming/outcoming port
- incoming/outcoming adapter

육각형 아키텍처를 기반으로, 패키지 구조를 다시 잡아보자.

- buckpal
  - account
    - adapter
      - in
        - web
          - AccountController
      - out
        - persistence
          - AccountPersistenceAdapter
          - SpringDataAccountRepository
    - domain
      - Account
      - Activity
    - application
      - SendMoneyService
      - port
        - in
          - SendMoneyUseCase
        - out
          - LoadAccountPort
          - UpdateAccountStatePort

최상위에는 Account 와 관련된 usecase 를 구현한 모듈임을 나타내는 account package 가 있다.
그 하위에는 domain, application, adapter 패키지가 있다.

domain package 에는, 도메인 모델이 속한다.

application package 에는, 도메인 모델을 둘러싼 서비스 계층이 포함된다.
SendMoneyService 는 incoming port interface 인 SendMoneyUseCase 를 구현한다.
그리고, SendMoneyService 는 outgoing port interface 이자 영속성 어뎁터에 의해 구햔된 LoadAccountPort 와 UpdateAccountStatePort 를 사용한다.

adapter package 에는, application 계층의 incoming port 를 호출하는 incoming adapter 를 포함한다.
그리고, application 계층의 outgoing port 에 대한 구현을 제공하는 outgoing adapter 를 포함한다.

---

만들면서 배우는 클린 아키텍처 <톰 홈버그>
