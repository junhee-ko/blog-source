---
layout: post
title:  "object 키워드"
date:   2021-08-07
categories: Kotlin
---

object 키워드가 사용되는 경우인, 다음 내용을 정리한다.

- 객체 선언
- 동반 객체
- 객체 식

## 객체 선언 : 싱글턴 객체 생성

인스턴스가 하나만 필요한 경우가 있다.
자바에서는 다음과 같이 singleton pattern 으로 구현할 수 있다.

1. 클래스의 생성자를 private 으로 제한하고,
2. 정적인 플드에 그 클래스의 유일핸 객체를 저장

코틀린은 '객체 선언' 기능을 통해, 싱글턴 패턴을 언어에서 기본 지원한다.
( 객체 선언 = 클래스 선언 + 그 클래스에 속한 단일 인스턴스 선언 )

object keyword 를 사용해서,
클래스를 정의하고 그 클래스의 인스턴스를 만들어서 변수에 저장하는 모든 작업을 한 문장으로 처리한다.
일반 클래스의 객체와 달리, 싱글턴 객체는 객체 선언문이 있는 위치에서 생성자 호출없이 즉시 만들어진다.

```kotlin
object Payroll {
    val allEmployees = arrayListOf<Person>()

    fun calculateSalary() {
        for (person in allEmployees) {
            //..
        }
    }
}

// usage
Payroll.allEmployees.add(Person(name = "jko", isMarried = false))
Payroll.calculateSalary()
```

클래스 안에 객체로 선언할 수도 있다.

```kotlin
data class Person(val name: String) {

    object NameComparator : Comparator<Person> {
        override fun compare(o1: Person, o2: Person): Int = o1.name.compareTo(o2.name)
    }
}

// usage
val persons = listOf(Person("jko"), Person("junhee-ko"))
println(persons.sortedWith(Person.NameComparator))
```

## 동반 객체 : 팩토리 메서드와 정적 멤버가 들어갈 장소

코틀린은, static 키워드를 지원하지 않는다. 대신, top-level function 을 지원한다.
하지만, top-level function 은 클래스의 비공개(private) 멤버에 접근할 수 없다.

그래서, 클래스의 객체와 관계없이 호출하며 클래스의 내부 정보에 접근해야하는 함수가 필요할 때는
클래스에 중첩된 객체 선언의 멤버 함수로 정의해야한다.

생성자를 통해 User 객체를 만들 수 없고 팩토리 메서드를 이용해야하는 예를 보자.

```kotlin
class User private constructor(val nickName: String) { // 주 생성자 비공개

    companion object { // 동반 객체 선언
        fun newSubscribingUser(email: String) = User(email.substringBefore('@'))
        fun newFacebookUser(accountId: Int) = User("test:accountId")
    }
}

// usage
val subscribingUser = User.newSubscribingUser("junheee.ko@gmail.com")
val facebookUser = User.newFacebookUser(1)
```

동반 객체도 결국, 클래스 안에 정의된 일반 객체이다. 그래서,

1. 이름을 붙이거나
2. 인터페이스를 상속하거나
3. 동반 객체 안에 확장 함수와 프로퍼티를 정의할 수 있다.

### 동반 객체에 이름 붙이기

```kotlin
class Person(val name: String) {

    companion object Loader { // 이름을 붙임
        fun fromJson(jsonText: String): Person = Person("$jsonText")
    }
}

// usage
println(Person.Loader.fromJson("{name: 'jko'}"))
println(Person.fromJson("{name: 'junhee-ko'}"))
```

### 동반 객체의 인터페이스 구현

```kotlin
interface JsonFactory<T>{
    fun fromJson(jsonText: String): T
}

class Person(val name: String) {

    companion object : JsonFactory<Person> { // 인터페이스 구현
        override fun fromJson(jsonText: String): Person = Person("$jsonText")
    }
}

fun loadFromJSON<T>(factory: JSONFactory<T>): T {
  //...
}

loadFromJSON(Person) // Person 클래스의 이름을 사용
```

### 동반 객체의 확장

```kotlin
class Person(val name: String) {

    companion object { // 비어있는 동반 객체 선언

    }
}

// 확장 함수 선언
fun Person.Companion.fromJson(json: String): Person = Person("$json")

// usage
val json = "..."
val p = Person.fromJson(json)
```

fromJson 은 클래스 밖에 정의한 확장 함수이다. 그런데, 동반 객체 안에 정의한 것 처럼 호출 할 수 있다.

## 객체 식 : 무병 내부 클래스를 다른 방식으로 작성

anonymous object (무명 객체) 를 정의할 때 object 키워드를 사용할 수 있다.
무명 객체는 자바의 무명 내부 클래스를 대신한다.
자바의 무명 내부 클래스로 많이 구현하는 event listener 를 코틀린으로 구현해보자.

```kotlin
window.addMouseListener(
  object: MouseAdapter() { // MouseAdapter 를 확장하는 무명 객체 선언
      override fun mouseClicked(e: MouseEvent) {
          // ..
      }

      override fun mouseEntered(e: MouuseEvent){
          // ..
      }
  }
)
```

객체에 이름을 붙여야한다면, 변수메 무명 객체를 대입하면 된다.

```kotlin
val listener = object: MouseAdapter() {
  override fun mouseClicked(e: MouseEvent) {
    // ..
  }

  override fun mouseEntered(e: MouuseEvent){
    // ..
  }
}
```

자바와 달리 final 이 아닌 변수도 객체 식 안에서 사용 가능하다.

```kotlin
fun countClicks(window: Window) {
    var clickCount = 0 //  final 이 아닌 변수

    window.addMouseListener(
        object : MouseAdapter() {
            override fun mouseClicked(e: MouseEvent?) {
                clickCount++
            }
        }
    )
}
```

---
Kotlin In Action <드미트리 제메로프, 스베트라나 이사코바>
