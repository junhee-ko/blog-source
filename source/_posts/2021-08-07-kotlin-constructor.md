---
layout: post
title:  "생성자"
date:   2021-08-04
categories: Kotlin
---

다음 내용을 정리한다.

- 주 생성자, 부 생성자
- 인터페이스의 프로퍼티
- field 키워드
- 접근자 가시성

## 클래스 초기화 : 주 생성자, 초기화 블록

클래스 이름 뒤에 괄호로 둘러싸인 코드가 주 생성자이다.

```kotlin
class User(val nickname: String)
```

같은 목적을 이룰 수 있는 다은 형태의 코드를 보자.

```kotlin
class User constructor(_nickname: String) {
    val nickname: String

    init {
        nickname = _nickname // _ : 프로퍼티와 생성자 파라미터를 구분하기 위해 사용
    }
}
```

constructor 키워드는, 주 생성자나 부 생성자를 정의할 때 사용한다.
init 키워드는, 초기화 블록을 시작한다. 초기화 블록은, 클래스의 객체가 만들어질 때 실행된다.

위 코드에서,
1. nickname 프로퍼티를 초기화 하는 코드를, nickname 프로퍼티 선언에 포함 가능
2. 주 생성자 앞에 다른 annotation 이나 가시성 변경자 없다면, constructor 키워드 생략 가능

```kotlin
class User(_nickname: String) {
    val nickname = _nickname // 프로퍼티를, 주 생성자의 파라미터로 초기화
}
```

그런데, 아래처럼 val 을 추가해서 주 생성자의 파라미터로 프로퍼티를 초기화 가능하다.

```kotlin
class User(val nickname: String) // val 은 파라미터에 상응하는 프로퍼티가 생성된다는 의미
```

클래스에 기반 클래스가 있으면, 주 생성자에서 기반 클래스의 생성자를 호출해야한다.
기반 클래스 이름 뒤에 괄호를 쳐서 생성자 인자를 넘기면 된다.

```kotlin
open class User(val nickname: String)
class FacebookUser(nickName: String) : User(nickName) // Here !!
```

클래스 정의할 때 별도로 생성자를 정의하지 않으면, 컴파일러가 인자 없는 디폴트 생성자를 만든다.
하위 클래스는 기반 클래스의 이 디폴트 생성자를 아래처럼 호출해야한다.

```kotlin
open class Button           // 인자 없는 디폴트 생성자가 만들어짐
class RadioButton: Button() // 생성자 호출
```

이 처럼, 기반 클래스의 이름 뒤에는 꼭 괄호가 들어간다.
인터페이스는 생성자가 없어서, 어떤 클래스가 구현하는 경우 괄호가 안 들어간다.

만약에 어떤 클래스를 클래스 외부에서 인스턴스화하지 못하게 하려면, 모든 생성자를 private 으로 만들자.

```kotlin
class Secret private constructor() {} // 이 클래스의 유일한 주 생성자를 비공개
```

## 부 생성자 : 상위 클래스를 다른 방식으로 초기화

클래스에 주 생성자가 없으면 모든 부 생성자는

- 상위 클래스를 초기화하거나
- 다른 생성자에게 생성을 위임해야한다.

```kotlin
open class View {

    constructor(button: String) {
        // ...
    }

    constructor(button: String, image: String) {
        // ...
    }
}

class Button : View {

    constructor(button: String) : this(button, "IMAGE"){ // 이 클래스의 다른 생성자에게 위임
        // ...
    }

    constructor(button: String, image: String) : super(button, image) { // 상위 클래스 초기화
        // ...
    }
}
```

## 인터페이스에 선언된 프로퍼티

인터페이스에 추상 프로퍼티 선언을 넣을 수 있다.

```kotlin
interface User {
    val nickname: String
}
```

이 인터페이스를 구현하는 방법을 보자.

```kotlin
// User 의 추상 프로퍼티를 구현하므로, override 를 붙여야한다.
class PrivateUser(override val nickname: String) : User
```

뒷받침 필드에 값을 저장하지 않고, 매번 이메일 주소에서 별명을 계산해서 반환한다.

```kotlin
class SubscribingUser(val email: String) : User {
    override val nickname: String
        get() = email.substringBefore('@') // Custom Getter
}
```

객체를 초기화하는 단계에서 한 번만, getFacebookName 을 호출한다.

```kotlin
class FacebookUser(val accountId: Int) : User {
    override val nickname = getFacebookName(accountId) // 프로퍼티 초기화 식

    private fun getFacebookName(accountId: Int): String {
        return "test:accountId"
    }
}
```

그리고 인터페이스에는 추상 프로퍼티 뿐만 아나라, getter 와 setter 가 있는 프로퍼티를 선언할 수 있다.

```kotlin
interface User {
    val email: String       // 하위 클래스에서 반드시 오버라이드 해야함
    val nickName: String    // 오버라이드 하지 않고 상속할 수 있음
        get() = email.substringBefore('@')
}
```

## getter, setter 에서 뒷받침하는 필드에 접근

field 키워드를 이용해서, 접근자 안에서 프로퍼티를 뒷밤침 필드에 접근할 수 있다.

```kotlin
class User(val name: String) {
    var address: String = "Incheon"
        set(value: String) {
            println(
                """
                address was changed !!
                $field -> $value
            """.trimIndent()    // 뒷받침하는 필드 값 읽기
            )

            field = value       // 뒷받침하는 필드 값 변경
        }
}
```

## 접근자 가시성 변경

접근자의 가시성은 프로퍼티 가시성과 같다.
get, set 앞에 가시성 변경자를 추가해서, 접근자 가시성을 변경할 수 있다.

```kotlin
class LengthCounter {

    var counter: Int = 0
        private set // 이 클래스 밖에서 이 프로퍼티 값 변경 불가능

    fun addWord(word: String) {
        counter += word.length
    }
}
```

---
Kotlin In Action <드미트리 제메로프, 스베트라나 이사코바>
