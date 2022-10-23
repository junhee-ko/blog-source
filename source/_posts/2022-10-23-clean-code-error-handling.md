---
layout: post
title: Error Handling
date: 2022-10-23
categories: Clean Code
---

오류 처리 코드로 프로그램 논리를 이해하기 어려워지면, 깨끗한 코드가 아니다.
오류를 처리하는 기법과 고려 사항을 정리하자.

## 오류 코드 보다 예외를 사용하자

오류 코드를 반환하는 코드를 보자.

```java
public class DeviceController {

    public void sendShutDown() {
        DeviceHandle handle = getHandle(DEV1);
        if (handle != DeviceHandle.INVALID) {
            DeviceRecord record = retrieveDeviceRecord(handle);
            if (record.getStatus() != DEVICE_SUSPENDED) {
                ...
            }
        }
    }
}
```

위 코드는, 호출자 코드가 복잡하다.
함수를 호출한 즉시, 오류를 확인해야하기 때문이다.

아래 처럼, 예외를 던지자.

```java
public class DeviceController {

    public void sendShutDown() {
        try {
            tryToShutDown();
        } catch (DeviceShutDownError e) {
            logger.log(e);
        }
    }

    private void tryToShutDown() throws DeviceShutDownError {
        DeviceHandle handle = getHandle(DEV1);
        DeviceRecord record = retrieveDeviceRecord(handle);

        pauseDevice(handle);
        ...
    }
}
```

## Try-Catch-Finally 문부터 작성하자

try 블록은 트랜잭션과 비슷하다.
try 블록에서 무슨 일이 생기든, catch 블록은 프로그램 상태를 일광성 있게 유지해야한다.

예외가 발생할 코드를 짤 때는, try-catch-finally 문으로 시작하자.
그러면, try 블록에서 무슨 일이 생기든 호출자가 기대하는 상태를 정의하기 쉽다.

## Unchecked 예외를 사용하자

Checked 예외는 OCP 를 위반한다.
메서드에서 Checked 예외를 던졌는데, catch 블록이 세 단계 위에 있다고 하자.
그 사이 메서드 모두가 선언부에 해당 예외를 정의해야한다.
즉, 하위 단계에서 코드를 변경하면 상위 단계 메서드 선언부 전부를 변경해야한다.

## 예외에 의미를 제공하자

예외를 던질 때, 오류 메세지에 정보를 담아 예외와 함께 던지자.
그러면 오류가 발생한 원인과 위치를 찾기가 쉽다.

## 호출자를 고려해서 예외 클래스를 정의하자

외부 라이브러리가 던질 예외를 모두 잡아내는 코드를 보자.

```java
ACMEPort port = new ACMEPort(12);

try {
    port.open();
} catch (DeviceResponseException e) {
    reportPortError(e);
} catch (ATM1212unlockedException e) {
    reportPortError(e);
} catch (...) {
    ...
} finally {
    ...
}
```

호출하는 라이브러리 API 를 감싸면서, 예외 유형 하나를 반환하는 코드를 보자.

```java
LocalPort port = new LocalPort(12);
try {
    prot.open();
} catch (PortDeviceFailure e) {
    reportError(e);
} finally {
    ...
}

public class LocalPort {
    private ACMEPort innerPort;

    public LocalPort(int portNumber) {
        innerPort = new ACMEPort(portNumber);
    }

    public void open() {
        try {
            innerPort.open();
        } catch (DeviceResponseException e){
            throw new PortDeviceFailure(e);
        } catch (ATM1212unlockedException e) {
            throw new PortDeviceFailure(e);
        }
  }
}
```

위와 같이, 외부 API 를 감싸면 외부 라이브러리와 프로그램 사이에 의존성이 줄어든다.
나중에 다른 라이브러리로 바꿔도 비용이 적다.
또한, 외부 라이브러리의 API 에 의존하지 않고, 프로그램이 사용하기 편리한 API 를 정의할 수 있다.

그리고, 외부 API 호출하는 대신 테스트 코드를 넣어서 프로그램 테스트도 쉽다.

## 정상 흐름을 정의하자

아래 코드를 보자.

```java
try {
    MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
    m_total += expenses.getTotal();
} catch(MealExpensesNotFound e) {
    m_total += getMealPerDiem();
}
```

식비를 비용으로 청구했으면, 직원이 청구한 식비를 총계에 더한다.
청구하지 않았으면, 기본 식비를 총계에 더한다.

예외가 논리를 따라가기 어렵게 만든다.
클래스에서 예외 상황을 캡슐화해서 클라이언트 코드가 예외 상황을 처리하지 않도록 하자.

ExpenseReportDAO 는 언제나 MealExpenses 객체를 반환하도록 한다.
청구한 식비가 없다면 일일 기본 식비를 반환하는 MealExpenses 객체를 반환하면 된다.

```java
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
m_total += expenses.getTotal();

public class PerDiemMealExpenses implements MealExpenses {
    public in getTotal() {
        // 기본값으로 일일 기본 식비를 반롼
    }
}
```

## null 을 반환하자 말자

null 을 반환하는 코드를 보자.

```java
public void registerItem(Item item) {
    if (item != null) {
        ItemRegistry registry = persistenceStore.getItemRegistry();
        if (registry != null) {
            Item existing = registry.getItem(item.getID());
            if (existing.getBillingPeriod().hasRetailOwner()){
                ...
            }
        }
  }
}
```

null 확인이 너무 많아 문제다.
메서드에서 null 을 반환하고 싶으면, 대신에 예외를 던지거나 특수 사례 객체를 반환하자.

또한, null 을 반환하는 아래 코드보다,

```java
List<Employee> employess = getEmployess();
if (employess != null){
    for(Employee e : employess){
        ...
    }
}
```

getEmployess() 에서, 빈 리스트를 반환하다면 아래처럼 작성할 수 있다.

```java
List<Employee> employess = getEmployess();
for(Employee e : employess){
    ...
}
```

## null 을 전달하지 말자

정상적인 인수로 null 을 기대하는 API 가 아니면, 메서드로 null 을 전달하지 말자.
애초에 null 을 전달하지 못하도록 금지할 필요가 있다.

```java
public class MetricsCalculator {
    public double xProjection(Point p1, Point p2){
        return (p2.x - p1.x) * 1.5;
    }
}

calculator.xProjection(null, new Point(12, 13));
```

호출자 코드에서 NPE 가 발생한다.
아래처럼 개선할 수 있다.

```java
public class MetricsCalculator {
    public double xProjection(Point p1, Point p2){
        if (p1 == null || p2 == null) {
            throw InvalidArgumentException("...");
        }
        return (p2.x - p1.x) * 1.5;
    }
}
```

위 코드는 InvalidArgumentException 를 catch 해서 처리하는 코드가 필요하다.
아래처럼 개선할 수 있다.

```java
public class MetricsCalculator {
    public double xProjection(Point p1, Point p2){
        assert p1 != null : "...";
        assert p2 != null : "...";

        return (p2.x - p1.x) * 1.5;
    }
}
```

---

클린코드 <로버트 C.마틴>
