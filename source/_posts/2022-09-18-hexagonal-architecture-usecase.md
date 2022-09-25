---
layout: post
title: Hexagonal Architecture - Usecase
date: 2022-09-18
categories: Architecture
---

육각형 아키텍처에서 유스케이스를 구현해보자.

## Domain Model

육각형 아키텍처는 도메인 모델의 중심 아키텍처이다.
도메인 엔티티를 먼저 만들자.

```kotlin
package com.example.buckpal.domain

data class Account(
  private val id: AccountId,
  private val baselineBalance: Money,
  private val activityWindow: ActivityWindow
) {

  fun calculateBalance(): Money =
    Money.add(
      this.baselineBalance,
      this.activityWindow.calculateBalance(id)
    )

  fun withdraw(money: Money, targetAccountId: AccountId): Boolean {
    if (!mayWithdraw(money)) {
      return false
    }

    val withdrawal = Activity(
      id,
      id,
      targetAccountId,
      LocalDateTime.now(),
      money
    )
    activityWindow.addActivity(withdrawal)

    return true
  }

  private fun mayWithdraw(money: Money): Boolean =
    Money.add(
      calculateBalance(),
      money.negate()
    )
      .isPositiveOrZero()

  fun deposit(money: Money, sourceAccountId: AccountId): Boolean {
    val deposit = Activity(
      id,
      sourceAccountId,
      id,
      LocalDateTime.now(),
      money
    )
    activityWindow.addActivity(deposit)

    return true
  }
}
```

## Usecase

이제 입금, 출금을 할 수 있는 Account 엔티티가 있으므로, 바깥 방향으로 나가자.
일반적으로 유스케이스는,

1. 입력을 받음 (from Incoming Adapter)
2. 비즈니스 규칙을 검증
3. 모델 상태를 조작
4. 출력을 반환

유스케이스는 일반적으로 비즈니스 규칙을 충족하면,
도메인 객체의 상태를 바꾼 뒤에 영속성 어뎁터를 통해 구현된 포트로 상태를 전달해서 저장될 수 있게 한다.
또한, 다른 outgoing 어뎁터를 호출할 수도 있다.

마지막으로 outgoing 어뎁터에서 온 출력 값을, 유스케이스를 호출한 어뎁터로 반환할 출력 객체로 변환해서 반환한다.

```kotlin
package com.example.buckpal.application.service

class SendMoneyService(
  private val loadAccountPort: LoadAccountPort
  private val accountLock: AccountLock
  private val updateAccountStatePort: UpdateAccountStatePort
) : SendMoneyUseCase {

  fun sendMoney(command: SendMoneyCommand): Boolean {
    // TODO: 비즈니스 규칙 검증
    // TODO: 모델 상태 조작
    // TODO: 출력 값 반환
  }
}
```

SendMoneyService 는 incoming port interface 인 SendMoneyUseCase 를 구현한다.
계좌를 불러오기 위해, outgoing interface 인 LoadAccountPort 를 호출한다.
DB 계좌 상태를 변경하기 위해, UpdateAccountStatePort 를 호출한다.

## 입력 유효성 검증

입력 유효성 검증은 유스케이스 클래스의 책임이 아니다. 하지만 여전히 application 계층의 책임이다.
호출하는 어뎁터가 유스케이스 입력을 전달하기 전에 입력 유효성을 검증하면,

1. 유스케이스에서 필요로하는 것을 caller 가 모두 검증했다고 믿을 수 없다.
2. 유스케이스는 하나 이상의 어뎁터에서 호출될텐데, 유효성 검증을 각 어뎁터에서 모두 구현해야한다.

application 계층에서 유효성 검증을 해야하는 이유는, 그렇게 하지 않으면 애플리케이션 코어의 바깥에서 유효하지 않은 입력 값을 받게 되고 모델의 상태를 해칠 수 있기 때문이다.

그런데, 유스케이스가 아니면 어디서 입력 유효성 검증을 해야하나 ? input model 의 생성자에서 하면 된다.

```kotlin
package com.example.buckpal.application.port.`in`

data class SendMoneyCommand(
  private val sourceAccountId: AccountId,
  private val targetAccountId: AccountId,
  private val money: Money
) {

  init {
    if (moeny.isNotGreaterThan(0)) throw IllegalArgumentException("Money should be greater than zero")
  }
}
```

모든 파라미터가 not null 이어야하고 송금액은 0 보다 커야한다.

SendMoneyCommand 는 유스케이스 API 의 일부이기 때문에 incoming port package 에 위치한다.
그래서, 유효성 검증이 application core 에 남아 있지만, 유스케이스를 오염시키지 않는다.

## 유스케이스마다 다른 입력 모델

각기 다른 유스케이스에 동일한 입력 모델을 사용하고 싶을 수 있다.
하지만, 각 유스케이스에 전용 입력 모델을 사용해야, 유스케이스를 명확하게 만들고 유스케이스 간 결합도를 제거해서 불필요한 부수효과를 막는다.

## 비즈니스 규칙 검증

비즈니스 규칙 검증은, 도메인 엔티티에 넣는다.

```kotlin
data class Account(
    // ...
) {

    fun withdraw(money: Money, targetAccountId: AccountId): Boolean {
        if (!mayWithdraw(money)) {
            return false
        }

        // ...
    }
}
```

만약에 도메인 엔티티에서 비즈니스 규칙 검증이 어렵다면, 유스케이스 코드에서 도메인 엔티티를 사용하기 전에 검증해도 된다.

```kotlin
class SendMoneyService : SendMoneyUseCase {

  // ...

  fun sendMoney(command: SendMoneyCommand): Boolean {
    requireAccountExists(command.sourceAccountId)
    requireAccountExists(command.targetAccountId)
    // ...
  }
}
```

## 유스케이스마다 다른 출력 모델

입력과 비슷하게 출력도 각 유스케이스에 맞게 구체적이어야한다.
유스케이스 간 출력 모델을 공유하면, 유스케이스 간에 강하게 결합하게 된다.

## 읽기 전용 유스케이스

읽기 전용 쿼리는 쓰기가 가능한 usecase 와 코드 상에서 명확하게 구분되므로, CQRS 개념과 잘 맞는다.

```kotlin
class GetAccountBalanceService(
  private val loadAccountPort: LoadAccountPort
) : GetAccountBalanceQuery {

  fun getAccountBalance(accountId: AccountId?): Money =
    loadAccountPort.loadAccount(accountId, LocalDateTime.now())
      .calculateBalance()
}
```

쿼리 서비스는 유스케이스 서비스와 동일한 방식으로 동작한다.
GetAccountBalanceQuery incoming port 를 구현하고, DB 에서 데이터를 로드하기 위해 LoadAccountPort outgoing port 를 호출한다.

---

만들면서 배우는 클린 아키텍처 <톰 홈버그>
