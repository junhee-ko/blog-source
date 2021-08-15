---
layout: post 
title:  "람다"
date:   2021-08-15 
categories: Kotlin
---

코틀린의 람다에 대해 정리한다.

## 람다 소개

아래와 같은 내용을 코드로 적용하려면 어떻게 해야할까 ?

- 이벤트가 발생하면 이 Handler 를 실행하자
- 모든 원소에 이 연산을 적용하자

자바에서는 무명 내부 클래스를 사용해, 클래스를 선언하고 그 클래스의 인스턴스를 함수에 넘길 수 있다.
함수형 프로그래밍에서는 함수를 직접 다른 함수에 전달할 수 있다.

자바의 다음 예를 보자.

```java
button.setOnClickListener(new onClickListener(){
    @Override
    public void onClick(View view){
        // event when clicked
    }
});
```

무명 내부 클래스를 선언해서, 코드가 번잡하다. 
코틀린에서는, 자바 8 과 마찬가지로 람다를 사용할 수 있다.

```kotlin
button.setOnClickListener { /* event when clicked */ }
```

## 람다와 컬렉션

이름과 나이를 저장하는 Person 클래스가 있다.

```kotlin
data class Person(val name: String, val age: Int)
```

사람 리스트에서, 가장 연장자를 찾아보자.

```kotlin
fun findTheOldest(people: List<Person>) {
    var maxAge = 0
    var theOldest: Person? = null

    for (person in people) {
        if (person.age > maxAge) {
            maxAge = person.age
            theOldest = person
        }
    }

    println(theOldest)
}

fun main() {
    val people = listOf(Person("jko", 12), Person("junhee-ko", 30))
    findTheOldest(people)
}
```

코틀린에서는, 라이브러리를 사용하면 아래와 같이 쉽게 구현이 가능하다.

```kotlin
println(people.maxByOrNull { it.age })
```

단지 함수나 프로퍼티를 반환하는 람다는, 멤버 참조로 대치 가능하다.

```kotlin
println(people.maxByOrNull(Person::age))
```

## 람다 식의 문법

람다 식을 선언하기 위한 문법은 아래와 같다.

![](/image/lamda-grammar.png)

람다 식은 변수에 저장할 수 있다.

```kotlin
val sum = { x: Int, y: Int -> x + y }
println(sum(1, 2))
```

다음 예를 다시 보자.

```kotlin
val people = listOf(Person("jko", 12), Person("junhee-ko", 30))
println(people.maxByOrNull { it.age })
```

위 코드를 정식으로 다시 작성하면,

```kotlin
people.maxByOrNull({ p: Person -> p.age })
```

이 코드를 차례대로 개선해보자. 
우선, 함수 호출 시 맨 뒤에 있는 파라피터가 람다 식이면 그 람다를 괄호 밖으로 추출 가능하다.

```kotlin
people.maxByOrNull() { p: Person -> p.age }
```

위 코드처럼, 람다가 어떤 함수의 유일한 인자이고 괄호 뒤에 람다를 썼다면 빈 괄호를 없앨 수 있다.

```kotlin
people.maxByOrNull { p: Person -> p.age }
```

그리고, 컴파일러는 람다 파라피터의 타입도 추론 가능하다. 그래서,

```kotlin
people.maxByOrNull { p -> p.age }
```

마지막으로, 람다의 파라미터가 하나 뿐이고 컴파일러가 타입을 추론할 수 있으면 it 을 사용할 수 있다.

```kotlin
people.maxByOrNull { it.age }
```

만약, 본문이 여러 줄로 이뤄진 경우 본문의 맨 마지막에 있는 식이 람다의 결과 값이다.

```kotlin
val sum = { x: Int, y: Int ->
    println("Ing sum...")
    x + y
}

println(sum(1, 2))
```

## 현재 영역에 있는 변수 접근

람다를 함수 안에서 사용하면,

1. 함수의 파라미터 뿐만 아니라
2. 람다 정의의 앞에 선언된 로컬 변수까지

람다에서 사용가능하다.

다음 예를 보면, 람다 안에서 함수의 prefix 파라미터를 사용하고 있다.

```kotlin
fun printMessageWithPrefix(messages: Collection<String>, prefix: String) {
    messages.forEach {
        println("$prefix $it") // HERE !!
    }
}

fun main() {
    val errors = listOf("403 Forbidden", "404 Not Found")
    printMessageWithPrefix(errors, "Error :")
}
```

자바와 다른 중요한 점은, 코틀린의 람다 안에서는 final 변수가 아닌 변수에도 접근할 수 있다는 것이다.
다음 예를 보자.

```kotlin
fun printProblemCounts(responses: Collection<String>) {
    var clientErrors = 0
    var serverErrors = 0

    responses.forEach {
        if (it.startsWith("4")) {
            clientErrors++
        } else if (it.startsWith("5")) {
            serverErrors++
        }
    }

    println("client errors : $clientErrors, server erros : $serverErrors")
}
```

위의 코드처럼, 
람다 안에서 사용하는 외부 변수 (clientErrors, serverErrors) 를 람다가 포획 (capture) 한 변수라고 한다.

## 멤버 참조

람다를 사용해서 코드 블록을 다른 함수에 넘기는 방법은 정리했다. 
그런데, 함수를 직접 넘기는 방법은 없을까 ? 
함수를 값으로 변경해서 넘기면 된다.

```kotlin
val getAge = Person::age
```

:: 를 사용하는 식이, 멤버 참조 (Member Reference) 이다. 
멤버 참조는 프로퍼티나 메서드를 단 하나만 호출하는 함수 값을 만들어준다.

### 최상위에 선언된 함수, 프로퍼티

최상위에 선언된 (다른 클래스의 멤버가 아닌) 함수나 프로퍼티 참조도 가능하다.

```kotlin
fun hello() = println("hello")
run(::hello)
```

위 코드를 보면, 클래스 이름을 생략했다. 
hello 라는 멤버 참조를 run 라이브러리 함수에 넘긴다. 
run 은 인자로 받은 람다를 호출하는 역할을 한다.

### 생성자 참조

생성자 참조를 사용하면 클래스 생성 작업을 연기하거나 저장 가능하다.

```kotlin
data class Person(val name: String, val age: Int)

val createPerson = ::Person
val person = createPerson("junhee", 30)
```

### 확장 함수

확장 함수도 멤버 함수와 동일한 방식으로 참조 가능하다.

```kotlin
fun Person.isAdult() = age > 21
val predicate = Person::isAdult
```

## 자바 메서드에 람다를 인자로 전달

함수형 인터페이스를 인자로 원하는 자바 메서드에 코틀린 람다를 전달할 수 있다.

```java
void postphoneComputation(int delay,Runnable computation)
```

위 자바 메서드에, 코틀린에서는 람다를 넘길 수 있다.

```kotlin
postphoneComputation(1000) { println(42) }
```

코틀린에서는, 람다를 Runnable 인스턴스로 변환해준다. 
즉, Runnable 을 구현한 무명 클래스의 인스턴스를 만들어준다.

Runnable 을 구현하는 무명 객체를 명시적으로 만들어서 사용할 수도 있다.

```kotlin
postphoneComputation(1000, object : Runnable {
    override fun run() {
        println(42)
    }
})
```

객체를 명시적으로 선언하는 경우, 메서드가 호출될 때마다 객체가 생성된다.
람다는, 람다에 대응하는 무명 객체를 메서드가 호출 될 때마다 반복 사용한다.

---
Kotlin In Action <드미트리 제메로프, 스베트라나 이사코바>
