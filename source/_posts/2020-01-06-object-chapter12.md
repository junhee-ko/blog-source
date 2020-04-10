---
layout: post
title:  "[오브젝트] 12장_다형성"
date:   2020-01-06
categories: OOP
---

상속의 진정한 목적은 코드 재사용이 아니라 `다형성을 위한 서브타입 계층을 구축` 하는 것이다.

상속의 관점에서 다형성이 구현되는 기술적인 메커니즘을 정리한다.

## 01 다형성

다형성(Polymorphism) 은 다음 둘의 합성어이다. 즉, 많은 형태를 가질 수 있는 능력이다.

- ploy : 많은
- morph : 형태

다형성은 다음과 같이 분류될 수 있다. 이번장은 포함 다형성에 대해 다룬다.

- 유니버셜 다형성 

  - 매개변수 다형성 

    클래스의 인스턴스 변수나 메서드의 매개변수 타입을 임의의 타입으로 선언한 후 사용하는 시점에 구체적인 타입으로 지정하는 방식.

     ex) List 인터페이스는 컬렉션에 보관할 요소의 타입을 임의의 타입 T로 지정하고 있으며 실제 인스턴스를 생성하는 시점에 T 를 구체적인 타입으로 지정

  - 포함 다형성 (서브타입 다형성)

    메세지가 동일해도 수신한 객체의 타입에 따라 실제로 수행되는 행동이 달라지는 능력.

- 임시 다형성

  - 오버로딩 다형성

    하나의 클래스 안에 동일한 이름의 메서드가 존재하는 경우.

  - 강제 다형성

    자동적인 타입 변환이나 사용자가 직접 구현한 타입 변환을 이용해 동일한 연산자를 다양한 타입에 사용할수 있는 방식.

    ex) 이항 연산자인 '+' 는 피연산자가 하나는 정수형이고 다른 하나는 문자열인 경우, 정수형 피연산자는 문자열 타입으로 강제 형변환

## 02 상속의 양면성

1. 데이터 관점의 상속

   부모 클래스에서 정의한 모든 데이터를 자식 클래스의 인스턴스에 자동으로 포함한다.

2. 행동 관점의 상속

   데이터뿐만 아니라 부모 클래스에서 정의한 일부 메서드 역시 자동으로 자식 클래스에 포함한다.

   외부의 객체가 부모 클래스의 인스턴스에 전송할 수 있는 모든 메세지는 자식 클래스의 인스턴스에도 전송할 수 있다.

##### 상속을 사용한 강의 평가

code : 394 p

1. 메서드 오버라이딩

   자식 클래스 안에 상속 받은 메서드와 `동일한 시그니처의 메서드를 재정의`해서 부모 클래스의 구현을 새로운 구현으로 대체하는 것이다.

2. 메서드 오버로딩

   부모 클래스에서 정의한 `메서드와 이름은 동일하지만 시그니처는 다른` 메서드를 자식 클래스에 추가하는 것이다.

## 03 업캐스팅과 동적 바인딩

##### 같은 메세지, 다른 메서드

```java
Professor professor01 = new Professor("다익스트라", new Lecture(...))
Professor professor02 = new Professor("다익스트라", new GradeLecture(...))
  
professor01.compileStatistics();
professor02.compileStatistics();
```

```java
public String compileStatistics() {
	...
	lecture.evalulate();
  ...
}
```

동일한 객체 참조인 lecture 에 대해 동일한 evaluate 메세지를 전송하는 동일한 코드 안에서, 서로 다른 클래스 안에 구현된 메서드를 실행할 수 있다.

이처럼, 코드 안에서 선언된 참조 타입과 무관하게 `실제로 메세지를 수신한 객체의 타입에 따라 실행되는 메서드가 달라질 수 있는 것`은 다음 두 메커니즘이 작용하기 때문이다.

1. 업캐스팅

   부모 클래스 타입으로 선언된 변수에 자식 클래스의 인스턴스를 할당하는 것이 가능

2. 동적 바인딩

   메세지를 처리할 적절한 메서드를 컴파일 시점이 아니라 실행시점에 결정

##### 업캐스팅

![](/image/object_polymorphism_upcasting.png)

업캐스팅의 대표적인 두 가지이다.

1. 대입문

   명시적으로 타입을 변환하지 않고도 부모 클래스의 타입의 참조변수에 자식 클래스의 인스턴스를 대입할 수 있다.

   ```java
   Lecture lecture = new GradeLecture(...);
   ```

2. 메서드 파라미터

   부모 클래스의 타입으로 선언된 파라미터에 자식 클래스의 인스턴스를 전달할 수 잇다.

   ```java
   public class Professor {
       public Professor(String name, Lecture lecture){
           ...
       }
   }
   
   Professor professor = new Professor("다익스트라", new GradeLecture(...));
   ```

다운 캐스팅은, 부모 클래스의 인스턴스를 자식 클래스 타입으로 변환하기 위해 명시적인 타입 캐스팅이 필요하다.

```java
Lecture lecture = new GradeLecture(...);
GradeLecture gradeLecture = (GradeLecture) lecture;
```

##### 동적 바인딩

1. 정적 바인딩

   컴파일 타임에 호출할 함수를 결정하는 방식

2. 동적 바인딩

   실행될 메서드를 런타임에 결정하는 방식. 

   실행 시점에 어떤 클래스의 인스턴스를 생성해서 전달하는지 알아야만 실제로 실행될 메서드를 알 수 있다.

## 04 동적 메서드 탐색과 다형성

객체지향 시스템은 다음 규칙에 따라 실행할 메서드를 선택한다.

1. 메세지를 수신한 객체는 자신을 생성한 클래스에 적합한 메서드가 존재하는지 검사한다.

   존재하면 메서드를 실행하고 탐색을 종료한다.

2. 존재하지 않으면, 부모 클래스에서 메서드 탐색을 계속한다.

   적합한 메서드를 찾을 때 까지 상속 계층을 따라 올라가며 계속된다.

3. 상속 계층의 최상위 클르스에 올라갔지만 메서드를 발견하지 못하면 예외를 발생시키며 탐색을 종료한다.

여기서 중요한 것이 sefl 참조 변수이다.

객체가 메세지를 수신하면, 컴파일러는 self 참조라는 임시 변수를 자동으로 생성해 메세지를 수신한 객체를 가리키도록 한다.

![](/image/object_polymorphism_self.png)

위 그림에서, 

1. GradeLecture 클래스에서 적절한 메서드를 찾지 못했다면 
2. parent 참조를 따라 부모 클래스인 Lecture 클래스로 이동한후 탐색을 계속한다. 
3. 상속 계층을 따라 최상위 클래스인 Object 클래스에 이를 때 까지 탐색을 계속한다.  
4. 최상위 클래스에서도 메서드를 찾지 못하면 에러를 발생시킨다.

동적 메서드 탐색은 두 원리로 구성된다.

1. 자동적인 메세지 위임

   자식 클래스는 이해할 수 없는 메세지를 전송 받으면 상속 계층을 따라 부모 클래스에 처리를 위임한다.

2. 동적인 문맥

   메세지를 수신했을 때, 실제로 어떤 메서드가 실행될지 결정하는 것은 컴파일 시점이 아니라 실행시점에 이뤄진다.

##### 자동적인 메세지 위임

1. 메서드 오버라이딩 : 자식 클래스의 메서드가 부모 클래스의 메서드를 감추게 된다.

   ![](/image/object_polymorphism_overriding.png)

   ```java
   Lecture lecture = new Lecture(...);
   lecture.evaluate();
   ```

   위 그림과 위 코드에서, 메서드 탐색은 self 참조가 가리키는 객체의 클래스인 Lecuture 에서 시작한다.

   Lecture 클래스 안에 evaluate 메서드가 존재하기 때문에, 메서드 실행한 후 탐색은 종료한다.

   ![](/image/object_polymorphism_grade_lecture.png)

   ```java
   Lecture lecture = new GradeLecture(...);
   lecture.evaluate();
   ```

   위 그림과 위 코드에서, Lecture 에 정의된 메서드가 아닌 실제 객체를 생성항 클래스인 GradeLecture 에 정의된 메서드가 실행된다. 

   self 참조가 가리키는 객체의 클래스인 GradeLecture 에서 탐색을 시작하고 GradeLecture 클래스 안에  evaluate 메서드가 구현되어 있기 때문이다.

2. 메서드 오버로딩 :  자식 클래스의 메서드와 부모 클래스의 메서드가 공존한다.

   ![](/image/object_polymorphism_overloading.png)

   ```java
   Lecture lecture = new GradeLecture(...);
   lecture.average();
   ```

   위 그림과 위 코드에서, GradeLecture 클래스 안에서 메세지에 응답할 수 있는 적절한 메서드를 찾지 못한다.

   그래서, 부모 클래스인 Lecture 클래스에서 메서드를 찾으려고 시도한다.

##### 동적인 문맥

메세지를 수신한 객체가 무엇이냐에 따라 메서드 탐색을 위한 문맥이 동적으로 바뀐다. 

이 동적인 문맥을 결정하는 것이 메세지를 수신한 객체를 가리키는 self 참조이다.

self 참조 가 동적 문맥을 결정한다는 것은, 종종 어떤 메서드가 실행될지 예상하기 어렵게 만든다. 대표적인 경우가 self 전송이다.

self 전송은 자식 클래스에서 부모 클래스 방향으로 진행되는 동적 메서드 탐색 경로를 `다시 self 참조가 가리키는 원래의 자식 클래스로 이동`시킨다. 

다음을 보자.

```java
public class Lecture {
    public String stats(){
        return getEvaluationMethod();
    }

    private String getEvaluationMethod() {
        return "Pass or Fail";
    }
}

public class GradeLecture extends Lecture {
    @Override
    public String getEvaluationMethod() {
        return "Grade";
    }
}
```

GradeLecture 에 stats 메시지를 전송하면, 다음 그림과 같다.

![](/image/object_polymorphism_self_send.png)

1. self 참조는 GradeLecture 인스턴스를 가리키도록 설정되고 탐색은 GradeLecture 부터 시작.
2. GradeLecture 클래스에는 stats 메세지를 처리할 메서드가 없기 때문에 부모 클래스인 Lecture 에서 메서드 탐색을 계속속 하다가, Lecture 에서 stats 메서드를 발견하고 실행
3. 실행 중에, self 참조가 가리키는 getEvaluationMethod 메세지를 전송하는 구문과 마주침
4. 메서드 탐색은 self 참조가 가리키는 객체에서 다시 시작

##### 이해할 수 없는 메세지

이해할 수 없는 메세지 처리는 두 타입 언어에 따라 다르다.

1. 정적 타입 언어

   코드를 컴파일 할 때 상속 계층 안의 클래스들이 메세지를 이해할 수 있는지 여부를 판단한다. 

   상속 계층 전체를 탐색한 후에도 메시지를 처리할 메서드를 발견하지 못하면 컴파일 에러가 발생한다. (안정적이다)

2. 동적 타입 언어

   실제로 코드 실행 전에는 메시지 처리 가능 여부를 판단 할 수 없다.

   하지만, 이해할 수 없는 메세지에 대해 예외를던 는 것 외에도 doesNotUnderstand 나 method_missing 메시제에 응답 할 수 있는 메서드를 구현할 수 있다. (유연하다)

##### self 대 super

![](/image/object_polymorphism_super.png)

super.average() 에 의해 호출되는 메서드는 부모 클래스의 메서드가 아니라, 더 상위에 위치한 조상 클래스의 메서드일 수 있다.

1. self 전송

   메세지를 수신한 객체의 클래스에 따라 메서드를 탐색할 시작 위치를 동적으로 결정

2. super 전송

   항상 메세지를 전송하는 클래스의 부모 클래스에서부터 시작

## 5. 상속 대 위임

![](/image/object_polymorphism_same_self.png)

GradeLecture 인스턴스 입장에서 self 참조는, GradeLecture 인스턴스 자신이다.

GradeLecture 인스턴스에 포함된 Lecture 입장에서 self 참조는, GradeLecture 인스턴스이다. self 참조는 항상 메세지를 수신한 객체를 가리키기 때문이다.

즉, 상속 계층을 구성하는 객체들 사이에서는 self 참조를 공유하기 때문에 개념적으로 각 인스턴스에서 self 참조를 공유하는 self 변수를 포함하는 것처럼 표현할 수 있다.

상속은 동적으로 메서드를 탐샘하기 위해 현재의 실행문맥을 가지고 있는 self 참조를 전달한다. 그리고 이 객체들 사이에서는 메시지를 전달하는 과정이 자동으로 이뤄진다. 그래서, 자동적인 메세지 위임이라고 한다.

---

오브젝트 <조영호>
