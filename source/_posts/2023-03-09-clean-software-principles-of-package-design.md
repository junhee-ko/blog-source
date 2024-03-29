---
layout: post
title: Principles of Package Design
date: 2023-03-09
categories: Agile
---

큰 애플리케이션을 조직화하기 위해서는, 클래스보다 더 큰 package 가 필요하다.
패키지의 응딥도에 대한 원칙과 결합도에 대한 원칙을 정리한다.

1. 패키지 응집도에 대한 원칙: 클래스를 패키지에 할당하는 것을 도와준다.
2. 패키지 결합도에 대한 원칙: 패키지 간 관계를 결정하는 것을 도와준다.

## 패키지 응집도에 대한 원칙

패키지 응집도에 대한 원칙은 아래 세 가지가 있다.

1. 재사용 릴리즈 등가 원칙 (REP: Reuse-Release Equivalent Principle)
2. 공통 재사용 원칙 (CRP: Common-Reuse Principle)
3. 공통 폐쇄 원칙 (CCP: Common-Closure Principle)

### 재사용 릴리즈 등가 원칙

`재사용의 단위는 릴리즈 단위다.`

재사용하는 모든 것은 반드시 릴리즈 된 다음에 추적 가능해야한다.
사용자들에게 필요한 통보, 안정성, 지원에 대한 보장을 제공하는 추적 시스템이 먼저 있어야 재사용성이라는 말을 할 수 있다.
재사용성은 패키지에 기반을 두어야하기 때문에, 재사용 가능한 패키지는 재사용 가능한 클래스를 포함해야한다.

또한, 재사용성 뿐만 아니라 재사용자가 누구인지도 고려해야한다.
컨테이너 클래스 라이브러리는 사용하고 싶지만, 금융 관련 프레임워크에는 관심 없는 사람이 있을 수 있다.
패지지 안의 모든 클래스는 동일한 재사용자를 대상으로 해야한다.

### 공통 재사용 원칙

`패키지 안의 클래스들은 같이 재사용되어야한다. 어떤 패키지의 클래스 하나를 재사용하면 나머지 클래스들도 모두 재사용한다.`

이 원칙에 따르면, 자주 함께 재사용되는 클래스들은 동일한 패키지에 속해있어야한다.
예를 들어, 컨테이너 클래스와 iterator 는 서로 단단히 결합되어 있기 때문에 함께 재사용된다.
그래서 이 클래스들은 동일한 패키지에 있어야한다.

### 공통 폐쇄 원칙

`패키지에 어떤 변화가 영향을 미치면, 그 변화는 그 패키지의 모든 클래스에 영향을 미쳐야하고 다른 패키지에는 영향을 미치지 않아야한다.`

대상이 패키지인 Single Responsibility Principle 이다.
클래스를 변경할 이유가 여러 가지면 안되는 SRP 처럼, 이 원칙은 패키지를 변경할 이유도 여러 가지면 안된다는 것이다.

동일한 이유로 변할 것 같은 클래스들이 같은 패키지에 있으면 릴리즈, 재검증, 재배포하는 일과 관련된 작업을 최소로 할 수 있다.

## 패키지 결합에 대한 원칙

패키지 결합도에 대한 원칙은 아래 세 가지가 있다.

1. 의존 관계 비순환 원칙 (ADP: Acyclic-Dependencies Principle)
2. 안정된 의존 관계 원칙 (SDP: Stable-Dependencies Principle)
3. 안정된 추상화 원칙 (SAP: Stable-Abstractions Principle)

### 의존 관계 비순환 원칙

`패키지 의존성 그래프에서 순환을 허용하지 말라.`

이 원칙을 지키면, 어떤 패키지에서 시작하더라도 의존 관계를 따러가서 다시 같은 패키지에 도달할 수 없다.
즉, 비순환 방향 그래프다.

어떤 패키지가 변경되었다고 해도, 이 패키지를 사용하는 다른 팀에 영향을 끼치지 않는다.
팀마다 사용하는 패키지의 새 릴리즈를 채택할지 말지 스스로 결정할 수 있다.

패지지 순환이 있는 다음 예를 보자.

![](/image/clean-software-cyclic-dependeciese-principle-ex.png)

패키지 순환을 끊는 방법으로 아래 두 가지 방법이 있다.

1. 의존 관계 역전 원칙을 적용한다.
   ![](/image/clean-software-acyclic-dependeciese-principle-dip.png)
2. 두 패키지가 모두 의존하는 클래스들을 모아 둔 새로운 패키지를 만든다.
   ![](/image/clean-software-acyclic-dependeciese-principle-new-pacakge.png)

### 안정된 의존 관계 원칙

`의존은 안정적인 쪽으로 향해야한다.`

설계를 계속 유지보수 하려면, 변동성이 필요하다.
안정된 의존 관계 원칙을 지키면 특정 변화에 쉽게 반응 가능한 패키지를 만들 수 있다.
이 패키지들은 쉽게 변하도록 설계된 패키지들이여서, 변경되리라 예상이 되는 패키지들이다.

쉽게 바뀔 것이라고 예상되는 패키지들은, 바뀌기 어려운 패키지들의 의존 대상이 되면 안된다.

### 안정된 추상화 원칙

`패키지는 자신이 안정적인 만큼, 추성적이어야한다.`

어떤 패키지가 안정적이라면, 확장할 수 있도록 추상 클래스들로 구성되어야한다.
확장이 가능한 안정적인 패키지는 유연하여 설계를 지나치게 제약하지 않는다.

---

클린 소프트웨어 <로버트 C.마틴>
