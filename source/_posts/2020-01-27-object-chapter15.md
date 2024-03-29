---
layout: post
title:  "[오브젝트] 15장_디자인 패턴과 프레임워크"
date:   2020-01-27
categories: OOP
---

1. 디자인 패턴
   특정한 변경을 일관성있게 다룰 수 있는 협력 텝플릿 제공한다.
   설계를 재사용하는 것이 목적이다.

2. 프레임워크
   특정한 변경을 일관성 있게 다룰 수 있는 확장 가능한 코드 템플릿을 제공한다.
   설계와 코드를 함께 재사용하기 위한 것이 목적이다.

## 디자인 패턴과 설계 재사용

### 소프트웨어 패턴

패턴의 특징은,

1. 반복적으로 발생하는 문제와 해법의 쌍으로 정의된다.
2. 이미 알려진 문제와 이에 대한 해법을 문서로 정리할 수 있고, 다른 사람과 의사소통 가능하다.
3. 추상적인 원칙과 실제 코드 작성 사이의 간극을 메워준다. 실질적인 코드 작성을 돕는다.
4. 패턴은 실무에서 탄생했다.

마틴 파울러에 의하면, 패턴은 하나의 실무 컨텍스트에서 유용하게 사용해왔고 다른 실무 컨텍스트에도 유용할 것이라고 예상되는 아이디어다.
프로젝트 조직을 구성하는 방법, 프로젝트 일정을 추정하는 방법 등 반복적인 규칙을 발견할 수 있는 모든 영역이 패턴의 대상이다.

### 패턴 분류

1. 디자인 패턴
   일반적인 설계 문제를 해결한다.
   협력하는 컴포넌트들 사이에서 반복적으로 발생하는 구조를 서술한다.

3. 아키텍쳐 패턴
   디자인 패턴의 상위에 있다.
   소프트웨어의 전체적인 구조를 결정한다.

4. 이디엄
   디자인 패턴의 하위에 있다.
   특정 프로그래밍 언어에만 국한된 하위 레벨 패턴이다.
   예를 들어, C++ 의 COUNT POINT 이디엄은 자바에서는 유용하지 않다.

5. 분석 패턴
   도메인 내의 개념적인 문제를 해결한다.

### 패턴과 책임-주도 설계

객체지향 설계에서 중요한 일을 다시 정리해보자.
바로, 올바른 책임을 올바른 객체에게 할당하고 객체 간의 유연한 협력 관계를 구축하는 것이다.

패턴은 공통으로 사용할 수 있는 역할, 책임, 협력의 템플릿이다. 예를 들면,

1. STRATEGY 패턴
   다양한 알고리즘을 동적으로 교체할 수 있는 역할과 책임을 제공한다.

2. BRIDGE 패턴
   추상화의 조합으로 인한 클래스의 폭발적 증가 문제를 해결하기 위해 역할과 책임을 추상화와 구현의 두 개의 커다란 집합으로 분해해서 설계를 확장가능하게 한다.

3. OBSERVER 패턴
   유연한 통지 매커니즘을 구축하기 위해 객체 간의 결합도를 낮출 수 있는 역할과 책임을 제공한다.

패턴의 구성 요소는 클래스가 아니라, '역할' 이다.
예를 들어, 클라이언트가 개별 객체와 복합 객체를 동일하게 취급할 수 있는 COMPOSITE 패턴을 보자.

![](/image/object_c15_01.png)

component, composite, leaf 는 패턴의 구성 요소이다.
이것들은, 클래스가 아니라 협력에 참여하는 객체들의 역할이다.
component 는 역할이기 때문에 component 가 제공하는 오퍼레이션을 구현하는 어떤 객체라도 component 의 역할을 수행할 수 있다.

![](/image/object_c15_02.png)

중복 할인 설계의 기본 구조는 COMPOSITE 패턴을 따른다.

### 캡슐화와 디자인 패턴

각 디자인 패턴은 특정한 변경을 캡슐화하기 위해 독자적인 방법을 정의하고 있다.

![](/image/object_c15_03.png)

위 그림은 STRATEGY 패턴을 적용한 영화 예매 시스템 설계이다.
변경을 캡슐화하기 위해 합성을 이용한다.
Movie 와 DiscountPolicy 사이의 결합도를 낮춰 런타임에 알고리즘을 변경할 수 있다.

![](/image/object_c15_04.png)

위 그림은 TEMPLATE METHOD 패턴을 적용한 설계이다.
부모 클래스의 calculateFee 메서드 안에서 추상 메서드를 호출하고 자식 클래스들이 이 메서드를 오버라이딩해서 변하는 부분을 구현한다.

TEMPLATE METHOD 패턴은, 변경을 캡슐화하기 위해 상속을 이용한다.
TEMPLATE METHOD 패턴은 합성 보다는 결합도가 높은 상속을 사용했기 때문에, 런타임에 객체의 알고리즘을 변경하는 것이 불가능하다.
하지만, 알고리즘 교체와 같은 요구사항이 없었으면 복잡도를 낮출 수 있다는 장점이 있다.


![](/image/object_c15_05.png)

위 그림은 DECORATOR 패턴을 적용한 설계이다.
객체의 행동을 동적으로 추가할 수 있는 패턴으로서, 객체의 행동을 결합하기 위해 객체 합성을 사용한다.

디자인 패턴에서 중요한 것은, 디자인 패턴의 구현 방법이나 구조가 아니다.
`어떤 변경을 캡슐화하는지 이해하고 변경을 캡슐화하기 위해 어떤 방법을 사용하는지` 이해해야한다.

### 패턴은 출발점이다

패턴 입문자는, 패턴을 적용하는 컨텍스트의 적절성을 무시하고 패턴의 구조에만 초점을 맞춘다.
패턴을 적용할 때는 설계를 단순하고 명확하게 만들 수 있는 방법이 없는지를 고민해야한다.
그리고, 코드를 공유하는 모든 사람들이 적용된 패턴을 알고 있어야한다.

## 프레임워크와 코드 재사용

### 코드 재사용 대 설계 재사용

프레임워크란,

1. 추상 클래스나 인터페이스를 정의하고 인스턴스 사이의 상호작용을 통해 시스템 전체 혹은 일부를 구현해 놓은 재사용 가능한 설계이다.
2. 애플리케이션 개발자가 현재의 요구사 항에 맞게 커스터마이징할 수 있는 애플리케이션의 골격이다.

### 상위 정책과 하위 정책으로 패키지 분리하기

프레임워크의 핵심은, 추상 클래스나 인터페이스와 같은 추상화이다.
아래 그림에서, 추상화는 짙은 색으로 표시되어 있다.

![](/image/object_c15_06.png)

의존성 역전 원칙으로, 구체 클래스들은 상위 클래스에 의존하지만 그 반대는 아니다.
상위 정책은 상대적으로 변경에 안정적이지만 세부 사항은 자주 변경된다.
만약, 상위 정책이 자주 변하는 세부 사항에 의존하면 변경에 대한 파급 효과로 상위 정책이 불안정해진다.
핵심은, 상위 정책이 세부 사항보다 더 다양한 상황에서 재사용할 수 있어야한다는 것이다.

프레임워크는 여러 애플리케이션에 걸쳐 재사용가능해야하기 때문에, 변하는 것과 변하지 않는 것을 서로 다른 주기로 배포할 수 있도록, 배포 단위를 분리해야한다.
이를 위해, 변하는 부분과 변하지 않는 부분을 별도의 패키지로 분리할 수 있다.

![](/image/object_c15_07.png)

중요한 것은, 패키지 사이의 의존성 방향이다.
세부사항을 구현한 패키지는 항상 상위 정책을 구현한 패키지에 의존해야한다.
상위 정책을 구현하고 있는 패키지를 다른 애플리케이션에 재사용할 수 있다.
즉, 재사용 가능한 요금 계산 로직을 구현한 프레임워크가 만들어진 것이다.

### 제어 역전 원리

의존성 역전 원리는, 프레임워크의 가장 기본적인 설계 메커니즘이다.
의존성 역적은, 의존성 방향 뿐만 아니라 제어 흐름의 주체 역시 역전시킨다.

전통적인 구조에서는 상위 정책이 구체적인 세부 사항에 의존한다.
상위 정책의 코드가 하부의 구체적인 코드를 호출한다.
즉, 애플리케이션의 코드가 재사용 가능한 라이브러리나 툴킷의 코드를 호출한다.

의존성을 역전 시킨 객치지향 구조에서는 그 반대이다.
프레임워크가 에플리케이션에 속하는 서브 클래스들의 메서드를 호출한다.
즉, 개별 애플리케이션에서 프레임워크로 제어 흐름의 주체가 이동한다.
이를 제어 역전 원리라고 한다.

![](/image/object_c15_08.png)

위 그림에서, 특정한 기본 정책을 구현하는 개발자는 FeeCondition 을 대체할 서브 타입만 개발하면 프레임워크에 정의된 플로우에 따라 요금이 계산된다.
프레임워크에서는 일반적인 해결책만 제공하고, 애플리케이션에 따라 달라질 수 있는 동작은 비워둔다.

---

오브젝트 <조영호>
