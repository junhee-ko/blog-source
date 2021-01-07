---
layout: post
title:  "[오브젝트] 1장_객체, 설계"
date:   2019-11-04
categories: OOP
---

## 티켓 판매 애플리케이션

관람객이 초대장이 있으면 초대장을 티켓으로 바꿔서 극장에 입장하고, 없으면 티켓을 구입한 후에 극장에 입장하는 간단한 애플리케이션을 구현한다.

초대장을 구현한다.

```java
import java.time.LocalDateTime;

public class Invitation {

  private LocalDateTime when;
}
```

티켓을 구현한다.

```java
public class Ticket {

  private Long fee;

  public Long getFee() {
    return fee;
  }
}
```

관람객의 가방을 구현한다.

```java
public class Bag {

  private Long amount;
  private Invitation invitation;
  private Ticket ticket;

  public Bag(Long amount) {
    this(null, amount);
  }

  public Bag(Invitation invitation, Long amount) {
    this.invitation = invitation;
    this.amount = amount;
  }

  public boolean hasInvitation() {
    return invitation != null;
  }

  public boolean hasTicket() {
    return ticket != null;
  }

  public void setTicket(Ticket ticket) {
    this.ticket = ticket;
  }

  public void minusAmount(Long amount) {
    this.amount -= amount;
  }

  public void plusAmount(Long amount) {
    this.amount += amount;
  }
}
```

관람객을 구현한다.

```java
public class Audience {

  private Bag bag;

  public Audience(Bag bag) {
    this.bag = bag;
  }

  public Bag getBag() {
    return bag;
  }
}
```

티켓 판매소를 구현한다.

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class TicketOffice {

  private Long amount;
  private List<Ticket> tickets = new ArrayList<>();

  public TicketOffice(Long amount, Ticket... tickets) {
    this.amount = amount;
    this.tickets.addAll(Arrays.asList(tickets));
  }

  public Ticket getTicket() {
    return tickets.remove(0);
  }

  public void minusAmount(Long amount) {
    this.amount -= amount;
  }

  public void plusAmount(Long amount) {
    this.amount += amount;
  }
}
```

판매원을 구현한다.

```java
public class TicketSeller {

  private TicketOffice ticketOffice;

  public TicketSeller(TicketOffice ticketOffice) {
    this.ticketOffice = ticketOffice;
  }

  public TicketOffice getTicketOffice() {
    return ticketOffice;
  }
}
```

극장을 구현한다.

```java
public class Theater {

  private TicketSeller ticketSeller;

  public Theater(TicketSeller ticketSeller) {
    this.ticketSeller = ticketSeller;
  }

  public void enter(Audience audience) {
    if (audience.getBag().hasInvitation()) {
      Ticket ticket = ticketSeller.getTicketOffice().getTicket();
      audience.getBag().setTicket(ticket);
    } else {
      Ticket ticket = ticketSeller.getTicketOffice().getTicket();
      audience.getBag().minusAmount(ticket.getFee());
      ticketSeller.getTicketOffice().plusAmount(ticket.getFee());
      audience.getBag().setTicket(ticket);
    }
  }
}
```

## 무엇이 문제인가

로머트 마틴에 따르면 모든 모듈 (크기와 상관 없이 클래스나 패키지, 라이브러리와 같이 프로그램을 구성하는 임의의 요소) 은

1. 제대로 실행되어야 하고
2. 이해하기 쉽고
3. 변경이 용이해야한다.

위 티켓 판매 애플리케이션은 1번만 만족한다. 왜일까 ?

### 예상을 빗나가는 코드

현재의 코드는 우리의 예상과는 너무 다르게 동작하기 때문에 코드를 읽는 사람과 의사소통하지 못한다. 
관람객의 입장에서 문제는, 소극장이라는 제 3자가 초대장 확인을 위해 가방을 마음대로 열어본다. 
판매원 입장에서 문제는, 소극장이 매표소에 보관 중인 티켓과 현금을 마음대로 열어본다.

### 변경에 취약한 코드

세부적인 사실이 바뀌면 해당 클래스뿐만 아니라 이 클래스에 의존하는 Theater 도 함께 변경해야한다. 
그래서, 객체 사이의 결합도를 낮춰 변경이 용이한 설계를 만들어야한다.

## 설계 개선하기

자율성을 높여서, 개선해보자. 
즉, Audience 와 TicketSeller 가 직접 Bag 과 TicketOffice 를 처리하도록 자율적인 존재가 되도록 수정하자.

Theater 를 다음과 같이 수정한다.

```java
public class Theater {

  private TicketSeller ticketSeller;

  public Theater(TicketSeller ticketSeller) {
    this.ticketSeller = ticketSeller;
  }

  public void enter(Audience audience) {
    ticketSeller.sellTo(audience);
  }
}
```

TicketSeller 를 다음과 같이 수정한다.

```java
public class TicketSeller {

  private TicketOffice ticketOffice;

  public TicketSeller(TicketOffice ticketOffice) {
    this.ticketOffice = ticketOffice;
  }

  public TicketOffice getTicketOffice() {
    return ticketOffice;
  }

  public void sellTo(Audience audience) {
    if (audience.getBag().hasInvitation()) {
      Ticket ticket = ticketOffice.getTicket();
      audience.getBag().setTicket(ticket);
    } else {
      Ticket ticket = ticketOffice.getTicket();
      audience.getBag().minusAmount(ticket.getFee());
      ticketOffice.plusAmount(ticket.getFee());
      audience.getBag().setTicket(ticket);
    }
  }
}
```

하지만, Audience 는 여전히 자율적인 존재가 아니다. 
TicketSeller 가 Audience 내부의 Bag 에 접근을 직접하고 있기 때문이다.

그래서, TicketSeller  다음과 같이 수정한다.

```java
public class TicketSeller {

  private TicketOffice ticketOffice;

  public TicketSeller(TicketOffice ticketOffice) {
    this.ticketOffice = ticketOffice;
  }

  public TicketOffice getTicketOffice() {
    return ticketOffice;
  }

  public void sellTo(Audience audience) {
    ticketOffice.plusAmount(audience.buy(ticketOffice.getTicket()));
  }
}
```

Audience  다음과 같이 수정한다.

```java
public class Audience {

  private Bag bag;

  public Audience(Bag bag) {
    this.bag = bag;
  }

  public Bag getBag() {
    return bag;
  }

  public long buy(Ticket ticket) {
    if (bag.hasInvitation()) {
      bag.setTicket(ticket);
      return 0L;
    } else {
      bag.setTicket(ticket);
      bag.minusAmount(ticket.getFee());
      return ticket.getFee();
    }
  }
}
```
## 무엇이 개선됐는가

Audience 와 TicketSeller 의 내부 구현을 변경하더라도 Theater 를 함께 변경할 필요가 없어졌다. 
수정된 코드는 변경 용이성의 측면에서 확실히 개선됐다.

## 어떻게 한 것인가

객체의 자율성을 높였다.
핵심은, 객체 내부의 상태를 캡슐화하고 객체 간에 오직 메시지를 통해 상호작용 하도록 만드는 것이다.

## 더 개선할 수 있다

Bag 을 자율적인 존재로 만들자.

```java
public class Bag {

  private Long amount;
  private Invitation invitation;
  private Ticket ticket;

  public Bag(Long amount) {
    this(null, amount);
  }

  public Bag(Invitation invitation, Long amount) {
    this.invitation = invitation;
    this.amount = amount;
  }

  public Long hold(Ticket ticket) {
    if (hasInvitation()) {
      setTicket(ticket);
      return 0L;
    } else {
      setTicket(ticket);
      minusAmount(ticket.getFee());
      return ticket.getFee();
    }
  }

  private boolean hasInvitation() {
    return invitation != null;
  }

  private void setTicket(Ticket ticket) {
    this.ticket = ticket;
  }

  private void minusAmount(Long amount) {
    this.amount -= amount;
  }
}
```

이제 Audience 는 Bag 의 구현이 아닌, 인터페이스에만 의존할 수 있다.

```java
public class Audience {

  private Bag bag;

  public Audience(Bag bag) {
    this.bag = bag;
  }

  public long buy(Ticket ticket) {
    return bag.hold(ticket);
  }
}
```

TickSeller 역시 TicketOffice 에 있는 Ticket 을 마음대로 접근하고 있다. 
개선하자.

TickSeller 를 수정한다.

```java
public class TicketSeller {

  private TicketOffice ticketOffice;

  public TicketSeller(TicketOffice ticketOffice) {
    this.ticketOffice = ticketOffice;
  }

  public void sellTo(Audience audience) {
    ticketOffice.sellTicketTo(audience);
  }
}
```

TicketOffice 를 수정한다.

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class TicketOffice {

  private Long amount;
  private List<Ticket> tickets = new ArrayList<>();

  public TicketOffice(Long amount, Ticket... tickets) {
    this.amount = amount;
    this.tickets.addAll(Arrays.asList(tickets));
  }

  public void sellTicketTo(Audience audience) {
    plusAmount(audience.buy(getTicket()));
  }

  private Ticket getTicket() {
    return tickets.remove(0);
  }

  private void plusAmount(Long amount) {
    this.amount += amount;
  }
}
```

## 의인화

Theater, Bag, TicketOffice 는 실세계에서 자율적인 존재가 아니다. 
하지만 비록 현실에서는 수동적인 존재라고 하더라도 객체지향 세계에서는 모든 것이 능동적이고 자율적인 존재가 된다. 
이것을 의인화라고 한다.

## 객체지향 설계

훌륭한 객체지향 설계는, 협력하는 객체 사이의 의존성을 적절하게 관리하는 설계다. 
협력하는 객체들 사이의 의존성을 적절하게 조절해서 변경에 용이한 설계를 만들어야한다.

## Source Code

https://github.com/junhee-ko/object-ticket-example

---

오브젝트 <조영호>