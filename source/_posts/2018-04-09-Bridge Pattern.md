---
layout: post
title:  "Bridge Pattern"
date:   2018-04-09
categories: Design Pattern
---

- Intent

  - decouple an abstraction from its implementation so that the two can vary independently.
  - 추상을 구현으로부터 분리하여, 
  - 독립적으로 변하게 함
  - 여기에서 구현의 의미
    - 추상을 구현한 concrete class X
    - 궂은 일을 하는 로직 / 코드
  - 구현 뿐만 아니라 추상화된 부분까지 변경시켜야 하는 경우에는 bridge pattern을 사용

- Class Diagram

  ![](/image/myBridge.png)

  ![](/image/myBridgee.png)


- Code

![](/image/b01.png)

![](/image/b02.png)

![](/image/b03.png)

![](/image/b04.png)

![](/image/b05.png)

![](/image/b06.png)

- Reference
  - <https://www.tutorialspoint.com/design_pattern/bridge_pattern.htm>





