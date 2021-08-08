---
layout: post 
title:  "데이터 클래스"
date:   2021-08-07 
categories: Kotlin
---

다음 내용을 정리한다.

- 데이터 클래스 : toString(), equals(), hashCode(), copy()
- 클래스 위임 패턴 : by

## 모든 클래스가 정의해야하는 메서드

간단한 클래스를 작성해보자.

```kotlin
class Client(val name: String, val postCode: Int)
```

이 클래스에 대해서,
toString(), equals(), hashCode() 메서드들이 어떻게 사용되는지 보자.

### toString()

기본 제공되는 객체의 문자열 표현은 다음과 같은 형식이다 : Client@5e9f23bf
기본 구현을 바꾸려면 toString 메서드를 오버라이드해야한다.

```kotlin
class Client(val name: String, val postCode: Int) {

    override fun toString(): String = "Client(name='$name', postCode=$postCode)"
}
```

### equals()

```kotlin
val client1 = Client("jko", 1234)
val client2 = Client("jko", 1234)
println(client1 == client2)
```

위 프린트의 결과는 false 이다.
코틀린에서 == 는 참조 동일성을 검사하지 않는다.
객체의 동등성을 검사한다.
그래서, equals 를 호출하는 식으로 컴파일 된다.

Client 클래스에 equals 를 추가해보자.

```kotlin
class Client(val name: String, val postCode: Int) {
    
    override fun equals(other: Any?): Boolean {
        if (this === other) return true
        if (javaClass != other?.javaClass) return false

        other as Client

        if (name != other.name) return false
        if (postCode != other.postCode) return false

        return true
    }

    override fun toString(): String = "Client(name='$name', postCode=$postCode)"
}
```

그러면, 이제 프린트의 결과는 true 이다.

### hashCode()

```kotlin
val hashSet = hashSetOf(Client("jko", 1234))
println(hashSet.contains(Client("jko", 1234)))
```

위 프린트의 결과는 false 이다.
HashSet 은 원소를 비교할 때의 비용을 줄이기 위해,
1. 객체의 해쉬 코드를 비교
2. 같으면, 실제 값을 비교

그런데, 위 예에서는 두 Client 객체의 해쉬 코드가 다르기 때문에
두 번째 Client 객체가 집합 안에 들어있지 않다고 판단한다.

이 문제를 해결하려면 Client 클래스가 hashCode 를 구현하면 된다.

```kotlin
class Client(val name: String, val postCode: Int) {

    override fun equals(other: Any?): Boolean {
        if (this === other) return true
        if (javaClass != other?.javaClass) return false

        other as Client

        if (name != other.name) return false
        if (postCode != other.postCode) return false

        return true
    }

    override fun hashCode(): Int {
        var result = name.hashCode()
        result = 31 * result + postCode
        return result
    }

    override fun toString(): String = "Client(name='$name', postCode=$postCode)"
}
```

## 데이터 클래스

어떤 클래스가 데이터를 저장하는 역할만을 수행한다면
toString, equals, hashCode 메서드를 반드시 오버라이드해야한다.

코틀린의 data 클래스는 이 메서드들이 컴파일러에 의해 자동으로 만들어진다.

```kotlin
data class Client(val name: String, val postCode: Int)
```

### copy()

데이터 클래스의 객체를 불변 객체로 쉽게 활용할 수 있도록,
코틀린 컴파일러는 copy 메서드를 만들어준다.

```kotlin
val origin = Client("jko", 1234)
val copy = origin.copy(name = "jko", postCode = 5678)
```

## by : 클래스 위임

상속을 허용하지 않는 클래스에 새로운 동작을 추가해아할 때가 있다.
이럴 때, Decorator 패턴을 사용한다. 핵심은,

- 기존 클래스와 같은 인터페이스를 데코레이터가 제공하고
- 기존 클래스를 데코레이터의 내부 필드로 유지

다음 예를 보자.

```kotlin
class DelegatingCollection<T> : Collection<T> {

    private val innerList = arrayListOf<T>()
    
    override val size: Int = innerList.size
    override fun contains(element: T): Boolean = innerList.contains(element)
    override fun containsAll(elements: Collection<T>): Boolean = innerList.containsAll(elements)
    override fun isEmpty(): Boolean = innerList.isEmpty()
    override fun iterator(): Iterator<T> = innerList.iterator()
}
```

위 코드는, 코틀린의 by 키워드를 사용해서
인터페이스에 대한 구현을 다른 객체에 위임 중이라고 명시할 수 있다.

```kotlin
class DelegatingCollection<T>(
    innerList : Collection<T> = ArrayList<T>()
) : Collection<T> by innerList {
    
}
```

---
Kotlin In Action <드미트리 제메로프, 스베트라나 이사코바>
