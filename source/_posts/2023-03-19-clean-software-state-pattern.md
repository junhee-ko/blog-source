---
layout: post
title: State Pattern
date: 2023-03-19
categories: Agile
---

스테이트 패턴을 정리한다.

## FSM (Finite State Machine)

지하철 개찰구가 동작하는 방식에서 간단한 유한 상태 기계 (FSM: Finite State Machine) 의 예를 보자.
아래 다이어그램을 STD (State Transition Diagram) 이라고 한다.

![](/image/clean-software-state-pattern-std.png)

- 기계가 Locked 상태에서 coin 이벤트를 받으면, Unlocked 상태로 전이가 되고 unlock 행동을 호출
- 기계가 Unlocked 상태에서 pass 이벤트를 받으면, Locked 상태로 전이가 되고 lock 행동을 호출

같은 의미로 STT (State Transition Table) 로도 표현할 수 있다.

```text
Locked      coin    Unlocked    unlock
Unlocked    pass    Lokced      lock
```

이 도구의 장점은, 설계자가 이상한 조건 또는 그 조건을 다룰 행위의 정의되지 않은 조건을 찾기 쉽다.
에를 들어,

- Unlocked 상태에서 coin 이벤트를 다루는 전이가 없고,
- Locked 상태에서 pass 이벤트를 다루는 전이도 없다.

추가해보자.

![](/image/clean-software-state-pattern-std-added-self-call.png)

## 구현

FSM 을 구현하는 전략의 가장 단순하고 직접적인 전략은 중첩된 switch-case 문으 사용하는 것이다.

```java
public class Turnstile {
    // 상태
    public static final int LOCKED  = 0;
    public static final int UNLOCKED  = 1;

    // 이벤트
    public static final int COIN = 0;
    public static final int PASS = 1;

    /* 전용 */
    int state = LOCKED;

    private TurnstileController controller;

    public Turnstile(TurnstileController action) {
        controller = action;
    }

    public void event(int event) {
        switch (state) {
          case LOCKED :
              switch (event) {
                case COIN :
                    state = UNLOCKED;
                    controller.unlock();
                    break;
                case PASS :
                    controller.alarm();
                    break;
              }
              break;

          // ...
        }
    }

}
```

중첩된 switch-case 문은 구현이 명쾌하고 효율적이다. 모든 상태와 코드를 한 페이지 안에서 볼 수 있다.
하지만, FSM 의 규모가 커지면 case 문 들이 계속 이어지는 것을 알아보기 힘들어진다.

## 스테이트 패턴

스테이트 패턴은 FSM 을 구현하기 위한 또 다른 기법이다.

![](/image/clean-software-state-pattern-turnstile-example.png)

Turnstile 의 이벤트 메서드 두 개 중 하나가 호출되면, 이 이벤트를 TurnstileState 객체에게 위임한다.

```java
interface TurnstileState {
    void coin(Turnstile t)
    void pass(Turnstile t)
}

public class LockedTurnstileState implements TurnstileState {

    public void coin(Turnstile t) {
        t.setUnlocked();
        t.unlock();
    }

    public void pass(Turnstile t) {
        t.alarm();
    }
}

public class UnlockedTurnstileState implements TurnstileState {

    public void coin(Turnstile t) {
      t.thankyou();
    }

    public void pass(Turnstile t) {
      t.setLocked();
      t.lock();
    }
}

public class Turnstile {

    private static TurnstileState lockedState = new LockedTurnstileState();
    private static TurnstileState unlockedState = new UnlockedTurnstileState();

    private TurnstileController controller;
    private TurnstileState state = lockedState;

    public Turnstile(TurnstileController action) {
        controller = action;
    }

    public void coin() {
        state.coin(this);
    }

    public void pass() {
        state.pass(this);
    }

    // ...
}
```

## 장/단점

스테이트 패턴은 논리와 행동을 분명히 분리한다.
행동은 Context 클래스에서 구현되고, 논리는 State 클래스의 파생형들 사이에서 분산된다.
그래서 다른 쪽에 영향을 주지 않고도 한 쪽을 변경하는 것위 쉬워진다.
예를 들어, 종류가 다른 State 클래스의 파생형을 사용해서 Context 클래스의 행동을 다른 상태 논리에 재사용할 수 있다.

하지만, State 의 파생형을 작성하는 작업에 대한 비용이 크다.
또한, 상태 기계의 논리를 볼 수 있는 장소가 없어서 코드를 유지보수 하기 힘들다.

---

클린 소프트웨어 <로버트 C.마틴>



