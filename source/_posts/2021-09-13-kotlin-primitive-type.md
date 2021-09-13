---
layout: post 
title:  Primitive Type
date:   2021-09-13 
categories: Kotlin
---

코틀린의 원시 타입을 정리한다.

## 원시 타입 : Int, Boolean ...

자바에서는 원시 타입과 참조 타입을 구분한다.
원시 타입의 변수에는 그 값이 들어가고, 참조 타입의 변수에는 메모리상의 객체 위치가 들어간다.

코틀린에서는 원시 타입과 참조 타입을 구분하지 않는다.
아래 코드에서는, 정수를 표현하기 위해 Int 를 사용한다.

```kotlin
val i: Int = 1
val list: List<Int> = listOf(1, 2, 3)
```

그리고, 코틀린에서는 아래처럼 원시 타입의 값에 대해 메서드 호출이 가능하다. 

```kotlin
fun showProgress(progress: Int){
    val percent = progress.coerceIn(0, 100)
    println("progress : $percent")
}
```

그런데, 원시 타입과 참조 타입 같으면 항상 객체로 표현되는 걸까? 아니다.
실행 시점에 가장 효츌적인 방식으로 표현된다.
예를 들어, 대부분의 경우 코틀린의 Int 타입은 자바 int 타입으로 컴파일 된다.

## 널이 될 수 있는 원시 타입 : Int?, Boolean? ...

자바에서는, null 참조를 참조 타입의 변수에만 대입할 수 있다.
그래서, 코틀린에서 널이 될 수 있는 원시 타입을 사용하면 자바의 래퍼타입으로 컴파일 된다.
아레 예제에서, age 프로퍼티의 값은 java.lang.Integer 로 저장된다.

```kotlin
data class Person(val name: String, val age: Int? = null) {
    
    fun isOrderThan(other: Person): Boolean? {
        if (age == null || other.age == null) {
            return null
        }

        return age > other.age
    }
}
```

## 숫자 변환

코틀린은 한 타입의 숫자를 다른 타입의 숫자로 자동 변환하지 않는다.
다음 코드는 컴파일 에러가 발생한다.

```kotlin
val i = 1
val l : Long = i
```

아래와 같이 직접 변환 메서드를 호출해야한다.

```kotlin
val i = 1
val l : Long = i.toLong()
```

## Any : 최상위 타입

자바에서는 Object 가 클래스 계층의 최상위 타입이다.
코틀린에서는 Any 타입이 모든 널이 될 수 없는 타입의 최상위 타입이다.
널을 포함하는 모든 값을 대입할 변수를 선언하려면 Any? 타입을 사용해라.

코틀린 함수가 Any 를 사용하면 자바 바이트코드의 Object 로 컴파일된다.

모든 코틀린 클래스에는 toString, equals, hashCode 라는 세 메서드가 들어있다.
이 세 메서드는 Any 에 정의된 메서드를 상속한 것이다.

## Unit : 코틀린의 void

코틀린 Unit 타입은 자바 void 와 같은 기능을 한다.
아래 두 함수는 같다.

```kotlin
fun f(): Unit {
}

fun f() {
}
```

그러면, Unit 이 자바의 void 와 다른 점이 무엇일까 ? 
void 와 달리 Unit 을 타입 인자로 사용 가능하다.
다음 예를 보자.

```kotlin
interface Processor<T> {
    fun process(): T
}

class NoResultProcessor : Processor<Unit> {

    override fun process() {
        // 명시적으로 Unit 반환할 필요 없음
        // 컴파일러가 묵시적으로 return Unit 넣어줌
    }
}
```

Processor 인터페이스는 process 함수가 어떤 값을 반환하라고 정의되어 있지만,
Unit 타입도 Unit 값을 제공하기 때문에 메서드에서 Unit 값을 반환해도 된다.

자바에서도 java.lang.Void 타입을 사용하면 "값 없음" 을 표현할 수 있다.
하지만 그래도, Void 타입에 대응하는 유일한 값인 null 을 반환하기 위해, return null 을 해야한다.

## Nothing : 함수 비정상 종료

함수가 정상적으로 끝나지 않는다는 사실을 표현하기 위해, Nothing 이라는 반환 타입이 있다.

테스트 라이브러리들은 fail 이라는 함수를 제공하는 경우가 많다. 
fail 은 특별한 메세지가 들어있는 예외를 던져 현재 테스트를 실패시킨다.
 
```kotlin
fun fail(message: String): Nothing {
    throw IllegalArgumentException(message)
}
```

---
Kotlin In Action <드미트리 제메로프, 스베트라나 이사코바>
