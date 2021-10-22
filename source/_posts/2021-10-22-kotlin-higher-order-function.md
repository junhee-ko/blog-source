---
layout: post 
title: 고차함수
date:   2021-10-22
categories: Kotlin
---

코틀린의 고차 함수에 대해 정리한다.

## 고차 함수

고차 함수란, "다른 함수를 인자로 받거나 반환하는 함수" 이다.

그런데 코틀린에서는, 람다나 함수 참조를 사용해서 함수를 값으로 표현할 수 있다.
그래서, "람다나 함수 참조를 인자로 반거나 반환하는 함수" 도 고차 함수이다. 

예를 들어, 표준 라이브러리에 있는 함수인 filter 는 고차 함수이다.
filter 는 술어 함수를 인자로 받는 함수이기 때문이다.

```kotlin
val numbers = listOf("one", "two", "three", "four")
val longerThan3 = numbers.filter { it.length > 3 } // 술어함수를 인자로 받는 filter 
```

## 함수 타입

위에서 정리한 것 처럼, 람다를 인자로 받는 함수는 고차함수이다.
그러면 람다 인자의 타입은 어떻게 선언할까 ?

더 단순한 단계인, 람다를 로컬 변수에 대입하는 경우를 보자.

```kotlin
val sum = { x: Int, y: Int -> x + y }
val action = { println(sum) }
```

위의 경우, 컴파일러는 sum 과 action 의 함수 타입을 아래 처럼 추론한다.

```kotlin
val sum: (Int, Int) -> Int = { x: Int, y: Int -> x + y }
val action: () -> Unit = { println(sum) }
```

물론 아래처럼, 널이 될 수 있는 함수 타입을 정의할 수 있다.

```kotlin
val funOrNull : ((Int, Int) -> Int)? = null
```

## 인자로 받은 함수 호출

이제, 고차 함수를 구현하는 방법을 보자.

```kotlin
fun twoAndThree(operation: (Int, Int) -> Int) {
    val result = operation(2, 3)
    println("result is : $result")
}
```

인자로 받은 함수를 호출하는 구문은, 일반 함수를 호출하는 구문과 같다.
그리고, 위 고차함수를 호출할 때는 아래 처럼 람다를 전달하면 된다.

```kotlin
twoAndThree { a, b -> a + b }
twoAndThree { a, b -> a * b }
```

## 디폴트 값 지정

파라미터를 함수 타입으로 선언할 때, 디폴트 값을 지정할 수 있다.

다음 예시를 보자.
toString() 을 사용해서 객체를 문자열로 바꾼다.

```kotlin
fun <T> Collection<T>.joinToString(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = ""
): String {
    val result = StringBuilder(prefix)

    for ((index, element) in this.withIndex()) {
        if (index > 0) {
            result.append(separator)
        } else {
            result.append(element) // HERE !!
        }
    }
    result.append(postfix)

    return result.toString()
}
```

toString() 을 디폴트 값으로 지정하고, 호출하는 쪽에서 지정하게 만들 수 있다.

```kotlin
fun <T> Collection<T>.joinToString(
    separator: String = ", ",
    prefix: String = "",
    postfix: String = "",
    transform: (T) -> String = { it.toString() }  // HERE !!!
): String {
    val result = StringBuilder(prefix)

    for ((index, element) in this.withIndex()) {
        if (index > 0) {
            result.append(separator)
        } 
        result.append(transform(element))   // HERE !!!
        
    }
    result.append(postfix)

    return result.toString()
}
```

호출하는 쪽에서서 joinToString() 에 아무 값도 전달하지 않으면, 
default 로 정의한 변환 함수를 사용한다.

```kotlin
val letters = listOf("A", "B", "C")
val result2 = letters.joinToString()
val result1 = letters.joinToString { it.toLowerCase() }
```

## 함수를 함수에서 반환

"다른 함수를 반환하는" 함수를 정의하려면,
반환 타입으로 함수 타입을 지정하면 된다. 

```kotlin
enum class Delivery {
    STANDARD, EXPEDITED
}

class Order(val itemCount: Int)

fun getShippingCostCalculator(
    delivery: Delivery
): (Order) -> Double {  // HERE !!!
    if (delivery == Delivery.EXPEDITED) {
        return { order -> 6 + 2.1 * order.itemCount }
    }
    return { order -> 1.2 * order.itemCount }
}

fun main() {
    val shippingCostCalculator = getShippingCostCalculator(Delivery.EXPEDITED)
    val cost = shippingCostCalculator(Order(3))
    println(cost)
}
```

getShippingCostCalculator 함수는 
Order 를 받아서 Double 을 반환하는 함수를 반환한다.

## 람다를 활용한 중복 제거

코드 중복을 줄이기 위해, 함수 타입을 사용할 수 있다.

웹 사이트의 방문 기록을 분석하는 예를 보자.

```kotlin
data class SiteVisit(
    val path: String,
    val duration: Double,
    val os: OS
)

enum class OS {
    WINDOWS,
    LINUX,
    MAC,
    IOS,
    ANDROID
}

val log = listOf(
    SiteVisit("/", 34.0, OS.WINDOWS),
    SiteVisit("/login", 10.0, OS.ANDROID),
    SiteVisit("/signup", 12.0, OS.IOS),
    SiteVisit("/", 20.0, OS.MAC),
    SiteVisit("/", 40.1, OS.LINUX),
)
```

윈도우 사용자의 평균 방문 시간을 구하자.

```kotlin
val averageWindowsDuration = log
    .filter { it.os == OS.WINDOWS }
    .map(SiteVisit::duration)
    .average()
```

맥 사용자의 평균 방문 시간을 구하자.

```kotlin
val averageMacDuration = log
    .filter { it.os == OS.MAC }
    .map(SiteVisit::duration)
    .average()
```

중복 코드를 피하기 위해, 
확장 함수로 정의해서 다음과 같이 개선할 수 있다.

```kotlin
fun List<SiteVisit>.averageDuration(os: OS) = 
    filter { it.os == os }.map(SiteVisit::duration).average()

log.averageDuration(OS.MAC)
```

만약에, 다음과 같은 복잡한 질의에 대해 분석하고 싶을 때는 어떻게 할까 ?
"ANDROID 사용자의 /login 페이지 평균 방문 시간은 ? "

이 때, 람다를 적용할 수 있다.

```kotlin
fun List<SiteVisit>.averageDuration(predicate: (SiteVisit) -> Boolean) =
    filter(predicate).map(SiteVisit::duration).average()

log.averageDuration { it.os == OS.ANDROID && it.path == "/login" }
```

---
Kotlin In Action <드미트리 제메로프, 스베트라나 이사코바>
