---
layout: post
title: Abstract Server, Adapter Pattern
date: 2023-03-18
categories: Agile
---

추상 서버 패턴과 어뎁터 패턴을 정리한다.

## 탁상 스탠드 설계

탁상 스탠드 내부의 소프트웨어를 설계해보자.
단순하게 다음과 같이 설계할 수 있다.

![](/image/clean-software-abstractserver-adapter-pattern-table-stand-simple-archi.png)

이 설계의 문제는,

1. DIP 위반: Light 로 향하는 의존 관계가 구체 클래스에 대한 의존 관계이다.
2. Switch 는 Light 이외의 객체를 제어할 수 있도록 확장이 힘들다.

## 추상 서버 패턴

Switch 와 Light 사이에 인터페이스를 도입해서, Switch 가 무엇이든 제어할 수 있게 만들 수 있다.

![](/image/clean-software-abstractserver-adapter-pattern-table-stand-abstractserver.png)

여기서 주목할 것은, 인터페이스의 이름이 ILight 가 아니라 Switchable 이다.
즉, 인터페이스의 클라이언트인 Switch 를 위한 쪽으로 인터페이스 이름을 지었다.
인터페이스는 파생 클래스나 파생 인터페이스에 속하는 것이 아니라, 클라이언트에 속하기 때문이다.

## 어뎁터 패턴

위 설계에도 문제가 있다.

1. 변화의 이유가 다를 수 있는 Light 와 Switchable 을 묶어놓아서, SRP 를 위반할 수 있다.
2. Switch 가 제어할 수 있는 클래스가 Switchable 로부터 파생 받을 수 없을 수 있다.

이 문제를 해결하기 위해, 어뎁터가 Switchable 을 상속하고 실제 일은 Light 에게 위임할 수 있다.

![](/image/clean-software-abstractserver-adapter-pattern-table-stand-dapter.png)

---

클린 소프트웨어 <로버트 C.마틴>
