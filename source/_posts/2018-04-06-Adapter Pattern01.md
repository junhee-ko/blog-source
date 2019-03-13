---
layout: post
title:  "Adapter Pattern01"
date:   2018-04-06
categories: Design Pattern
---

- Intent
  - Convert the interface of a class into another interface clients expect.
  - Adapter lets classes work together, that could not otherwise because of incompatible interfaces.
  - 레거시 시스템을 원하는 인터페이스로 사용가능케 함.
  - Wrapper로도 불림


- Class Diagram
  - Clinet : Target Interface 만 볼 수 있다
  - Adapter : Target Interface를 구현, Adaptee로 구성
  - Adaptee : 모든 요청은 Adaptee에게 위임

![](/image/duck.png)

- Code

### Target Interface

![](/image/11111.png)

### Target

![](/image/22222.png)

### Adaptee Interface

![](/image/33333.png)

### Adaptee 

![](/image/44444.png)

### Adapter 

![](/image/55555.png)

### Client

![](/image/66666.png)

### Reference

- <http://www.oodesign.com/adapter-pattern.html>
- Head First Design Patterns