---
layout: post
title: Effective Kotlin - Reusability 2
date: 2022-04-20
categories: Kotlin
---

코틀린의 재사용성을 활용하는 방법을 정리한다.

- 일반적인 알고리즘을 구현할 때, 제네릭을 사용해라
- 타입 파라미터의 shadowing 을 피해라
- 공통 모듈을 추출해서 여러 플랫폼에서 재사용해라

## 일반적인 알고리즘을 구현할 때, 제네릭을 사용해라

type parameter 를 가지는 함수를 generic function 이라고 한다.
예를 들어, type parameter T 를 가지는 filter function 이 있다.

```kotlin
public inline fun <T> Iterable<T>.filter(predicate: (T) -> Boolean): List<T> {
  return filterTo(ArrayList<T>(), predicate)
}
```

타입 파라미터는 구체적인 타입의 서브 타입만 사용하도록 타입을 제한할 수 있다.
타입에 제한이 걸려서, 내부에서 해당 타입이 제공하는 메서드를 사용할 수 있다.

아래를 보자.
Number 를 타입 파라미터 T 의 상한으로 지정한다.
그리고, 내부에서 Number 클래스에 정의되 메서드를 호출한다.

```kotlin
fun <T: Number> oneHalf(value: T): Double {
    return value.toDouble() / 2.0
}

oneHalf(3) // 실제 타입 인자인 Int 가 Number 를 확장하기 때문에 가능
```

## 타입 파라미터의 shadowing 을 피해라

지역 파라미터가 외부 scope 에 있는 프로퍼티를 가리는 것을 shadowing 이라고 한다.
예를 들어,

```kotlin
class Forest(val name: String){

    fun addTree(name: String){
        // ...
    }
}
```

클래스 타입 파라미터와 함수 타입 파라미터 사이에도 발생 가능하다.
그래서 아래처럼, Forest 와 addTree 의 타입 파라미터가 독립적으로 동작할 수 있는 문제가 생길 수 있다.

```kotlin
interface Tree
class A: Tree
class B: Tree

class Forest<T: Tree> {

    fun <T: Tree> addTree(tree: T){
        // ...
    }
}

val forest = Forest<A>()
forest.addTree(A())
forest.addTree(B()) // Oops !
```

이런 문제를 해결하기 위해, addTree 가 클래스 타입 파라미터인 T 를 그대로 사용해야한다.

```kotlin
interface Tree
class A: Tree
class B: Tree

class Forest<T: Tree> {

    fun addTree(tree: T){
        // ...
    }
}

val forest = Forest<A>()
forest.addTree(A())
forest.addTree(B()) // Type mismatch
```

## 공통 모듈을 추출해서 여러 플랫폼에서 재사용해라

코틀린을 사용하면 다양한 플랫폼을 대상으로 개발할 수 있다.
그래서, 코틀린 공통 코드를 작성해서 여러 플랫폼에서 재사용할 수 있다.
예를 들어, 아래 각각의 플랫폼에서 코틀린 공통 코드를 사용할 수 있다.

- 서버(코틀린/JVM)
- 브라우저 (코틀린/JS)
- 안드로이드 (코틀린/JVM)
- 스위프트 or 코틀린/네이티브

---

이펙티브 코틀린 <마르친 모스칼라>
