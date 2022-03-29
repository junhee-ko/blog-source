---
layout: post
title: Effective Kotlin - Readability 1
date: 2022-03-27
categories: Kotlin
---

코틀린을 가독성이 좋게 작성하는 방법을 정리한다.

- 가독성을 목표로 설계해라
- 연산자 오버로드할 때는 의미에 맞게 사용해라
- Unit? 을 리턴하지 마라
- 변수 타입이 명확하지 않으면 확실하게 지정해라
- 리시버를 명시적으로 참조해라
- 프로퍼티는 동작이 아니라, 상태를 나타내야한다

## 가독성을 목표로 설계해라

로버트 마틴의 클린코드에서 다음과 같은 내용이 있다.
`개발자가 코드 작성하는데 1분 걸리지만, 읽는데는 10분 걸린다`

프로그래밍은 쓰기보다 읽기가 중요하다는 의미이다.
그래서, 가독성을 생각하면서 코드를 작성해야한다.

### 인지 부하 감소

인지 부하를 줄이는 방향으로 코드를 작성하자.
자주 사용되는 패턴을 활용하면, 뇌가 프로그램의 작동 방식을 이해하는 과정을 더 짧게 만들 수 있다.

다음 두 코드를 보자.

```kotlin
// A
if (person != null && person.isAdult) {
    view.showPerson(person)
} else {
    view.showError()
}

// B
person?.takeIf { it.isAdult}
  ?.let(view.showPerson)
  ?: view.showError()
```

어느 코드가 더 좋을까 ? A 가 훨씬 가독성이 좋은 코드이다.
가독성이란, 코드를 읽고 얼마나 빠르게 이해할 수 있는지를 의미힌다.
구현 A 가 더 좋은 코드인 이유는,

1. 일반적인 관용구인 "if/else, &&, 메서드 호출" 을 사용하고 있기때문이다.
2. 수정하기 쉽기 때문이다. if 블록에 작업을 추가한다고 하면, 쉽게 추가할 수 있다.
3. 디버깅이 간단하기 떄문이다.
4. IDE 지원을 받을 수 있기 때문이다. 성인, 아동에 따라 다른 처리를 하는 조건문을 추가한다고 하면 인텔리제이의 리팩터링 기능을 사용할 수 있다.

그리고 참고로, 두 코드의 실행 결과는 다르다.
B 는 showPerson() 이 null 을 리턴하면, showError() 도 호출된다.

### 극단적이 되지 마라

let 을 절대 사용하지 말라는 의미가 아니다.
디버그하는데 어려울 수 있고, 이해하기 어려울 수 있지만 이 비용을 지불할 만한 가치가 있는 경우가 있다.

#### Safe Call

nullable 한 가변 프로퍼티가 있고, null 이 아닐 경우에만 어떤 작업을 한다고 하자.
다음과 같이 safe call let 을 사용할 수 있다.
이런 관용구는 널리 사용되며, 많은 사람들이 쉽게 인식할 수 있다.

```kotlin
class Person(val name: String)
var person : Person? = null

fun printName() {
    person?.let {
        print(it.name)
    }
}
```

#### 연산을 뒤로 이동

연산을 argument 처리 후로 이동시킬 때도 활용할 수 있다.
print(students.filter{...}.joinToString{...}) 코드에서, print 연산을 뒤로 이동시켰다.

```kotlin
students
  .filter { it.result >= 50}
  .joinToString(separator = "\n") {
      "${it.name} ${it.surname}, ${it.result}"
  }
  .let(::print)
```

#### 데코레이터

데코레이터로 객체를 wrap 할 때 사용할 수 도 있다.

```kotlin
val obj = FileInputStream("/file.gz")
  .let(::BufferedInputStream)
  .let(::ZipInputStream)
  .let(::ObjectInputStram)
  .readObject() as SomeObject
```

## 연산자 오버로드할 때는 의미에 맞게 사용해라

연산자 의미가 명확히지 않으면, 연산자 오버로딩을 사용하지 말자.
팩토리얼을 구하는 확장 함수를 작성해보자.

```kotlin
fun Int.factorial(): Int = (1..this).product()
fun Iterable<Int>.product(): Int = fold(1) { acc, i -> acc * i }

print(10 * 6.factorial())
```

팩토리얼 기호 그대로, 아래처럼 사용할 수 도 있다.

```kotlin
operator fun Int.not() = factorial()

fun Int.factorial(): Int = (1..this).product()
fun Iterable<Int>.product(): Int = fold(1) { acc, i -> acc * i }

print(10 * !6)
```

이렇게 사용해도 될까? 안된다.
함수의 이름이 not 이기 때문에, 논리 연산에 사용해야지 팩토리얼 연산에 사용하며 안된다.
코드를 위와 같이 작성하면 오해의 소지가 있다.

코틀린의 모든 연산자는 구체적인 이름을 가진 함수에 대한 별칭일 뿐이다.
관례를 따라, 연산자를 사용해야한다.

- a==b: a.equals(b)
- a>b: a.compareTo(b) > 0
- a in b: b.contains(a)
- a*b: a.times(b)

### 분명하지 않은 경우

관례를 충족하는지 아닌지 확실하지 않을 때가 있다. 이럴 때는,

1. infix 를 활용해서 확장함수를 정의해서 사용할 수 있다.
2. top-level function 을 사용하는 것도 방법이다.

예를 들어, 함수를 세 배하다는 것 (* 연산자) 는 무슨 의미일까 ?

함수를 세 번 반복하는 새로운 함수를 만들어 낸다고 생각할 수 있다.

```kotlin
operator fun Int.times(operation: () -> Unit): () -> Unit =
  { repeat(this) { operation() } }

val tripleJko: () -> Unit = 3 * { println("jko") }
tripleJko()
```

함수를 세 번 호출한다는 것으로 이해할 수도 있다.

```kotlin
private operator fun Int.times(operation: () -> Unit) =
  repeat(this) { operation() }

3 * { println("jko") }
```

이렇게 의미가 명확하지 않으면, infix 를 활용해서 확장함수를 정의해서 사용할 수 있다.

```kotlin
private infix fun Int.timesRepeated(operation: () -> Unit): () -> Unit =
  { repeat(this) { operation() } }

val tripleJko = 3 timesRepeated { print("jko") }
tripleJko()
```

top-level function 을 사용하는 것도 방법이다.

```kotlin
repeat(3) { println("jko")}
```

## Unit? 을 리턴하지 마라

Unit? 을 리턴하면 예측하기 어려운 코드를 만들 수 있다.
Boolean 을 리턴하도록 하자.

```kotlin
// A
fun keyIsCorrect(key: String): Boolean = // ...
if (!keyIsCorrect(key)) return

// B
fun verifyKey(key: String): Unit? = // ...
verifyKey(key) ?: return
```

## 변수 타입이 명확하지 않으면 확실하게 지정해라

코틀린의 타입 추론 시스템은, 유형이 명확하지 않으면 남용하지 말아야한다.
A 는 타입을 숨기고 있지만, B 는 명시적으로 타입을 지정함으로써 코드를 더 쉽게 읽을 수 있다.

```kotlin
// A
val data = getSomeData()

// B
val data: UserData = getSomeData()
```

## 리시버를 명시적으로 참조해라

짧게 적을 수 있다는 이유만으로, 리시버를 제거하지 말자.
여러 개의 리시버가 있으면, 리시버를 명시적으로 적어줘야한다.
그러면, 어떤 리시버의 함수인지를 명확하게 알 수 있어서 가독성이 향상된다.

예를 들어 apply, with, run 함수를 사용할 때가 있다.

```kotlin
class Node(val name: String) {
    fun makeChild(childName: String) =
        create("$name.$childName")
            .also { print("Created ${it?.name}")} // Created parent.child

    fun create(name: String): Node? = Node(name)
}

val node = Node("parent")
node.makeChild("child")
```

## 프로퍼티는 동작이 아니라, 상태를 나타내야한다

프로퍼티는 개념적으로,

- val 의 경우에 getter
- var 의 경우에 getter 와 setter 를 나타낸다.

그래서 프로퍼티를 정의하여 오버라이드 가능하다.

```kotlin
open calss Computer {
    open val answer: Long = 42
}

class AppleComputer : SuperComputer() {
    override val answer: Long = 99
}
```

프로퍼티는 접근자를 나타내기 때문에, 함수 대신 사용할 수 있다.
하지만, 완전히 대체해서 사용하지 말아야한다.

```kotlin
val Tree<Int>.sum: Int
    get() = when (this) {
        is Leaf -> value
        is Node -> left.sum + right.sum
    }
```

위에서의 sum 프로퍼티는 모든 요소를 반복 처리하므로, 알고리즘 동작이라고 할 수 있다.
큰 컬렉션의 경우 많은 계산량이 필요할 수 있다.
그런데, 이런 getter 에는 많은 계산량이 필요하다고 예상하지 않는다.
그래서, 이런 처리는 아래와 같이 프로퍼티가 아니라 함수로 구현해야한다.

```kotlin
fun Tree<Int>.sum(): Int = when (this) {
    is Leaf -> value
    is Node -> left.sum() + right.sum()
}
```

프로퍼티는 상태를 추출, 설정할 때 사용하자.
그러면 언제 프로퍼티 대신 함수를 사용해야할까 ?

1. 연산 비용이 높거나, 복잡도가 O(1) 보다 큰 경우
2. 비즈니스 로직을 포함한 경우
3. 결정적이지 않은 경우: 같은 동작을 연속 두 번 했는데 다른 값이 나올 수 있으면 함수를 사용하자.
4. 변환의 경우: 변환은 관습적으로 Int.toDouble() 같은 변환 함수를 사용한다.
5. getter 에서 프로퍼티 상태 변경이 일어나는 경우

---

이펙티브 코틀린 <마르친 모스칼라>
