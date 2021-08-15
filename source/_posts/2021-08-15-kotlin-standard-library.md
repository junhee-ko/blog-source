---
layout: post 
title:  "표준 라이브러리"
date:   2021-08-15 
categories: Kotlin
---

컬렉션을 다루는 코틀린 표준 라이브러리를 정리한다.

- filter, map
- all, any, count, find
- groupBy
- flatMap, flatten
- with, apply


## filter, map

filter 는 컬렉션에서 원치 않는 원소를 제거한다.

```kotlin
val list = listOf(1, 2, 3, 4)
println(list.filter { it % 2 == 0 })
```

결과는, 
컬렉션의 원소 중에, 주어진 술어 (==predicate, 참/거짓을 반환하는 함수) 를 만족하는 원소로 이루어진 새로운 컬렉션이다.

map 은 원소를 변환한다.

```kotlin
val list = listOf(1, 2, 3, 4)
println(list.map { it * it })
```

결과는, 
원본 리스트와 원소 개수는 같지만 각 원소는 주어진 함수에 따라 변환된 새로운 컬렉션이다.

## all, any, count, find

- all, any : 컬렉션의 모든 원소가 어떤 조건을 만족하는지 판단
- count : 조건을 만족하는 원소의 개수 반환
- find : 조건을 만족하는 첫 번째 원소 반환

어떤 사람의 나이가 25 살 이하인지 판단하는 술어 함수를 만들자.

```kotlin
val under25 = { p: Person -> p.age < 25 }
```

모든 원소가 이 술어를 만족하는지 판단하려면 all 을 함수를 쓰자.

```kotlin
val people = listOf(Person("jko", 30), Person("junhee-ko", 20))
println(people.all(under25))
```

술어를 만족하는 원소가 하나라도 있는지 판단하려면 any 를 쓰자.

```kotlin
val people = listOf(Person("jko", 30), Person("junhee-ko", 20))
println(people.any(under25))
```

술어를 만족하는 원소 개수를 구하려면, count 를 쓰자.

```kotlin
val people = listOf(Person("jko", 30), Person("junhee-ko", 20))
println(people.count(under25))
```

술어를 만족하는 원소 하나를 찾고 싶으면, find 를 쓰자.

```kotlin
val people = listOf(Person("jko", 30), Person("junhee-ko", 20))
println(people.find(under25))
```

## groupBy

컬렉션의 모든 원소를 어떤 특성에 따라 여러 그룹으로 나눌 때는, groupBy 함수를 쓰자.

```kotlin
val people = listOf(
    Person("jko", 30), Person("son", 30),
    Person("junhee-ko", 20)
)

println(people.groupBy { it.age })
```

## flatMap

flatMap 함수는 먼저 인자로 주어진 람다를 컬렉션의 모든 객체에 적용한다. (map)
그리고, 람다를 적용한 결과로 얻어지는 리스트를 한 리스트에 모은다. (flatten)

간단한 예를 보자.

```kotlin
val strings = listOf("abc", "def")
println(strings.flatMap { it.toList() })
```

![](/image/lamda-flatten-example.png)

다른 예를 보자.

```kotlin
val books = listOf(
    Book("Kotlin", listOf("junheeko", "jko")),
    Book("Java", listOf("junheeko", "Chulsu")),
    Book("C++", listOf("BTS", "PSY")),
)

println(books.flatMap { it.authors }.toSet()) // [junheeko, jko, Chulsu, BTS, PSY]
```

## with

다음 코드를 보자.

```kotlin
fun alphabet(): String {
    val result = StringBuilder()
    for (letter in 'A'..'Z') {
        result.append(letter)
    }

    result.append("\nThe End")

    return result.toString()
}
```

result 에 대해 다른 여러 메서드를 호출하면서 매번 반복 사용한다. 
이 예제를 개선해보자.

```kotlin
fun alphabet(): String {
    val stringBuilder = StringBuilder()
    return with(stringBuilder) {    // 수신 객체 지정
        for (letter in 'A'..'Z') {
            stringBuilder.append(letter)
        }

        append("\nThe End")

        this.toString()
    }
}
```

위 with 문은 파라미터가 2개인 함수이다. 
첫 번째 파라미터는 stringBuilder, 두 번째 파라미터는 람다이다.

with 함수는 첫 번째 인자로 받은 객체를, 두 번재 인자로 받은 람다의 수신 객체로 만든다. 
람다 본문 에서는, 
- this 를 사용해서 수신 객체에 접근하거나 
- 프로퍼티나 메서드 이름만 사용해도 수신 객체의 맴버에 접근 가능하다.

더 개선하면,

```kotlin
fun alphabet() = with(StringBuilder()) {
    for (letter in 'A'..'Z') {
        append(letter)
    }

    append("\nThe End")

    this.toString()
}
```

## apply

거의 with 과 같다. 
유일한 차이는, 항상 자신에게 전달된 객체 (수신 객체) 를 반환한다는 것 뿐이다.

```kotlin
fun alphabet() = StringBuilder().apply {
    for (letter in 'A'..'Z') {
        append(letter)
    }

    append("\nThe End")
}.toString()
```

apply 를 호출한 결과는 StringBuilder 객체다. 
그래서, 이 객체의 toString 을 호출해서 String 객체를 얻을 수 있다.

그런데, 표준 라이브러리의 buildString 함수를 사용하면 alphabet 함수를 더 단순화 할 수 있다. 
buildString 의 인자는 수신 객체 지정 람다이며, 수신 객체는 항상 StringBuilder 이다.

```kotlin
fun alphabet() = buildString {
    for (letter in 'A'..'Z') {
        append(letter)
    }

    append("\nThe End")
}
```

---
Kotlin In Action <드미트리 제메로프, 스베트라나 이사코바>
