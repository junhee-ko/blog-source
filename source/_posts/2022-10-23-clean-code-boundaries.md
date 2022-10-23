---
layout: post
title: Boundaries
date: 2022-10-23
categories: Clean Code
---

외부 코드를 우리 코드에 깔끔하게 통합하는 방법을 정리한다.

## 외부 코드 사용

java.util.Map 은 아래처럼, 다양한 인터페이스를 제공한다.

https://docs.oracle.com/javase/8/docs/api/java/util/Map.html

Map 이 제공하는 기능성과 유연성은 유용하지만, 위험도 크다.
Map 을 만들어서 여기저기 넘긴다고 하자.
넘기는 쪽에서는 Map 내용을 삭제하지 않는다고 생각할 수 있다.
하지만, Map 사용자라면 clear() 로 내용을 지울 권한이 있다.
또한, Map 은 객체 유형을 제한하지 않기 때문에, 누구나 어떤 객체 유형도 추가할 수 있다.

이런 위험을 개선하는 코드를 보자.

```java
public class Sensors {
    private Map sensors = new HashMap();

    public Sensor getById(String id) {
        return (Sensor) sensors.get(id);
    }
}
```

경계 인터페이스인 Map 을 Sensors 안으로 숨겼다.
그래서, Map 인터페이스가 변해도 프로그램에 영향을 미치지 않는다.
그리고, Sensors 클래스는 필요한 인터페이스만 제공해서 코드를 이해하기 쉽고 오용하기 어렵다.
Sensors 클래스는 설계 규칙과 비즈니스 규칙을 따르도록 강제할 수 있다.

## 경계 살피고 익히기

외부에서 가져온 패키지를 사용하고 싶으면 학습 테스트부터 시작하자.
우리 코드를 작성해 외부 코드를 호출하는 대신, 먼저 간단한 테스트 케이스를 작성해 외부 코드를 익히자.
학습 테스트는, API 를 사용하려는 목적에 초점을 맞춘다.

## 아직 존재하지 않는 코드 사용하기

경계와 관련해 또 다른 유형은, 아는 코드와 모르는 코드를 분리하는 것이다.
Transmitter 라는 시스템에 대해 지식이 없다고 하고, 인터페이스가 아직 정의되어 있지 않다고 하자.

우리가 바라는 자체적인 인터페이스를 먼저 정의하자.
우리가 인터페이스를 전적으로 통제할 수 있고, 코드 가독성도 높아지고 코드 의도도 분명해진다.


Transmitter API 가 정의되면, TransmitterAdapter 를 구헌해 간극을 매꿀수 있다.
FakeTransmitter 클래스로, 테스트도 쉬워진다.

![](/image/clean-code-boundaries-transmitter-exmaple.png)

---

클린코드 <로버트 C.마틴>
