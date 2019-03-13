---
layout: post
title:  "Facade Pattern"
date:   2018-04-10
categories: Design Pattern
---

- Intent

  - hides the complexities of the system 
  - provides an interface to the client using which the client can access the system
  - 인터페이스를 단순화 & 클라이언트와, 구성 요소로 이루어진 서브 시스템을 분리

- Implementation

  ![](/image/facadeImple.png)


- Code

#### Create an interface

![](/image/fa01.png)

#### Create concrete classes implementing the same interface

![](/image/fa02.png)

![](/image/fa03.png)

![](/image/fa04.png)

#### Create a facade class

- 사용하고자 하는 서브 시스템의 모든 구성요들이 인스턴스 변수 형태로 저장

![](/image/fa05.png)

#### Use the facade to draw various types of shapes.

![](/image/fa06.png)

#### Reference

<https://www.tutorialspoint.com/design_pattern/facade_pattern.htm>

