---
layout: post
title: Effective Kotlin - Readability 2
date: 2022-03-30
categories: Kotlin
---

코틀린을 가독성이 좋게 작성하는 방법을 정리한다.

- 리시버를 명시적으로 참조해라
- 프로퍼티는 동작이 아니라, 상태를 나타내야한다
- named arguments 를 사용해라
- 코딩 컨벤션을 지켜라

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

## named arguments 를 사용해라

arguments 의 의미가 명확하지 않은 경우가 있다.

```kotlin
val text = (1..10).joinToString("|")
```

파라미터가 명확하지 않으면, named argument 를 사용하자.

```kotlin
val separator = "|"
val text = (1..10).joinToString(separator = separator)
```

named arguments 의 장점은,

1. 이름을 기반으로 값이 무엇인지 나타낸다.
2. 파라미터 입력 순서와 상관 없어서 안전하다.

named arguments 는 이럴 때 사용하면 좋다.

1. 프로퍼티가 default arguments 를 가질 경우
2. 같은 타입의 파라미터가 많은 경우: 파라미터에 같은 타입이 있으면 잘못 입력할 때 문제 발견이 어렵다
3. 함수 타입 파라미터
  ```kotlin
  observable.getUsers()
    .subscribeBy(
      onNext = { users: List<User> -> ... },
      onError = {throwable : Throwable -> ... },
      onCompleted = { ... }
    )
  }
  ```

## 코딩 컨벤션을 지켜라

코틀린 코딩 컨벤션은 여기에 잘 정리되어 있다:
https://kotlinlang.org/docs/coding-conventions.html

코딩 컨벤션을 지키면,

1. 어떤 프로젝트를 접해도 이해하기 쉽다.
2. 다른 개발자도 프로젝트 코드를 이해하기 쉽다.
3. 프로젝트의 코드 일부를 다른 코드로 이동하는 것이 쉽다.

컨벤션을 지킬 때 도움이 되는 도구로,

1. IntelliJ formatter
   ![](/image/kotlin-intellij-style-guide.png)

2. https://ktlint.github.io/


---

이펙티브 코틀린 <마르친 모스칼라>
