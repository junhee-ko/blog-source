---
layout: post
title:  "abstract class, interface"
date:   2018-12-05
categories: Java
---

##### 추상 클래스와 인터페이스가 뭔가요 ?

추상클래스는 추상 메소드를 하나라도 가지는 클래스입니다.
인터페이스는 바디가 없는 메소드의 그룹입니다.

##### 추상 클래스와 인터페이스를 사용하는 이유가 뭔가요 ? 

추상클래스는 추상클래스를 상속받아 그 기능을 이용하고 확장키는 목적으로 사용됩니다.
인터페이스는 메소드의 구현을 강제하기 위한 목적으로 사용됩니다.

##### 다중 상속이 되나요?

추상클래스 다중 상속은 불가능하고, 인터페이스 다중 상속은 가능합니다.

##### 추상클래스 예시는 ?

```java
abstract class Shape { //추상클래스 (슈퍼클래스)
    int x, y;
    public void move(int x, int y){
        this.x = x;
        this.y = y;
    }
    public abstract void draw(); //추상메서드
}

class Rectangle extends Shape{ //서브클래스
    int w, h;
    public void draw(){ //메서드 재정의
       System.out.println("Rectangle draw");
    }
}
```

##### 인터페이스 예시는?

```java
interface IWorker { //인터페이스
    public void work();
   	public void eat();
}
   
class Worker implements IWorker{
    public void work() {
        // ...working
   	}
   	public void eat() {
   		// ...eating in launch break
   	}
}
```

