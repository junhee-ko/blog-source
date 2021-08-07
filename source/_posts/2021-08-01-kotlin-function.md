---
layout: post 
title:  "함수 정의와 호출"
date:   2021-08-01 
categories: Kotlin
---

다음 내용들을 정리한다.

- 함수 정의와 호출
- 확장 함수와 확장 프로퍼티

## 컬렉션

코틀린에서의 자체 컬렉션 기능은 없다. 
자바의 컬렉션을 사용한다.

```kotlin
val set = hashSetOf(1, 2, 3)
val list = arrayListOf(1, 2, 3)
val map = hashMapOf(1 to "one", 2 to "two", 3 to "three ")

println(set.javaClass) // java 의 getClass() 와 동일
println(list.javaClass)
println(map.javaClass)
```

출력 결과는 다음과 같다.

```text
class java.util.HashSet 
class java.util.ArrayList 
class java.util.HashMap
```

## 함수 구현

collection 의 원소에 prefix, postfix 를 붙이고 separator 로 구분짓는 메서드를 만들자.

```kotlin
fun <T> joinToString(
    collection: Collection<T>,
    separator: String,
    prefix: String,
    postfix: String
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in collection.withIndex()) {
        if (index > 0) result.append(separator)
        result.append(element)
    }

    result.append(postfix)
    return result.toString()
}

fun main() {
    val list = listOf(1, 2, 3)
    println(joinToString(list, "; ", "(", ")"))
}
```

출력 결과는 다음과 같다.

```text
(1; 2; 3)
```

## 이름 붙인 인자

함수 호출 부분의 가독성을 높여보자. 
함수에 전달하는 인자 중, 일부 또는 전부의 이름을 명시할 수 있다.

```kotlin
joinToString(
    collection = list, 
    separator = "; ", 
    prefix = "(", 
    postfix = ")"
)
```

## 디폴트 파라미터 값

자바에서는 하위 호환성을 유지하거나 API 사용자에게 편의를 주기 위해, 오버로딩 메서드들이 만들어진다. 
즉, 코드 중복의 문제가 발생한다.

코틀린에서는, 함수 선언에서 파라미터의 디폴트 값을 지정함으로써 이 문제를 해결한다.

```kotlin
fun <T> joinToString(
    collection: Collection<T>,
    separator: String = ", ",
    prefix: String = "",
    postfix: String = ""
)
```

그러면, 함수를 호출할 때, 인자를 모두 쓰거나 일부 생략이 가능하다.

```kotlin
joinToString(list)
joinToString(list, "; ")
```

## 최상위 함수

자바에서는, 모든 코드를 클래스의 메서드로 작성해야한다. 
그래서, 정적 메서드를 모아두는 역할을 하는 Util 이라는 이름이 붙은 클래스가 만들어지는 경우가 있다.

코틀린에서는, 함수를 직접 소스 파일의 최상위 수준에 위치시키면 된다. 
그리고 코틀린 컴파일러는 소스파일의 이름에 대응하는 클래스를 생성해낸다.

```kotlin
package function

fun <T> joinToString(
    collection: Collection<T>,
    separator: String = ", ",
    prefix: String = "",
    postfix: String = ""
)
```

## 확장 함수와 확장 프로퍼티

확장 함수란, 어떤 클래스의 메서드인 것처럼 호출할 수 있지만 그 클래스의 밖에 선언된 함수이다.

```kotlin
package strings

fun String.lastChar(): Char = this.get(this.length - 1)

fun main() {
    println("Kotlin".lastChar())
}
```

확장 함수를 만들기 위해서는, 추가하려는 함수 이름 앞에 함수가 확장할 클래스의 이름을 붙이면 된다.
클래스 이름을 "수신 객체 타입", 호출 되는 대상이 되는 값을 "수신 객체" 라고 한다.

위 코드에서는,
- String : 수신 객체 타입
- "Kotlin" : 수신 객체

## 확장 함수 임포트

함장 함수를 사용하려면, 그 함수를 임포트해야한다.

```kotlin
package stringsclient

import strings.lastChar

fun main() {
    println("Kotlin".lastChar())
}
```

as 키워드를 붙이면 임포트한 클래스나 함수를 다른 이름으로 호출할 수 있다.

```kotlin
package stringsclient

import strings.lastChar as last

fun main() {
    println("Kotlin".last())
}
```

## 확장 함수로 정의

이제 joinToString 함수를 확장함수로 만들어보자.

```kotlin
fun <T> Collection<T>.joinToString( // Collection<T> 에 대한 확장 함수 선언
    separator: String = ", ",
    prefix: String = "",
    postfix: String = ""
): String {
    val result = StringBuilder(prefix)
    for ((index, element) in this.withIndex()) { // this: 수신 객체를 가리킴. 여기서는, Collection<T>
        if (index > 0) result.append(separator)
        result.append(element)
    }

    result.append(postfix)
    return result.toString()
}
```

사용할 때는,

```kotlin
import function.joinToString

fun main() {
    val list = listOf(1, 2, 3)
    val joinToString = list.joinToString(separator = "; ", prefix = "(", postfix = ")")
    println(joinToString)
}

```

## 확장 프로퍼티

```kotlin
var StringBuilder.lastChar: Char
    get() = get(length - 1)
    set(value: Char) {
        this.setCharAt(length - 1, value)
    }

fun main() {
    val stringBuilder = StringBuilder("Kotlin?")
    val lastChar = stringBuilder.lastChar
    println(lastChar)
}
```

## 자바 컬렉션 API 확장

```kotlin
fun main() {
    val list = listOf("one", "two", "three")
    println(list.last())

    val set = setOf(1, 2, 3)
    println(set.maxOrNull())
}
```

어떻게 Java 의 컬렉션 API 에 정의되어있지 않은 last() 와 maxOrNull() 을 호출할 수 있었을까 ?
last() 와 maxOrNull() 이 확장함수 이기 때문이다.
다음과 같이 정의되어 있다.

```kotlin
public fun <T> List<T>.last(): T {
    if (isEmpty())
        throw NoSuchElementException("List is empty.")
    return this[lastIndex]
}

@SinceKotlin("1.4")
public fun <T : Comparable<T>> Iterable<T>.maxOrNull(): T? {
    val iterator = iterator()
    if (!iterator.hasNext()) return null
    var max = iterator.next()
    while (iterator.hasNext()) {
        val e = iterator.next()
        if (max < e) max = e
    }
    return max
}
```

## 가변 인자 함수

리스트를 생성할 때, 다음과 같이 원하는 만큼 인자를 전달할 수 있다.

```kotlin
val list = listOf(1,2,3,4,5,6)
```

listOf 함수의 정의를 보자.

```kotlin
public fun <T> listOf(vararg elements: T): List<T> = if (elements.size > 0) elements.asList() else emptyList()
```

vararg 는, 자바에서의 ... 와 동일한 역할을 한다.

## 중위 호출

맵을 만들기 위해서는 

```kotlin
val map = mapOf(1 to "one", 2 to "two")
```

여기서 to 는 일반 메서드이다.
중위 호출 이라는 방식으로 to 라는 일반 메서드를 호출한 것이다.
다음 두 호출은 동일하다.

1.to("one")
1 to "one"

인자가 하나뿐인 일반 메서드나 확장 함수에 중위 호출을 사용할 수 있다.
중위 호출이 가능하게 만들려면, infix 변경자를 함수 선언 앞에 붙이면 된다.

to 메서드는 다음과 같이 정의되어 있다.

```kotlin
public infix fun <A, B> A.to(that: B): Pair<A, B> = Pair(this, that)
```

## 로컬 함수와 확장

```kotlin
class User(val id: Int, val name: String, val address: String)

fun saveUser(user: User) {
    if (user.name.isEmpty()) {
        throw IllegalArgumentException("${user.id}")
    }

    if (user.address.isEmpty()) {
        throw IllegalArgumentException("${user.id}")
    }

    // save 
}
```

검증 코드를 로컬함수로 분리해보자.

```kotlin
class User(val id: Int, val name: String, val address: String)

fun saveUser(user: User) {
    fun validate(user: User, value: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException("${user.id}")
        }
    }

    validate(user, user.name)
    validate(user, user.address)

    // save
}
```

위 코드에서, 로컬 함수에게 user 객체를 계속 전달하고 있다.
하지만, 사실 전달할 필요가 없다.
로컬함수는 자신이 속한 바깥 함수의 모든 파라미터에 접근 가능하기 때문이다.

```kotlin
fun saveUser(user: User) {
    fun validate(value: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException("${user.id}")
        }
    }

    validate(user.name)
    validate(user.address)

    // save
}
```

위 코드를 더 개선하고자 하면, 검증 로직을 User 클래스의 확장 함수로 만들 수도 있다.

```kotlin
class User(val id: Int, val name: String, val address: String)

fun User.validateBeforeSave() {
    fun validate(value: String) {
        if (value.isEmpty()) {
            throw IllegalArgumentException("$id")
        }
    }

    validate(name)
    validate(address)
}

fun saveUser(user: User) {
    // save
}
```

---
Kotlin In Action <드미트리 제메로프, 스베트라나 이사코바>
