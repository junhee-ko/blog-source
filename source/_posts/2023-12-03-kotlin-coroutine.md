---
layout: post
title: "코루틴의 실제 구현" 
date: 2023-12-03
categories: Kotlin
---

## Continuation Passing Style

중단 함수는 Continuation Passing Style 로 구현된다.
Continuation 은 함수에서 함수로 인자를 통해 전달되며, 마지막 파라미터로 전달된다.

```kotlin
suspend fun getUser(): User?
suspend fun setUser(user: User)

fun getUser(continuation: Continuation<*>): Any?
fun setUser(user: User, continuation: Continuation<*>): Any
```

반환 타입이 달라진 이유는, 중단 함수를 실행하는 도중에 중단되면 선언된 타입의 값을 반환하지 않을 수 있기 때문이다.
getUser 함수는 User? 또는 COROUTINE_SUSPENDED marker 를 반환할 수 있다.

## Continuation, Label

```kotlin
suspend fun myFunction() {
    println("before")
    delay(1000) // suspend function
    println("after")
}
```

위 함수의 signature 는 아래와 같이 추론할 수 있다.

```kotlin
fun myFunction(continuation: Continuation<*>): Any
```

이 함수는 상태를 저장하기 위해 자신만의 continuation 객체가 필요하다.
함수의 본문이 시작될 때, 파라미터인 continuation 을 자신만의 continuation 인 MyFunctionContinuation 으로 wrapping 한다.

```kotlin
val continuation = 
    if (continuation is MyFunctionContinuation) continuation
    else MyFunctionContinuation(continuation)

// more simple
val continuation = continuation as? MyFunctionContinuation 
    ?: MyFunctionContinuation(continuation)
```

다시 함수를 보자.

```kotlin
suspend fun myFunction() {
    println("before")
    delay(1000) // suspend function
    println("after")
}
```

함수가 시작되는 지점은 아래 두 곳이다.

1. 함수의 시작점 (함수가 처음 호출될 때)
2. 중단 이후 재개 지점 (continuation 이 resume 을 호출할 때)

현재 상태를 저장하기 위해 label 이라는 필드를 사용하는데, 

1. 처음 시작될 때 이 값은 0으로 설정되며
2. 중단되기 전에 다음 상태로 설정된다.

```kotlin
fun myFunction(continuation: Continuation<Unit>): Any {
    val continuation = continuation as? MyFunctionContinuation 
        ?: MyFunctionContinuation(continuation)
    
    if (continuation.label == 0) {
        println("before")
        continuation.label = 1
        
        if (delay(1000, continuation) == COROUTINE_SUSPENDED) {
            return COROUTINE_SUSPENDED
        }
    }

    if (continuation.label == 1) {
        println("after")
        
        return Unit
    }
    
    error("Impossible")
}
```

이제, MyFunctionContinuation 를 보자.

```kotlin
class MyFunctionContinuation(
    val completion: Continuation<Unit>
) : Continuation<Unit> {
    
    override val context: CoroutineContext
        get() = completion.context
    
    var label = 0
    var result: Result<Any>? = null
    
    override fun resumeWith(result: Result<Unit>) {
        this.result = result
        
        val res = try {
            val r = myFunction(this)
            if (r == COROUTINE_SUSPENDED) return
            
            Result.success(r as Unit)
        } catch (e: Throwable) {
            Result.failure(e)
        }
        
        completion.resumeWith(res)
    }
}
```

## 상태를 가진 함수

함수가 중단된 후에 다시 사용할 지역 변수나 파라미터와 같은 상태를 가지고 있으면, 함수의 continuation 객체에 상태를 저장해야한다.
다음 함수를 보자.

```kotlin
suspend fun myFunction() {
    println("before")
    var counter = 0
    
    delay(1000) // suspend function
    
    counter++
    println("counter: $counter")
    println("after")
}
```

지역 변수나 파라미터와 같이 함수 내에서 사용되던 값들은 중단되기 직전에 저장되고, 이후 함수가 재개될 때 복구된다.

```kotlin
fun myFunction(continuation: Continuation<Unit>): Any {
    val continuation = continuation as? MyFunctionContinuation 
        ?: MyFunctionContinuation(continuation)
    
    var counter = continuation.counter
    
    if (continuation.label == 0) {
        println("before")
        counter = 0 
        
        continuation.counter = counter
        continuation.label = 1
        
        if (delay(1000, continuation)) == COROUTINE_SUSPENDED) {
            return COROUTINE_SUSPENDED
        }
    }

    if (continuation.label == 1) {
        counter = (counter as Int) + 1
        println("counter: ${counter}")
        println("after")
        
        return Unit
    }
    
    error("Impossible")
}

class MyFunctionContinuation(
    val completion: Continuation<Unit>
) : Continuation<Unit> {

    override val context: CoroutineContext
        get() = completion.context

    var label = 0
    var result: Result<Unit>? = null
    var counter = 0

    override fun resumeWith(result: Result<Unit>) {
        this.result = result

        val res = try {
            val r = myFunction(this)
            if (r == COROUTINE_SUSPENDED) return

            Result.success(r as Unit)
        } catch (e: Throwable) {
            Result.failure(e)
        }

        completion.resumeWith(res)
    }
}
```

## 값을 받아 재개되는 함수

```kotlin
suspend fun myFunction(token: String) {
    println("before")
    
    val userId = getUserId(token)               // suspend point
    println("got userId: ${userId}")
    
    val username = getUsername(userId, token)   // suspend point
    println(User(userId, username))

    println("after")
}
```

1. 함수가 값으로 재개되면 결과는 Result.Success(value) 이고 이 값을 얻어 사용할 수 있다. 
2. 함수가 예외로 재개되면 결과는 Result.Failure(exception) 이고 이 때는 예외를 던진다.

```kotlin
fun myFunction(token: String, continuation: Continuation<*>): Any {
    val continuation = continuation as? MyFunctionContinuation 
        ?: MyFunctionContinuation(continuation, token)

    var result: Result<Any>? = continuation.result
    var userId: String? = continuation.userId
    val username: String

    if (continuation.label == 0) {
        println("before")
        continuation.label = 1

        val res = getUserId(token, continuation)
        if (res == COROUTINE_SUSPENDED) {
            return COROUTINE_SUSPENDED
        }
        
        result = Result.success(res)
    }

    if (continuation.label == 1) {
        userId = result!!.getOrThrow() as String
        println("got userId: ${userId}")

        continuation.label = 2
        continuation.userId = userId
        
        val res = getUsername(userId, token, continuation)
        if (res == COROUTINE_SUSPENDED) {
            return COROUTINE_SUSPENDED
        }
        
        result = Result.success(res)
    }

    if (continuation.label == 2) {
        username = result!!.getOrThrow() as String
        println(User(userId as String, username))
        println("after")
        
        return Unit
    }

    error("Impossible")    
}
```

---

코틀린 코루틴 <마르친 모스카와>
