---
layout: post
title: Hexagonal Architecture - Web Adapter
date: 2022-09-24
categories: Architecture
---

육각형 아키텍처에서 웹 어뎁터를 구현해보자.

## 의존성 역전

웹 어뎁터는 incoming adapter 이다. 외부에서 요청을 받아 애플리케이션 코어를 호출해서 무슨 일을 해야하는지 알려준다.

애플리케이션 계층은 웹 어뎁터가 통신할 수 있는 포트를 제공한다. 서비스는 이 포트를 구현하고, 웹 어뎁터는 이 포트를 호출한다.

웹 어뎁터와 유스케이스 사이에 간접 계층 (port) 를 두는 이유는, 애플리케이션 코어가 외부 세계와 통신하는 명세가 포트이기 때문이다. 포트를 적절한 곳에 두면, 외부와 어떤 통신을 하고 있는지 정확히 알 수 있다.

![](/image/incoming-adapter-and-port.png)

## 웹 어뎁터의 책임

웹 어뎁터는 다음 일을 한다.

1. HTTP 요청을 객체로 매핑: HTTP 요청의 컨텐츠와 파라미터를 객체로 직렬화
2. 권한 검사
3. 입력 유효성 검증
4. 입력을 유스케이스 입력 모델로 매핑
5. 유스케이스 호출
6. 유스케이스의 출력을 HTTP 로 매핑
7. HTTP 응답을 반환

## 컨트롤러 나누기

계좌 리소스와 관련된 모든 것이 AccountController 에 모여 있으면,

1. 시간이 지나 코드가 길어지면, 파악하기 힘들다.
2. 특정 프로덕션 코드에 해당하는 테스트 코드를 찾기 어렵다.
3. 데이터 구조를 재활용할 가능성이 있고, 이는 부수효과를 가져온다.
	( AccountController 의 모든 연산에서 AccountResource 를 재활용 )

그래서, 별도의 패키지에 별도의 컨트롤러를 만들어야한다.
그리고, 가급적 메서드와 클래스명은 유스케이스를 최대한 반영해야한다.

```kotlin
package com.example.buckpal.adapter.`in`.web

@RestController
class SendMoneyController(
  private val sendMoneyUseCase: SendMoneyUseCase
) {

  @PostMapping("/accounts/send/{sourceAccountId}/{targetAccountId}/{amount}")
  fun sendMoney(
    @PathVariable("sourceAccountId") sourceAccountId: Long,
    @PathVariable("targetAccountId") targetAccountId: Long,
    @PathVariable("amount") amount: Long
  ) {
    val command = SendMoneyCommand(
      sourceAccountId = AccountId(sourceAccountId),
      targetAccountId = AccountId(targetAccountId),
      money = Money.of(amount)
    )

    sendMoneyUseCase.sendMoney(command)
  }
}
```

---

만들면서 배우는 클린 아키텍처 <톰 홈버그>
