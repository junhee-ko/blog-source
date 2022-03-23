---
layout: post
title: Effective Kotlin - Safety 2
date: 2022-03-23
categories: Kotlin
---

코틀린을 안전하게 사용하기 위한 방법을 정리한다.

- 사용자 정의 오류보다 표준 오류를 사용해라
- 결과 부족이 발생하면, null 과 Failure 를 사용해라
- 적절하게 null 을 처리해라
- use 를 사용해서 리소스를 닫아라
- 단위 테스트를 만들어라

## 사용자 정의 오류보다 표준 오류를 사용해라

표준 라이브러리의 오류는 많은 개발자가 알고 있기 때문에, 이를 재사용하는 것이 좋다.
다른 사람들이 API 를 더 쉽게 배우고 이해할 수 있다. 예를 들어,
- IllegalArgumentException,
- NoSuchElementException
- UnsupportedOperationException
- ...

## 결과 부족이 발생하면, null 과 Failure 를 사용해라

함수가 원하는 결과를 만들지 못할 때가 있다. 예를 들어,

1. 인터넷 연결 문제로, 서버로부터 데이터를 읽어 들이지 못할 때
2. 조건에 맞는 첫 번째 요소가 없을 때
3. 텍스트를 파싱해서 객체를 만들려고 했는데, 텍스트 형식이 맞지 않을 때

이를 처리하는 방법에는,

1. null 을 리턴한다.
2. 실패를 나타내는 sealed 클래스를 리턴한다. (일반적으로, Failure 라는 이름)
3. 예외를 throw 한다.

예외를 throw 하는 방식의 단점은,

1. 코틀린의 모든 예외는 unchecked exception 이여서, 예외를 처리하지 않아 애플리케이션의 흐름을 중지시킬 수 있다.
2. 예외는 예외적인 상황 처리를 위해 만들어져서, 명시적인 테스트만큼 빠르게 동작하지 않는다.
3. try-catch 블록 내부에 코드를 배치하면, 컴파일러가 할 수 있는 최적화가 제한된다.

반면에, null 이나 Failure 를 리턴하는 방법은 명시적으로 처리해야한다.
또한, 애플리케이션의 흐름을 중지시키지도 않는다.

따라서, 예측할 수 있는 범위의 오류는 null 이나 Failure 를 사용해라.
예측하기 어려운 범위의 오류는 예외를 throw 해라.

아래 예를 보자.

```kotlin
sealed class Result<out T>
class Success<out T>(val result: T) : Result<T>()
class Failure(val throwable: Throwable) : Result<Nothing>()
class JsonParsingException : Exception()

inline fun <reified T> String.readObjectOrNull(): T? {
    if (incorrectSign) {
      return null
    }
    return result
}

inline fun <reified T> String.readObject(): T? {
    if (incorrectSign) {
      return Failure(JsonParsingException())
    }
    return Success(result)
}
```

이러한 오류는, 다루기 쉽고 놓치기 어렵다.
safe call 이나 Elvis 연산자 같은 null-safety 기능을 활용할 수 있다.

```kotlin
val age = userText.readObjectOrNull<Person>()?.age ?: -1
```

그럼, 언제 null 을 사용하고 언제 sealed result 클래스를 사용해야할까 ?
- 추가적인 정보를 전달해야한다면 sealed result 클래스 를,
- 그렇지 않다면 null 을 사용해라.

## 적절하게 null 을 처리해라

nullable type 은 세 가지 방법으로 처리할 수 있다.

1. ?., smart casting, Elvis 연산자 를 활용
2. 오류 throw
3. nullable type 이 나오지 않도록 함수 or 프로퍼티를 refactoring


### ?., smart casting, Elvis 연산자

```kotlin
// safe call
printer?.print()

// smart casting
if (printer != null) printer.print()

// Elvis
val printerName = printer?.name ?: "Unnamed"
```

### 오류 throw

개발자가 "당연히 그럴 것이다" 라고 생각할 수 있는 부분이 있고,
이 부분에서 문제가 발생하면 개발자에게 오류를 강제로 발생시켜주자.
requireNotNull, checkNotNull, throw, !! 등을 사용할 수 있다.

```kotlin
fun process(user: User) {
    requireNotNull(user.name)
    checkNotNull(context)
    val network = getNetwork(context) ?: throw NoInternetConnection()
    network.getData { data, userData -> show(data!!, userData!!) }
}
```

### !! (not-null assertion) 은 피해라

nullable 을 처리할 때 가장 간단한 방법은 not-null assertion 을 사용하는 것이다.
!! 는 타입이 nullable 이지만, null 이 나오지 않는다고 확신할 때 사용된다.
하지만, 현재 확실하다고 미래에도 확실한 것은 아니다.
!! 를 사용하면 자바에서 nullable 을 처리할 때 발생할 수 있는 문제가 그대로 발샐한다.

### 의미없는 nullability 는 피해라

nullability 는 어떻게든 적절하게 처리해야해서, 추가 비용이 발생한다.
따라서 필요한 경우가 아니라면, nullability 를 피하자.

nullability 를 피할 수 있는 몇 가지 방법으로,

1. 클래스에서 nullability 에 따라 여러 함수를 제공할 수 있다. (ex) List<T> 의 get(), getOrNull()
2. 어떤 값이 클래스 생성 이후에 설정된다는 보장이 있으면, "lateinit property" or "notNull delegate" 를 사용해라.
3. collection 의 element 가 부족하다는 것을 나타내려면, empty collection 을 사용해라.

### lateinit property

lateinit 은 프로퍼티를 처음 사용하기 전에, 반드시 초기화될 것이라고 예상될 때 사용된다.
메서드 호출에 명확한 순서가 있을 경우에 사용될 수 있다.
다음을 보자.

```kotlin
class Test {
    private var dao: UserDao? = null
    private var controller: UserController? = null

    @BeforeEach
    fun init() {
        dao = mockk()
        controller = UserController(dao!!)
    }
    @Test
    fun test(){
      controller!!.doSth()
    }
}
```

controller 를 nullable 에서 null 이 아닌 것으로 타입 변환하고 있다.
테스트 전에 설정될 것이 명확하기 때문에, 의미 없는 코드라고 할 수 있다.
그래서 다음처럼 해결함으로써, !! 연산자로 unpack 하지 않아도 된다.

```kotlin
class Test {
    private lateinit var dao: UserDao
    private lateinit var controller: UserController

    @BeforeEach
    fun init() {
        dao = mockk()
        controller = UserController(dao!!)
    }
    @Test
    fun test(){
      controller.doSth()
    }
}
```

### notNull delegate

JVM 에서 Int, Long, Double, Boolean 같은 기본 타입으로 초기화해야하는 경우에
Delegates.notNull 을 사용할 수 있다.

```kotlin
class DoctorActivity: Activity() {
    private var doctorId: Int by Delegates.notNull()
    private var fromNotification: Boolean by Delegates.notNull()

    override fun onCreate(...){
        ...
        doctorId = intent.extras.getInt(DOCTOR_ID_ARG)
        fromNotification = intent.extras.getBoolean(FROM_NOTIFICATION_ARG)
    }

}
```

## use 를 사용해서 리소스를 닫아라

더 이상 필요하지 않을 때 close 메서드로 명시적으로 닫아야하는 리소스가 있다.
예를 들어,

1. InputStream, OutputStream
2. java.sql.Connection
3. java.io.Reader

이 리소스들은 AutoCloseable 를 상속받는 Closeable 인터페이스를 구현한다.
리소스는 최종적으로 리소스에 대한 레퍼런스가 없어질 때, garbage collector 에 의해 처리된다.
garbage collector 에 의해 처리되는 것의 단점은,

1. 굉장히 느리다.
2. 쉽게 처리되지 않는다.
3. 그동안 리소스를 유지하는 비용이 많이 든다.

그래서, 아래 처럼 명시적으로 close 를 호출하는 것이 좋다.

```kotlin
val reader = BufferedReader(FileReader(path))
try {
    return reader.lineSequence().sumBy { it.length }
} finally {
    reader.close()
}
```

하지만 이런 코드의 단점은,

1. 굉장히 복잡하다.
2. 리소스를 닫을 때 예외가 발생 가능한데, 이를 따로 처리하지 않는다.
3. try block 과 finally block 내부에서 오류가 발생하면, 둘 중 하나만 전파된다.

표준 라이브러리의 use 를 사용하면 위 단점을 해결할 수 있다.
이러한 코드는 Closeable 객체에 사용할 수 있다.

```kotlin
val reader = BufferedReader(FileReader(path))
reader.use {
    return reader.lineSequence().sumBy { it.length }
}
```

## 단위 테스트를 만들어라

안정적인 프로그램을 만들기 위해 가장 중요한 것은, 올바르게 동작하는지 확인하는 테스트이다.
테스트 중에 가장 효츌적으로 활용할 수 있는 테스트가 단위 테스트이다.

단위 테스트를 다음 내용들을 확인한다.

1. 일반적인 유스 케이스 (happy path): 요소가 사용될거라 예상되는 일반적인 방법을 테스트
2. 일반적인 오류 케이스와 잠재적인 문제: 제대로 동작하지 않을거라고 예상되는 일반적인 부분, 과거에 문제가 발생했던 부분을 테스트
3. edge 케이스와 잘못된 arguments: Int 의 경우 Int.MAX_VALUE 를 사용하는 경우, 양의 정수로만구할 수 있는데 음위 정수를 넣는 경우

단위 테스트의 장점으로는,

1. 테스트가 잘된 요소는 신뢰할 수 있다.
2. 테스트가 잘 만들어져있으면 refactoring 에 두렵지 않다.
3. 수동으로 테스트하는 것보다 빠르다.

---

이펙티브 코틀린 <마르친 모스칼라>
