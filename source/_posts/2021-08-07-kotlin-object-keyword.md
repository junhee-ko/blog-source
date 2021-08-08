---
layout: post 
title:  "object 키워드"
date:   2021-08-07 
categories: Kotlin
---

object 키워드가 사용되는 경우인,
다음 내용을 정리한다.

- 객체 선언
- 동반 객체
- 객체 식

## 객체 선언 : 싱글턴 객체 생성

코틀린은 '객체 선언' 을 통해, 싱글턴 패턴을 지원한다.
'객체 선언' = '클래스 선언 + 그 클래스에 속한 단일 객체 선언'

```kotlin
object Payroll {
    val allEmployees = arrayListOf<Person>()

    fun calculateSalary() {
        for (person in allEmployees) {
            //..
        }
    }
}
```

사용할 때는,

```kotlin
Payroll.allEmployees.add(Person(name = "jko", isMarried = false))
Payroll.calculateSalary()
```

일반 클래스의 객체와 달리,
싱글턴 객체는 객체 선언문이 있는 위치에서 생성자 호출없이 즉이 만들어진다.

java.util.Comparator 인터페이스는, 
두 객체를 인자로 받아 어느 객체가 더 큰지 알려주는 정수를 반환한다.
아래와 같이, 클래스 안에 객체로 선언할 수 있다.

```kotlin
data class Person(val name: String) {

    object NameComparator : Comparator<Person> {
        override fun compare(o1: Person, o2: Person): Int = o1.name.compareTo(o2.name)
    }
}
```

사용할 때는,

```kotlin
val persons = listOf(Person("jko"), Person("junhee-ko"))
println(persons.sortedWith(Person.NameComparator))
```

## 동반 객체 : 팩토리 메서드와 정적 멤버가 들어갈 장소

코틀린은, static 키워드를 지원하지 않는다.
대신, top-level function 을 지원한다.
하지만, top-level function 은 클래스의 비공개(private) 멤버에 접근할 수 없다.

그래서, 클래스의 객체와 관계없이 호출하며 클래스의 내부 정보에 접근해야하는 함수가 필요할 때는
클래스에 중첩된 객체 선언의 멤버 함수로 정의해야한다.

아래 코드는,
생성자를 통해 User 객체를 만들 수 없고 팩토리 메서드를 이용해야한다.

```kotlin
class User private constructor(val nickName: String) {

    companion object {
        fun newSubscribingUser(email: String) = User(email.substringBefore('@'))
        fun newFacebookUser(accountId: Int) = User("test:accountId")
    }
}
```

사용할 때는, 

```kotlin
val subscribingUser = User.newSubscribingUser("junheee.ko@gmail.com")
val facebookUser = User.newFacebookUser(1)
```

동반 객체도 결국, 클래스 안에 정의된 일반 객체이다.
그래서,
1. 이름을 붙이거나
2. 인터페이스를 상속하거나
3. 동반 객체 안에 확장 함수와 프로퍼티를 정의할 수 있다.

### 이름 붙이기

```kotlin
class Person(val name: String) {
    
    companion object Loader {
        fun fromJson(jsonText: String): Person = Person("$jsonText")
    }
}
```

사용할 때는,

```kotlin
println(Person.Loader.fromJson("{name: 'jko'}"))
println(Person.fromJson("{name: 'junhee-ko'}"))
```

### 인터페이스 구현

```kotlin
interface JsonFactory<T>{
    fun fromJson(jsonText: String): T
}

class Person(val name: String) {

    companion object : JsonFactory<Person> {
        override fun fromJson(jsonText: String): Person = Person("$jsonText")
    }
}
```

### 동반 객체 확장

```kotlin
class Person(val name: String) {

    companion object {
        // empty
    }
}

fun Person.Companion.fromJson(json: String): Person = Person("$json")
```

fromJson 은 클래스 밖에 정의한 확장 함수이다.
그런데, 동반 객체 안에 정의한 것 처럼 호출 할 수 있다.
이렇게.

```kotlin
Person.fromJson("....")
```

## 객체 식 : 무병 내부 클래스를 다른 방식으로 작성

anonymous object (무명 객체) 를 정의할 때 object 키워드를 사용할 수 있다.
무명 객체는 자바의 무명 내부 클래스를 대신한다.

```kotlin
fun countClicks(window: Window) {
    var clickCount = 0

    window.addMouseListener(
        object : MouseAdapter() {
            override fun mouseClicked(e: MouseEvent?) {
                clickCount++
            }
        }
    )
}
```

객체 선언과 다르게
1. 클래스나 인스턴스에 이름을 붙이지 않는다.
2. 싱글턴이 아니다. 즉, 객체 식이 쓰일 때마다 새로운 인스턴스를 생성한다.

객체 식은 무명 객체 안에서 여러 메서드를 오버라이드해야하는 경우 유용하다.
메서드가 하나 뿐인 인터페이스를 구현해야하면, 코틀린의 SAM 변환 지원을 활용하는 것이 좋다.

---
Kotlin In Action <드미트리 제메로프, 스베트라나 이사코바>
