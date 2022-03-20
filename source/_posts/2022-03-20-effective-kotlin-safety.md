---
layout: post
title: Effective Kotlin - Safety
date: 2022-03-20
categories: Kotlin
---

코틀린을 안전하게 사용하기 위한 방법들을 정리한다.

## 가변성을 제한하라

가변성을 제한할 수 있는 방법에는 다음이 있다.

1. 읽기 전용 프로퍼티 val
2. 가변 컬렉션과 읽기 전용 컬렉션 구분
3. 데이터 클래스의 copy

### 읽기 전용 프로퍼티 val

val 로 선언된 프로퍼티는 일반적인 방법으로는 변하지 않는다.

```kotlin
val a = 10
a = 20 // Val cannot be reassigned
```

하지만, 읽기 전용 프로퍼티가 완전히 변경 불가능한 것은 아니다.
아래 예를 기준으로, list = mutableListOf(1, 2, 3, 4, 5) 와 같이 재할당하는 것이 불가능한 것이다.

```kotlin
val list = mutableListOf(1, 2, 3)
list.add(4)

assertEquals(listOf(1, 2, 3, 4), list) // true
```

읽기 전용 프로퍼티는 다른 프로퍼티를 활용하는 사용자 정의 게터로도 정의 가능하다.
아래 예를 기준으로, var 프로퍼티가 변할 때 값이 변할 수 있다.

```kotlin
var firstName = "junhee"
var lastName = "ko"
val fullName
    get() = "$firstName $lastName"

@Test
fun `읽기 전용 프로퍼티는 다른 프로퍼티를 활용하는 사용자 정의 게터로도 정의 가능`() {
    assertEquals("junhee ko", fullName)

    firstName = "updated junhee"
    assertEquals("updated junhee ko", fullName)
}
```

값을 추출할 때마다 사용자 정의 게터가 호출되서 아래 코드처럼 사용할 수도 있다.

```kotlin
 private fun calculate(from: String): Int {
    println("Calculating $from")
    return 42
  }

val fizz = calculate("fizz")
val buzz
    get() = calculate("buzz")

@Test
fun `값을 호출할 때마다 사용자 정의 게터 호출`() {
    assertEquals(42, fizz)
    assertEquals(42, buzz)
}
```

val 은 스마트 캐스트 활용 가능하다.
full1 은 값을 사용하는 시점의 first 에 따라서 다른 결과가 나올 수 있어서 스마트 캐스트를 할 수 없다.

```kotlin
val first: String? = "lets"
val second: String? = "go"
val full1: String?
    get() = first?.let { "$it $second" }
val full2: String? = first?.let { "$it $second" }

@Test
fun `val 은 스마트 캐스트 활용 가능`() {
    if(full1 != null){
        println(full1.length) // Smart cast to 'String' is impossible, because 'full1' is a property that has open or custom getter
    }

    if (full2 != null) {
      println(full2.length)
    }
}
```

### 가변 컬렉션과 읽기 전용 컬렉션 구분

코틀린은, 읽고 쓸 수 있는 프로퍼티와 읽기 전용 프로퍼티로 구분되듯이
읽고 쓸 수 있는 컬렉션과 읽기 전용 컬레션으로 구분된다.

### 데이터 클래스의 copy

immutable 객체를 사용하면 다음과 같은 장점이 있다.

1. 한 번 정의된 상태를 유지해서, 코드 이해가 쉽다.
2. 공유할 때 충돌이 나지 않아서, 병렬 처리를 안전하게 할 수 있다.
3. immutable 객체에 대한 참조가 변경되지 않아서, 쉽게 캐쉬 가능하다.
4. 객체를 복사할 때 깊은 복사를 따로 하지 않아도 된다.
5. 실행을 더 쉽게 에측할 수 있다.
6. set or map 의 키로 사용할 수 있다.

immutable 객체는 변경할 수 없기 때문에, 자신의 일부를 수정하고자 하면 새로운 객체를 만들어야한다.
Int 는 내부적으로 plus 와 minus 메서드로 자신을 수정한 새로운 Int 를 리턴한다.
직접 만드는 immutable 객체도 마찬가지로 동작해야한다.

```kotlin
class User(
    val name: String,
    val surname: String
) {
    fun withSurname(surname: String) = User(name, surname)
}

@Test
fun `자신을 수정한 새로운 객체를 만듦`() {
    var user = User("junhee", "ko")
    user = user.withSurname("kooo")
    assertEquals("kooo", user.surname)
}
```

모든 프로퍼티를 대상으로, withSurname 같은 함수를 하나하나 만드는 것은 귀찮다.
그래서 data 한정자를 사용한다.

```kotlin
data class User2(
    val name: String,
    val surname: String
)

@Test
fun `data class`() {
    var user = User2("junhee", "ko")
    user = user.copy(surname = "kooo")
    assertEquals("kooo", user.surname)
}
```

## 변경의 스코프를 최소화해라

1. 프로퍼티보다 지역 변수를 사용해라.
2. 최대한 좁은 스코프를 갖는 변수를 사용해라. ex) 반복문 내부에서만 사용되는 변수

스코프를 좁게 만들면 좋은 이유는,

1. 프로그램 추적 및 관리가 쉽다.
2. 변수의 스코프가 넓으면, 다른 개발자에 의해 변수가 잘못 사용될 수 있다.

## 최대한 플랫폼 타입을 사용하지 마라

자바에서 String 타입을 리턴하는 메서드가 있다고 하자.
@Nullable 이 붙어 있다면, nullable 로 추정하고 String? 으로 변경된다.
@NotNull 이 붙어 있다면, String 으로 변경된다.

만약, 자바에서 List<User> 를 리턴하고 아무런 annotation 이 붙어 있지 않다고 하자.
코틀린이 디폴트로 모든 타입을 nullable 로 다룬다면, 리스트 자체와 리스트 내부에 있는 것들도 널인지 확인해야한다.

이처럼, 널 확인이 복잡하기 때문에 다른 언어에서 넘어온 타입은 platform type 으로 다룬다.
타입 이름 뒤에 ! 를 붙여서 표기한다.

```java
public class UserRepo{
    public User getUser() {
      // ...
    }
}
```

```kotlin
val repo = UserRepo()
val user1 = repo.user // type: User!
val user2: User = repo.user
val user3: User? = repo.user
```

문제는 null 이 아니라고 생각되는 것이 null 일 가능성이 있어서 위험하다는 것이다.
그래서,

1. 함수가 당장 널을 리턴하지 않아도 미래에 변경될 수 있다는 것을 염두해두거나,
2. 자바를 직접 조작할 수 있으면, @Nullable 이나 @NotNull 을 붙여셔 사용해야한다.
3. 플랫폼 타입이 다른 곳에서 사용되는 것은 항상 위험을 내포하므로, 가능하면 제거해라.

## inferred 타입으로 리턴하지 마라

리턴 타입은 외부에서 확인할 수 있도록 명시적으로 지정하자.
inferred 타입으로 리턴할 경우 발생할 수 있는 문제를 예로 보자.

자동차를 생상하는 CarFactory 가 있다.

```kotlin
interface CarFactory {
    fun produce(): Car
}
```

대부분의 공장에서 디폴트로 레이를 생성한다고 하자.
그래서, DEFAULT_CAR 는 Car 타입으로 명시하고, produce 의 리턴 타입을 제거하자.

```kotlin
val DEFAULT_CAR: Car = Ray()

interface CarFactory {
  fun produce(): DEFAUT_CAR
}
```

그런데 이후에, 다른 개발자가 DEFAULT_CAR 는 타입 추론으로 Car 타입으로 지정될 것이므로 다음과 같이 수정했다고 하자.

```kotlin
val DEFAULT_CAR = Ray()

interface CarFactory {
  fun produce(): DEFAUT_CAR
}
```

이제, CarFactory 에서는 Ray 이외의 자동차 생산을 할 수 없다.

## 예외를 활용해 코드에 제한을 걸어라

확실하게 어떤 형태로 동작해야하는 코드가 있으면, 예외를 활용해 제한을 걸어라.
다음 방법들이 있다.

1. require 블록: argument 제한
2. check 블록: 상태와 관련된 동작 제한
3. assert 블록: 어떤 것이 true 인지 확인 (테스트 모드에서 동작)
4. Elvis 연산자: return 또는 throw 와 함께 활용

### Argument

require 함수는 제한을 확인하고, 만족하지 못하면 IllegalArgumentException 을 발생시킨다.
일반적으로 함수 가장 앞부분에 위치해서, 코드를 읽을 때 쉽게 확인 가능하다.

```kotlin
fun factorial(n: Int): Long {
    require(n >= 0)
    // ...
}
```

### 상태

check 함수는 상태와 관련된 제한을 걸 때 사용한다.
지정된 상태가 아니라면, IllegalStateException 을 발생시킨다.
일반적으로 require 블록 뒤에 위치시킨디.

```kotlin
fun getUserInfo(): UserInfo {
    checkNotNull(token)
    // ...
}
```

### Assert 계열 함수 사용

스스로 구현한 내용을 확인할 때 assert 계역 함수를 사용할 수 있다.
단위 테스트로 구현의 정확성을 확인할 수 있지만, 함수 내부에서 직접 확인해볼 수 있다.

```kotlin
fun pop(num: Int = 1): List<T> {
    // ...
    assert(ret.size == num)
    return ret
}
```

테스트할 때만 활성화되므로, 오류가 발생해도 사용자는 알 수 없다.
그래서, 심각한 오류라면 check 를 사용하는 것이 좋다.
그리고, assert 를 활용하더라도 여전히 단위 테스트는 작성해야한다. assert 는 양념처럼 사용해라.

### nullability 와 스마트 캐스팅

require 과 check 블록으로 어떤 조건을 확인해서 true 이면, 해당 조건은 이후로도 true 로 가정한다.
이를 활용해서 타입 비교를 하면, 스마트 캐스트가 작동한다.

```kotlin
fun changeDress(person: Person){
    require(person.outfit is Dress)
    val dress: Dress = person.outfit
}
```

어떤 대상이 null 인지 확인할 때도 유용하다.
requireNotNull, checkNotNull 함수를 사용해도 좋다.

```kotlin
fun sendEmail(person: Person, text: String){
    require(person.email != null)
    val email: String = person.email
}
```

return 과 throw 를 활용한 Elvis 연산자는 nullable 을 확인할 때 많이 사용된다.

```kotlin
fun sendEmail(person: Person, text: String){
    val email: String = person.email ?: return
}
```

run 함수를 조합해서 사용할 수도 있다.

```kotlin
fun sendEmail(person: Person, text: String){
    val email: String = person.email ?: run {
      log("Email not send, no email address")
      return
    }
}
```

## 사용자 정의 오류보다 표준 오류를 사용해라

표준 라이브러리의 오류는 많은 개발자가 알고 있기 때문에, 이를 재사용하는 것이 좋다.
다른 사람들이 API 를 더 쉽게 배우고 이해할 수 있다. 예를 들어,
- IllegalArgumentException,
- NoSuchElementException
- UnsupportedOperationException

---

이펙티브 코틀린 <마르친 모스칼라>
