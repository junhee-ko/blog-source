---
layout: post 
title:  "코틀린 기초"
date:   2021-08-01 
categories: Kotlin
---

코틀린의 기초적인 내용들을 정리한다.

- 함수, 변수, 클래스, enum, property
- 제어 구조
- smart cast
- exception

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
"식이 본문인 함수" 는 반환 타입을 작성하지 않아도 컴파일러가 식의 결과 타입을 반환 타입으로 지정해준다. 

그래서, 위 함수를 아래 처럼 수정할 수 있다.

```kotlin
fun max(a: Int, b: Int) = if (a > b) a else b
```

## 변수

```kotlin
val name = "jko"
val age: Int = 20
```

"식이 본문인 함수" 처럼, 타입을 지정하지 않으면 컴파일러가 초기화 식을 분석해서 초기화 식의 타입을 변수 타입으로 지정한다. 

변수 선언 시 사용되는 키워드는 var, val 두 가지가 있다.

1. var : 변수의 값이 바뀔 수 있다. (variable)
2. val : 변수의 값이 바뀔 수 없다. (value)

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
        get() = height == width
}
```

isSquare 프로퍼티에는 자체 값을 저장할 필요가 없다. 
이 프로퍼티에는 자체 구현을 제공하는 게터만 존재한다.

## enum

enum 클래스 안에는 프로퍼티나 메서드를 정의할 수 있다.

```kotlin
enum class Color(
    private val r: Int,
    private val g: Int,
    private val b: Int
) {

    RED(255, 0, 0),
    ORANGE(255, 165, 0),
    YELLOW(255, 255, 0);

    fun rgb() = (r * 256 + g) * 256 + b
}
```

코틀린에서 유일하게 세미콜론이 필수인 부분이다. 
enum class 안에 메서드를 정의 하는 경우에, enum 상수와 메서드 사이에 세미콜론이 필수다.

## when

Java 의 switch 에 대응하는 구성 요소가 when 이다. 
if 와 마찬가지로 when 도 값을 만들어내는 식이다. 
그래서, 식이 본문인 함수에서 when 을 바로 사용할 수 있다.

```kotlin
fun get(color: Color) =
    when (color) {
        Color.RED, Color.ORANGE -> "This is Red, Orange"
        Color.YELLOW -> "This is Yellow"
    }
```

분기 조건에 상수만을 허용하는 Java 의 switch 와 다르게 when 의 분기 조건은 아래와 같이 객체를 허용한다.

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
컴파일러가 캐스팅을 해준다.
Sum 타입인지 검사하고 난 뒤에, e.right 와 e.left 를 사용할 수 있다.

```kotlin
if( e is Sum){
    return eval(e.right) + eval(e.left)
}
```

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
fun recognize(c: Char) = when (c) {
    in '0'..'9' -> "yes"
    in 'a'..'z' -> "no"
    else -> "?"
}
```

## 예외 처리

checked exception 과 unchecked exception 을 구별하지 않는다.
함수가 던지는 예외를 지정하지 않을 뿐만 아니라, 발생한 예외를 잡아내도 되고 잡아내지 않아도 된다.

try 키워드는 if 나 when 과 마찬가지로 식이다. 그래서, try 의 값을 변수에 대입 가능하다.

```kotlin
fun readNumber(reader: BufferedReader) {
    val number = try {
        Integer.parseInt(reader.readLine())
    } catch (e: NumberFormatException) {
        null
    }
}
```

---
Kotlin In Action <드미트리 제메로프, 스베트라나 이사코바>
