---
layout: post 
title: Convention
date:   2021-09-18
categories: Kotlin
---

코틀린의 관례에 대해 다음 순으로, 정리한다.

1. 산술 연산자
2. 비교 연산자
3. 컬렉션, 범위
4. 구조 분해 선언

관례란, 언어 기능과 미리 정해진 이름의 함수를 연결해주는 기법이다.

## 산술 연산자 오버로딩

자바에서는, 
1. 원시 타입에 대해서 산술 연산자를 사용할 수 있으며
2. String 에 대해 + 연산자를 사용할 수 있다.

코틀린에서는, 다른 클래스에도 산술 연산자를 사용할 수 있다.

### 이항 산술 연산 오버로딩

연산자를 오버로딩하는 함수 앞에는 operator 키워드가 붙어야한다.
다음을 보자.

```kotlin
fun main() {
    val p1 = Point(10, 20)
    val p2 = Point(30, 40)

    println(p1 + p2)
}

data class Point(val x: Int, val y: Int) {
    
    operator fun plus(other: Point): Point {
        return Point(x + other.x, y + other.y)
    }
}
```

실행 결과는 다음과 같다.

```kotlin
Point(x=40, y=60)
```

연산자를 멤버 함수로 만드는 대신, 확장 함수로 정의할 수 있다.
외부 함수의 클래스에 대한 연산자를 정의할 때는, 
관례를 따르는 이름의 확장 함수로 구현하는 것이 일반적인 패턴이다.

```kotlin
operator fun Point.plus(other: Point): Point {
    return Point(x + other.x, y + other.y)
}
```

### 복합 대입 연산 오버로딩

plus 같은 연산자를 오버로딩 하면, +=, -= 등의 복합 대입 연산자도 같이 지원한다.
다음을 보자.

```kotlin
var point = Point(1,2)
point += Point(3,4)
```

### 단항 연산자 오버로딩

```kotlin
operator fun BigDecimal.inc() = this + BigDecimal.ONE

var number = BigDecimal.ZERO
println(number++) // 0
println(++number) // 2
```

## 비교 연산자 오버로딩

산술 연산자와 마찬가지로, 원시 타입 값 뿐만 아니라 모든 객체에 대해 비교 연산을 수행할 수 있다.

### 동등성 연산자 : equals

동등성 검사는 equals 호출과 null 검사로 컴파일 된다.
즉,

```kotlin
a == b
```

는 아래와 같이 컴파일 된다.

```kotlin
a?.equals(b) ?: (b == null)
```

data class 의 경우에, 컴파일러가 자동으로 equals 메서드를 생성해준다.
직접 구현하면 다음과 같다.

```kotlin
data class Point(val x: Int, val y: Int){
    override fun equals(other: Any?): Boolean {
        if (this === other) return true
        if (other !is Point) return false

        return other.x == x && other.y == y
    }
}
```

위 equals 메서드에 대해 정리해보자.

1. override
    다른 연산자 오버로딩 관례와 달리, equals 는 Any 에 정의된 메서드라서 override 를 붙여야한다.
    operator 키워드를 붙이지 않아도, 상위 클래스의 operator 지정이 적용된다.
2. ===
    자바의 == 와 같다.
    두 피연산자가 서로 같은 객체를 가리키는지 비교한다.
3. is
    타입을 검사한다.
    Point 로 스마트 캐스트 된 뒤에, x 와 y 의 프로퍼티에 접근하고 있다.

### 순서 연산자 : compareTo

자바에서 정렬이나, 최대값과 최소값을 비교하려면 Comparable 인터페이스를 구현해야한다.
Comparable 의 compareTo 메서드는 한 객체와 다른 객체의 크기를 비교해 정수로 나타낸다.
아래와 같이.

```java
element1.compareTo(element2)
```

코틀린에서도 같은 Comparable 인터페이스를 지원한다.
자바와 다른 점은, compareTo 메서드를 호출하는 관례를 제공한다.

즉, 다음 코드는

```kotlin
a >= b
```

아래와 같이 컴파일 된다.

```kotlin
a.compareTo(b) >= 0
```

다음 예를 보자.

```kotlin
class Person(
    private val firstName: String,
    private val lastName: String
) : Comparable<Person> {

    override fun compareTo(other: Person): Int {
        return compareValuesBy(
            this,
            other,
            Person::lastName,
            Person::firstName
        )
    }
}
```

equals 와 마찬가지로, Comparable 의 compareTo 에도 operator 변경자가 붙어 있어서
하위 클래스의 오버라이딩 함수에 operator 를 붙이지 않아도 된다.

## 컬렉션과 범위에 쓸 수 있는 관례

### 인덱스로 원소에 접근 : get, set

각괄호를 사용한 접근은 get 함수 호출로 컴파일된다.

```kotlin
operator fun Point.get(index: Int): Int {
    return when(index){
        0 -> x
        1 -> y
        else -> throw IndexOutOfBoundsException()
    }
}

val point = Point(3, 4)
println(point[0])
println(point[1])
```

각 괄호를 사용한 대입문은 set 함수 호출로 컴파일 된다.

```kotlin
data class MutablePoint(var x: Int, var y: Int)

operator fun MutablePoint.set(index: Int, value: Int) {
    return when(index){
        0 -> x = value
        1 -> y = value
        else -> throw IndexOutOfBoundsException()
    }
}

val mutablePoint = MutablePoint(1, 2)
mutablePoint[0] = 5
mutablePoint[1] = 6
```

### in

in 연산자는 contains 함수 호출로 변환된다.

```kotlin
val numbers = listOf(1, 2, 3)
println(1 in numbers)
```

위의 경우,
Collection interface 에 다음과 같이 정의되어 있기 때문에, contains 함수 호출로 변환되는 것이다. 

```kotlin
public operator fun contains(element: @UnsafeVariance E): Boolean
```

### rageTo

.. 연산자는 rageTo 함수 호출로 변환된다.

### iterator

for loop 는 범위 검사와 똑같이, in 연산자를 사용한다.
하지만 의미는 다르다.

```kotlin
for (x in list) {...}
```

위 코드는,

1. list.iterator() 를 호출해서 이터레이션을 얻는다.
2. 그리고, 이터레이션에 대해 hasNext 와 next 호출을 반복한다.

## 구조 분해 선언 (Destructing Declaration)

구조분해 선언은 componentN 함수 호출로 변환된다.
다음 코드는,

```kotlin
val (a, b) = p
```

아래와 같이 컴파일된다.

```kotlin
val a = p.component1()
val b = p.component2()
```

그래서 아래와 같은 코드가 가능한 것이다.

```kotlin
val p = Point(3, 5)
val (x, y) = p
println(x)
println(y)
```

다음 예를 보자.

```kotlin
fun print(map: Map<String, String>) {
    for ((key, value) in map) {
        println("$key -> $value")
    }
}
```

위 코드에서는 두 가지 관례가 사용되었다.

1. 이터레이션 
  표준 라이브러리에 맵에 대한 확장 함수로 iterator 가 있다.
  iterator 는 맵 원소에 대한 이터레이터를 반환한다.
2. 구조 분해 선언
  Map.Entry 에 대한 확장 함수로 component1 과 component2 를 제공한다.

즉,

```kotlin
fun print(map: Map<String, String>) {
    for (entry in map.entries) {
        val key = entry.component1()
        val value = entry.component2()
        println("$key -> $value")
    }
}
```

---
Kotlin In Action <드미트리 제메로프, 스베트라나 이사코바>
