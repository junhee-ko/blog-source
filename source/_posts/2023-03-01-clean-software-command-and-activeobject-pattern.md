---
layout: post
title: Command Pattern && Active Object Pattern
date: 2023-03-01
categories: Agile
---

커멘드 패턴과 액티비 오브젝트 패턴을 정리한다.

## Command Pattern

```java
public interface Command
{
    public void do();
}
```

대부분의 클래스는 한 벌의 메서드와 변수의 집합을 결합시키는데, 커멘드 패턴은 그렇지 않다.
오히려 함수를 캡슐화해서 변수에서 해방시킨다. 즉, 함수의 역할을 클래스 수준으로 격상시킨다.

### AddEmployeeTransaction

![](/image/clean-software-command-pattern-transaction-ex.png)

직원들의 DB 를 관리하는 시스템을 작성하는 예를 보자.
command 객체는 검증되지 않은 데이터를 위한 저장소 역할을 하고, 검증 메서드를 구현하고 마지막으로 트랜잭션을 실행하는 메서드를 구현한다.

예를 들어, AddEmployeeTransaction 은 Employee 가 포함하는 데이터를 모두 가지고 있다.
validate 메서드는 모든 데이터를 살펴보고 문법적으로나 의미적으로 오류가 없는지 확인한다.
또한, 트랜잭션의 데이터가 기존의 데이터베이스 상태와 일치하는지 확인할 수도 있다.
execute 메서드는 검증된 데이터를 사용해 데이터베이스를 갱신한다.

### 물리적, 시간적 분리

이 방식의 장점은, 사용자에게 데이터를 받는 코드와 데이터 검증 및 작업 코드를 분리할 수 있다는 것이다.
검증과 실행 코드를 따로 떼어 AddEmployeeTransaction 클래스에 넣으면, 이 코드를 입력 인터페이스에서 물리적으로 분리한 것이 된다.

또한, 데이터를 받으면 검증 및 실행을 즉시 호출할 필요가 없다. 트랜잭션 객체를 우선 목록에 저장된 뒤 나중에 검증되고 실행이 될 수 있다.
ex) 낮 동안은 변경되지 않아야 하는 DB

## Active Object Pattern

커멘트 패턴의 응용 방식 중 하나이다. 다중 제어 스레드 구현을 위한 기법 중 하나이다.
ActiveObjectEngine 객체는 Command 객체의 연결 리스트를 유지한다.
이 엔진에 새로운 명령을 추가할 수 있고, run() 을 호출할 수도 있다.

```java
public interface Command {
  public void execute() throws Exception;
}

public class ActiveObjectEngine {
    LinkedList commands = new LinkedList();

    public void addCommand(Command c) {
        commands.add(c);
    }

    public void run() throws Exception {
        while (!commands.isEmpty()){
            Command c = (Command) commands.getFirst();
            commands.removeFirst();
            c.execute();
        }
    }
}
```

---

클린 소프트웨어 <로버트 C.마틴>
