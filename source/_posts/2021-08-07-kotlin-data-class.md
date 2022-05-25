---
layout: post
title:  "데이터 클래스"
date:   2021-08-06
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
println(client1 == client2) // false
```

코틀린에서 == 는 참조 동일성을 검사하지 않고, 객체의 동등성을 검사한다.
그래서, equals 를 호출하는 식으로 컴파일 된다.

서로 다른 객체가 내부에 동일한 데이터를 가지고 있으면 이 둘을 동등한 객체로 간주해보자.
이 요구사항을 충족시키기 위해서는, equals 를 오버라이드해야한다.

```kotlin
class Client(val name: String, val postCode: Int) {

    override fun equals(other: Any?): Boolean {
        if (other == null || other !is Client) return false

        return name == other.name && postCode == other.postCode
    }

    override fun toString(): String = "Client(name='$name', postCode=$postCode)"
}
```

### hashCode()

JVM 언어에는, hashCode 가 지켜야하는 규약으로 다음이 있다 :
equals() 가 true 를 반한화는 두 객체는 같은 hashCode() 를 반환해야한다.
위에서 작성한 Client 는 이를 어기고 있다.

다음을 보자.

```kotlin
val hashSet = hashSetOf(Client("jko", 1234))
println(hashSet.contains(Client("jko", 1234))) // false
```

HashSet 은 원소를 비교할 때의 비용을 줄이기 위해,

1. 객체의 해쉬 코드를 비교
2. 같으면, 실제 값을 비교

그런데, 두 Client 객체의 해쉬 코드가 달라서 두 번째 Client 객체가 집합 안에 들어있지 않다고 판단한다.
이 문제를 해결하려면 Client 클래스가 hashCode 를 구현하면 된다.

```kotlin
class Client(val name: String, val postCode: Int) {

    // ..

    override fun hashCode(): Int = name.hashCode() * 31 + postCode
}
```

## 데이터 클래스

어떤 클래스가 데이터를 저장하는 역할만을 수행한다면
toString, equals, hashCode 메서드를 반드시 오버라이드해야한다.
코틀린의 data 클래스는 이 메서드들이 컴파일러에 의해 자동으로 만들어진다.

```kotlin
data class Client(val name: String, val postCode: Int)
```

- equals: 인스턴스 간 비교
- hashCode: HashMap 과 같은 해시 기반 컨테이너에서 키로 사용하는 hashCode
- toString: 클래스의 각 필드를 선언 순서대로 표시하는 문자열

### copy()

데이터 클래스의 프로퍼티가 val 일 필요는 없다. 원하면 var 프로퍼티를 사용해도 된다.
하지만, 모든 프로퍼티를 읽기 전용으로 만들어서 데이터 클래스를 immutable 클래스로 만들어라.
왜냐하면,

1. 데이터 클래스 객체를 키로 하는 값을 HashMap 등의 컨테이너에 담고, 데이터 객체의 프로퍼티를 변경하면 컨테이터 상태가 잘못될 수 있다.
2. 불변 객체를 사용하면 프로그램 추론이 쉽다.
3. 다중 스레드 환경에서, 불변 객체를 사용하면 스레드에서 사용 중인 데이터를 다른 스레드에서 변경할 수 없어서 동기화할 필요가 준다.

불변 객체로 쉽게 활용할수 있도록 코틀린 컴파일러는 copy() 라는 편의 메서드를 제공한다.
복사를 하면서 프로퍼티 값을 변경하거나, 복사본을 제거해도 원본을 참조하는 다른 부분에 영향이 없다.

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

    private val innerList = arrayListOf<T>() // 내부 필드로 유지

    override val size: Int = innerList.size
    override fun contains(element: T): Boolean = innerList.contains(element)
    override fun containsAll(elements: Collection<T>): Boolean = innerList.containsAll(elements)
    override fun isEmpty(): Boolean = innerList.isEmpty()
    override fun iterator(): Iterator<T> = innerList.iterator()
}
```

이런 방법의 단점은, 준비 코드가 많다는 것이다.
아무 동작도 변경하지 않는 데코레이터를 만들 때조차 복잡한 코드를 작성해야한다.

이를, 코틀린의 by 키워드를 사용해서 인터페이스에 대한 구현을 다른 객체에 위임 중이라고 명시할 수 있다.

```kotlin
class DelegatingCollection<T>(
    innerList : Collection<T> = ArrayList<T>()
) : Collection<T> by innerList {

}
```

제거된 메서드는, 컴파일러가 자동 생성해준다.
메서드 중 일부 동작을 변경하고 싶으면 오버라이드하면 된다.

---
Kotlin In Action <드미트리 제메로프, 스베트라나 이사코바>
