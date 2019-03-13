---
layout: post
title:  "Refactoring"
date:   2018-12-07
categories: Software Engineering
---

##### code refactoring 이란?

외부동작을 바꾸지 않으면서 내부 구조를 개선하는 방법입니다.

##### 왜 해야하죠?

1. 소프트웨어의 디자인을 개선시킵니다.
2. 소프트웨어를 더 이해하기 쉽게 만든다.
3. 버그를 찾도록 도와줍니다.

##### 언제 해야하죠?

1. 삼진규칙, 즉 스트라이크 세 개면 리팩토링을 합니다.

   어떤 것을 처음 할 때는, 그냥 합니다.두번째로 비슷한 어떤 것을 하게 되면, 그냥 중복되도록 합니다. 세 번째로 비슷한 것을 하게 되면, 그때 리팩토링을 합니다.

2. 기능을 추가할 때 리팩토링을 합니다.

3. 버그를 수정해야 할 때 리팩토링을 합니다.

4. Code Review 를 할 때 리팩토링을 합니다.

##### 어떤 것을 해야하죠?

1. Shotgun Surgery (기능의 산재)

   수정할 때마다 여러 클래스에서 수많은 부분을 고쳐야 한다면, 이 부분을 의심합니다. 이 경우, 메서드 또는 필드를 하나의 클래스로 추출합니다. 기존의 클래스 중 적절한 것이 없다면 새로운 클래스를 만들어 추출합니다.

2. Lazy Class (직무유기 클래스)

   리팩토링으로 인해 기능이 축소된 클래스, 또는 수정할 계획으로 작성했으나 수정을 실시하지 않아 쓸모없어진 클래스입니다. 

3. Middle Man (과잉 중개 메서드)

   클래스가 간단한 위임을 너무 많이 하고 있는 경우입니다. 많은 메소드가 이와 같이 하고 있다면 Person 클래스에 간단한 위임 메소드가 너무 많이 생기게 됩니다.

   ```java
   class Person {
     Department _department;
     public Person getManager() {
       return _department.getManager();
     }
   }
   
   class Department {
     private Person _manager;
     public Department (Person manager) {
       _manager = manager;
     }
   }
   
   //client
   manager = john.getManager();
   ```

   ```java
   class Person {
     public Department getDepartment() {
       return _department;
     }
   }
   
   manager = john.getDepartment().getManager();
   ```

4. Refused Bequest (방치된 상속물)

   하위 클래스는 부모 클래스의 메서드와  데이터를 상속받습니다. 이 때, 하위 클래스에서 필요한 것을 제외한 나머지는 방치됩니다. 이는 잘못된 계층 구조 때문에 발생합니다.

##### reference

https://wikidocs.net/593