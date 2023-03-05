---
layout: post
title: Facade && Mediator Pattern
date: 2023-03-03
categories: Agile
---

퍼사드 패턴과 미디에이터 패턴을 정리한다.

## Facade Pattern

복잡하고 일반적인 인터페이스를 가진 객체 그룹에 간단하고 구체적인 인터페이스를 제공할 때 사용된다.

![](/image/clean-software-facade-pattern-transaction-ex.png)

DB 클래스가 Application 이 java.sql 의 구체적인 내용을 알 필요가 없게 보호하고 있다.
즉, java.sql 의 일반성과 복잡성을 간단하고 구체적인 인터페이스 뒤에 숨긴다.

퍼사드 패턴의 사용은 개발자가 모든 DB 호출이 DB 클래스를 통과해야한다 라는 규정을 의미힌다.
Application 코드의 어떤 부분이 직접 java.sql 을 사용한다면 규정을 위반하는 것이다.
이 처럼, 퍼사드는 자신의 정책을 Application 에 적용하고 있다.

## Mediator Pattern

미디에이터 패턴도 정책을 적용한다.
퍼사드가 자신의 정책을 가시적이고 강제적인 방식으로 적용했다면, 미디에이터 패턴은 자신의 정책을 은밀하고 강제적이지 않은 방식으로 적용한다.

```java
public class QuickEntryMediator {

    private JTextField itsTextField;
    private JList itsList;

    public QuickEntryMediator(JTextField t, JList l){
        itsTextField = t;
        itsList = l;

        itsTextField.getDocument().addDocumentListerner(new DocumentListener() {
            public void changeUpdate(DocumentEvent e){
                textFieldChanged();
            }
        })
        // ...
    }

    private void textFieldChanged() {
        // ...
    }
}
```

QuickEntryMediator 는 익명 DocumentListener 를 JTextField 에 등록한다.
이 리스너는 텍스트에 변화가 있을 때마다, textFieldChanged 를 호출한다.

JTextField 와 JList 는 이 미디에이터의 존재를 모른다.
이 미디에이터는 은밀하고 강제적이지 않은 방식으로 자신의 정책을 적용한다.

---

클린 소프트웨어 <로버트 C.마틴>
