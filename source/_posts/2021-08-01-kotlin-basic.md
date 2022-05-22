---
layout: post
title:  "코틀린 기초"
date:   2021-07-30
categories: Kotlin
---

코틀린의 기초적인 내용들을 정리한다.

- 함수, 변수
- 클래스, 프로퍼티
- enum, when
- iteration
- 예외처리

## 함수

두 값 중 큰 값을 반환하는, max 함수를 작성해보자.

```kotlin
fun max(a: Int, b: Int): Int {
    return if (a > b) a else b
}
```

위 함수 처럼, 본문이 중괄호로 둘러싸안 함수를 "블록이 본문인 함수" 라고 한다.

```kotlin
fun max(a: Int, b: Int): Int = if (a > b) a else b
```

위 함수 처럼, 본문이 등호와 식으로 이루어진 함수를 "식이 본문인 함수" 라고 한다.
"식이 본문인 함수" 는 반환 타입을 작성하지 않아도 컴파일러가 식의 결과 타입을 반환 타입으로 지정한다.
이렇게 컴파일리거 타입을 분석해 프로그램 구성 요소의 타입을 지정하는 기능을 타입 추론이라고 한다.

그래서, 위 함수를 아래 처럼 수정할 수 있다.

```kotlin
fun max(a: Int, b: Int) = if (a > b) a else b
```

> expression(식) 은 값을 만들어내고, 다른 expression 의 하위 요소로 계산에 참여 가능
> statement(문) 은 값을 만들어내지 않고, 자신을 둘러싸고 있는 가장 안쪽 불록의 최상위 요소

## 변수

```kotlin
val name = "jko"
val age: Int = 20
```

"식이 본문인 함수" 처럼, 타입을 지정하지 않으면 컴파일러가 초기화 식을 분석해서 초기화 식의 타입을 변수 타입으로 지정한다.
초기화 식이 없다면 변수에 저장될 값에 대한 정보가 없어서, 컴파일러가 타입을 추론 할 수 없다.
그래서, 이런 경우 타입을 지정해야한다.

```kotlin
val answer: Int
answer = 33
```

변수 선언 시 사용되는 키워드는 var, val 두 가지가 있다.

1. var : 변수의 값이 바뀔 수 있다. (variable)
2. val : 초기화했으면, 재대입이 불가능하다. (value)

주의할 점은, val 참조 자체는 불변이라도 그 참조가 가리키는 객체 내부의 값은 변경될 수 있다.

```kotlin
val languages = arrayListOf("Java")
languages.add("Kotlin")
```

## 문자열 템플릿

변수를 문자열 안에 사용할 수 있다. 변수 앞에 $를 추가하면 된다.

```kotlin
val name = "Kotlin"
println("Hello, $name")
```

## 클래스

name, isMarried 프로퍼티를 가지는 Person 클래스를 작성하자.

```kotlin
class Person(
    val name: String,
    var isMarried: Boolean
)
```

코틀린의 기본 가시성은 public 이므로, 변경자를 생략해도 된다.

## 프로퍼티

자바에서는, 필드와 접근자(getter, setter) 를 묶어서 property 라고 부른다.
코틀린 프로퍼티는 자바의 필드와 접근자 메서드를 완전히 대신한다.

프로퍼티를 선언할 때는 변수 선언곽 마찬가지로, val 이나 var 를 사용한다.
val 로 선언한 name 프로퍼티는 읽기 전용이다. private 이며, public getter 를 만들어낸다.
var 로 선언한 isMarried 프로퍼티는 변경 가능하다. private 이며, public getter/setter 를 만들어낸다.

사용할 때는 다음과 같이 사용한다.

```kotlin
val person = Person("jko", false)

println(person.name)
println(person.isMarried)

person.isMarried = true
```

## 커스텀 접근자

프로퍼티의 접근자를 직접 작성하는 방법이 있다.

```kotlin
class Rectangle(
    private val height: Int,
    private val width: Int
) {
    val isSquare: Boolean
        get() = height == width // property getter
}
```

isSquare 프로퍼티에는 자체 값을 저장할 필요가 없다.
이 프로퍼티에는 자체 구현을 제공하는 게터만 존재한다.
클라이언트가 프로퍼티에 접근할 때마다, 게터가 프로퍼티 값을 매번 다시 계산한다.

## enum

enum 클래스 안에는 프로퍼티나 메서드를 정의할 수 있다.

```kotlin
enum class Color(
    private val r: Int,     // 상수의 프로퍼티 정의
    private val g: Int,
    private val b: Int
) {

    RED(255, 0, 0),         // 각 상수 생성할 때, 프로퍼티 값을 지정
    ORANGE(255, 165, 0),
    YELLOW(255, 255, 0);    // 새미콜론 필수 (코틀린에서 유일)

    fun rgb() = (r * 256 + g) * 256 + b
}
```

## when

Java 의 switch 에 대응하는 구성 요소가 when 이다.
if 처럼 when 도 값을 만들어내는 식이다. 그래서, 식이 본문인 함수에서 when 을 바로 사용할 수 있다.
그리고, 한 분기 안에 여러 값을 매치 패턴으로 사용할 수 있는데 이럴 때는 콤마로 분리한다.

```kotlin
fun get(color: Color) =
    when (color) {
        Color.RED, Color.ORANGE -> "This is Red, Orange"
        Color.YELLOW -> "This is Yellow"
    }
```

분기 조건에 상수만을 허용하는 Java 의 switch 와 다르게, when 의 분기 조건은 객체를 허용한다.

```kotlin
fun mix(c1: Color, c2: Color) =
    when (setOf(c1, c2)) {
        setOf(Color.RED, Color.YELLOW) -> Color.ORANGE
        setOf(Color.RED, Color.ORANGE) -> Color.YELLOW
        else -> throw Exception("Nothing")
    }
```

## smart cast

어떤 번수가 원하는 타입인지 is 로 검사하면, 굳이 변수를 원하는 타입으로 캐스팅 하지 않아도 된다.
처음부터 그 변수가 원하는 타입으로 선언된 것처럼 사용할 수 있다.
컴파일러가 캐스팅을 해주기 때문이다. 이를 스마트 캐스트라고 한다.
원하는 타입으로 명시적으로 타입 캐스팅을 하려면,

```kotlin
val n = e as Num
```

## iteration

수에 대한 iteration 예를 보자.

```kotlin
for (i in 1..100){
    println(i)
}

for (i in 100 downTo 1 step 2){
    println(i)
}
```

100 downTo 1 는 역방향 수열을 만든다.
step 2 는 증가 값의 절대값이 2로 바뀐다.

map 에 대한 iteration 예를 보자.

```kotlin
val map = TreeMap<Char, String>()

for(c in 'A'..'F'){
    map[c] = Integer.toBinaryString(c.code)
}

for ((letter, binary) in map){
    println("$letter = $binary")
}
```

## in

in 연산자를 이용해서, 어떤 값이 범위에 속하는지 검사 가능하다.

```kotlin
fun isLetter(c: Char) = c in 'a'..'z' || c in 'A'..'Z'

fun recognize(c: Char) = when (c) {
    in '0'..'9' -> "yes"
    in 'a'..'z' -> "no"
    else -> "?"
}
```

## 예외 처리

자바에서는 함수를 작성할 때, 함수 선언 뒤에 체크 예외를 던지면 throws IOException 처럼 붙인다.
IOException 이 checked exception 이기 때문에, 명시적으로 체크 예외를 처리해야한다.

하지만 코틀린에서는, checked exception 과 unchecked exception 을 구별하지 않는다.
함수가 던지는 예외를 지정하지 않을 뿐만 아니라, 발생한 예외를 잡아내도 되고 잡아내지 않아도 된다.

try 키워드는 if 나 when 처럼 식이다. 그래서, try 의 값을 변수에 대입 가능하다.

```kotlin
fun readNumber(reader: BufferedReader) {
    val number = try {
        Integer.parseInt(reader.readLine()) // 이 식의 값이 try 식의 값이 됨
    } catch (e: NumberFormatException) {
        null
    }
}
```

---
Kotlin In Action <드미트리 제메로프, 스베트라나 이사코바>
