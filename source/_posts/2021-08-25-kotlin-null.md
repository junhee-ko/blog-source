---
layout: post 
title:  "Null"
date:   2021-08-25 
categories: Kotlin
---

코틀린에서 null 을 어떻게 처리하는지 정리한다.

## nullability

nullability (널 가능성) 은, NPE 를 피할 수 있도록 돕기 위한 코틀린 타입 시스템의 특성이다.
널이 될 수 있는지의 여부를 타입 시스템에 추가해서, 
컴파일러가 오류를 미리 감지해 실행 시점에 발생할 수 있는 가능성을 줄인다.

## 널이 될 수 있는 타입

다음 자바 함수를 보자.

```java
int stringLength(String s) {
    return s.length();
}
```

이 함수의 파라미터로 null 을 전달하면, NPE 가 발생한다.
이 함수를 코틀린으로 작성하면,

```kotlin
fun stringLength(s: String) = s.length
```

이 함수의 파라미터로 null 을 전달하면, 컴파일 시 오류가 발생한다.
그래서, 실행 시점에 NPE 가 발생하지 않는다고 확신할 수 있다.

만약 위 함수가 널과 문자열을 받을 수 있게 하려면, ? 를 명시해야한다.

```kotlin
fun stringLength(s: String?) = ...
```

## 연산 제한

널이 될 수 있는 타입의 변수이면, 수행할 수 있는 연산이 제한된다.
예를 들어, "변수.메서드()" 처럼 메서드를 직접 호출하면 컴파일 에러가 발생한다.

```kotlin
fun stringLength(s: String?) = s.length
```

아래 처럼, null 검사를 추가하면 컴파일에 성공한다.

```kotlin
fun stringLength(s: String?): Int {
    if (s != null) {
        return s.length
    } else {
        return 0
    }
}
```

그런데, 이렇게 널 가능성을 체크하기 위해 if 를 통해 검사를 해야한다면, 코드가 번잡해질 것이다.
그래서 코를린은 널을 다룰 수 있는 여러 도구를 제공한다.

## ?.

?. 는 널 검사와 메서드 호출을 한 번의 연산으로 수행한다.
아래의 코드는,

```kotlin
if (s ! = null) {
    s.toUpperCase()
} else {
    null
}
```

이것과 같다.

```kotlin
s?.toUpperCase()
```

주의할 점은, ?. 의 결과 타입도 널이 될 수 있다.
다음을 보자.

```kotlin
fun printAllCaps(s: String?) {
    val allCaps: String? = s?.toUpperCase()
    println(allCaps)
}
```

allCaps 는 null 일 수 있다.

## ?:

널 대신 사용할 디폴트 값을 지정할 때, 엘비스 연산자 (?:) 를 사용하면 된다.

```kotlin
fun foo(s: String?) {
    val t: String = s ?: ""
}
```

s 가 null 이면 결과는 "" 이다.

다른 예제를 보자.

```kotlin
class Address(val street: String, val zipCode: Int, val city: String)
class Company(val name: String, val address: Address?)
class Person(val name: String, val company: Company?)

fun printShippingLabel(person: Person) {
    val address = person.company?.address
        ?: throw IllegalArgumentException("No Address")

    with(address) {
        println(street)
        println("$zipCode $city")
    }
}
```

address 가 null 이면 예외를 던진다.

## as?

as? 연산자는 어떤 값을 지정한 타입으로 cast 한다. 
cast 할 수 없으면 null 을 반환한다.

아래 코드를 보자.

```kotlin
class Person(val firstName: String, val lastName: String){

    override fun equals(o: Any?): Boolean {
        val otherPerson = o as? Person ?: return false

        // 안전한 캐스트 이후엔, otherPerson 이 Person 타입으로 스마트 캐스트됨
        return otherPerson.firstName == firstName &&
                otherPerson.lastName == lastName 
    }

    override fun hashCode(): Int {
        var result = firstName.hashCode()
        result = 31 * result + lastName.hashCode()
        return result
    }
}
```

equals 메서드에서, 타입이 서로 일치하지 않으면 false 를 반환한다. 

## !!

!! (not-null assertion) 는, 어떤 값이든 널이 될 수 없는 타입으로 바꾼다.

```kotlin
fun ignoreNulls(s: String?) {
    val notNullString = s!!
    println(notNullString.length)
}
```

만약에 s 가 null 이면, 예외가 발생한다.
주의할 점은,

1. notNullString.length 에서 발생하는 것이 아니라, val notNullString = s!! 에서 발생한다.
2. !! 를 한 줄에 쓰지 말자. 예외 stack trace 에서 파일의 몇 번째 줄인지는 알 수 있지만, 어떤 식에서 예외가 발생했는지는 알 수 없기 때문이다.

```kotlin
perosn.company!!.address!!.country // 이렇게 한줄에 쓰지 말자
```

## let

다음 함수를 보자.

```kotlin
fun sendEmailTo(email: String){}
```

위 함수에는 널이 될 수 있는 타입의 값을 넘길 수 없다.
그래서 아래와 같이, 파라미터를 넘기기 전에 값이 널인지 검사해야한다.

```kotlin
val email: String? = ...
if (email != null) {
    sendEmailTo(email)
}
```

이럴 때 let 함수를 사용할 수 있다.
let 함수는 자신의 수신 객체를 인자로 전달받은 람다에 넘긴다.
아래와 같이 사용할 수 있다.

```kotlin
email?.let{email -> sendEmailTo(email)}
```

it 을 사용하면 더 줄일 수 있다.

```kotlin
email?.let{ sendEmailTo(it)}
```

## lateinit 변경자

코틀린에서는
1. 일반적으로 생성자에서 모든 프로퍼티를 초기화해야한다.
2. 프로퍼티 타입이 널이 될 수 없는 타입이라면, 반드시 널이 아닌 값으로 초기화해야한다.

다음 코드를 보자.

```kotlin
class MyService {
    fun action(): String = "foo"
}

class MyTest {
    private var myService: MyService? = null

    @Before
    fun setUp() {
        myService = MyService()
    }

    @Test
    fun testAction() {
        Assert.assertEquals("foo", myService!!.action())
    }
}
```

myService 를 null 로 초기화하기 위해, 널이 될 수 있는 타입으로 선언했다.
testAction() 에서는 널 가능성을 신경써서 !! 를 사용했다.
myService 를 계속 사용해야한다면, !! 를 계속 사용해야한다.

MyTest 클래스를 개선해보자.
프로퍼티에 lateinit 변경자를 붙이면, 프로퍼티를 나중에 초기화할 수 있도록 만들 수 있다. 

```kotlin
class MyTest {
    private lateinit var myService: MyService

    @Before
    fun setUp() {
        myService = MyService()
    }

    @Test
    fun testAction() {
        Assert.assertEquals("foo", myService.action())
    }
}
```

myService 프로퍼티를 초기화하지 않고 널이 될 수 없는 프로퍼티로 선언했다.
testAction() 에서는 더 이상 !! 를 사용하지 않는다.

val 프로퍼티는 final 필드로 컴파일되며, 생성자 안에서 반드시 초기화해야한다.
그래서, 생성자 밖에서 초기화하는 나중에 초기화하는 프로퍼티는 항상 var 이어야한다.

## 널이 될 수 있는 타입 확장

null 이 될 수 있는 수신 객체에 대해 확장 함수를 호출할 수 있다.
String? 타입의 수신 객체애 대해 호출 가능한 isNullOrEmpty 와 isNullOrBlank 메서드가 있다.

다음 코드를 보자.

```kotlin
fun verifyInput(input: String?) {
    if (input.isNullOrBlank()) {
        println("null or blank")
    }
}
```

여기서, input 은 널이 될 수 있는 타입의 값이다.
여기서, isNullOrBlank 은 널이 될 수 있는 타입의 확장 함수이다.

isNullOrBlank 함수를 자세히 봐보자.

```kotlin
public inline fun CharSequence?.isNullOrBlank(): Boolean {
    contract {
        returns(false) implies (this@isNullOrBlank != null)
    }

    return this == null || this.isBlank()
}
```

this 가 널이 될 수 있다는 것을 알 수 있다.
코틀린에서는, 널이 될 수 있는 타입의 확장 함수 안에서는 this 가 널이 될 수 있다.
자바에서는, 메서드 안의 this 는 메스드가 호출된 수신 객체이므로 널이 될 수 없다.


## 플랫폼 타입

플랫폼 타입은 코틀린이 널 관련 정보를 알 수 없는 타입이다.
널이 될 수 있는 타입으로 처리해도 되고, 널이 될 수 없는 타입으로 처리해도 된다.
코틀린에서 플랫폼 타입을 선언할 수 없고, 자바 코드에서 가져온 타입만 플랫폼 타입이 될 수 있다.

다음 자바 코드를 보자.

```kotlin
public class MyPerson {
    private final String name;

    public MyPerson(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

getName() 은 null 을 리턴할까 안할까 ?
코틀린 컴파일러는 이 경우, String 타입의 널 가능성에 대해 모른다.
그래서, 우리가 직접 처리해야한다.
만약, 아래의 yellAt 함수에 null 을 인자로 전달한다면, 예외가 발생한다.

```kotlin
fun yellAt(person: MyPerson){
    println(person.name.uppercase())
}
```

플랫폼 타입은 아래 두 선언이 모두 가능하다.

```kotlin
val s1: String ? = person.name
val s2: String  = person.name
```

자바에서 가져온 널 값을 널이 될 수 없는 코틀린 변수에 대입하면, 실행 시점에 대입 될 때 예외가 발생한다.
자바 API 를 다룰 때는 문서를 잘 보고 사용하자.

---
Kotlin In Action <드미트리 제메로프, 스베트라나 이사코바>
