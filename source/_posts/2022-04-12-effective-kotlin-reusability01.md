---
layout: post
title: Effective Kotlin - Reusability 1
date: 2022-04-12
categories: Kotlin
---

코틀린의 재사용성을 활용하는 방법을 정리한다.

- knowledge 를 반복해서 사용하지 마라
- 일반적인 알고리즘을 반복해서 구현하지 마라
- 일반적인 프로퍼티 패턴은 프로퍼티 위임으로 만들어라

## knowledge 를 반복해서 사용하지 마라

### knowledge

knowledge 를 반복해서 사용하지 말라는 의미는, 아래와 같이도 표현할 수 있다.

- DRY 규칙: Don't Repeat Yourself
- WET 안티 패턴: We Enjoy Typing, Waste Everyone's Time, Write Everything Twice

knowledge 는 코드 또는 데이터로 표현할 수 있는, 의도적인 정보이다.
프로그램에서 중요한 두 가지 knowledge 는,

1. 비즈니스 로직: 프로그램이 어떤 식으로 동작하는지 (시간에 따라 계속해서 변한다)
2. 공통 알고리즘: 원하는 동작을 하기 위한 알고리즘 (한 번 정의되면 쉽게 변하지 않는다)

둘의 가장 큰 차이는 "변화" 이다.

### 모든 건은 변화한다

대부분의 프로젝트는 요구 사항과 내부적인 구조가 계속해서 변경된다.
변화할 때 가장 큰 문제는 knowledge 가 반복되어 있는 부분이다.

만약에, 프로그램 내부의 반복되어있는 코드를 변경하려면 어떻게 해야할까 ?
반복된 부분 모두를 찾아 변경하는 것은, 또 다른 문제를 야기한다.
실수로 변경하지 못한 부분이 있을 수 있을 수 있다.

따라서, knowledge 반복은 프로젝트의 scalable 을 막고 fragile 하게 만든다.

### 단일 책임 원칙

knowledge 반복처럼 보이지만, 실질적으로 다른 knowledge 이여서 추출해서는 안되는 경우가 있다.
코드를 추출해도 되는지 확인할 수 있는 원칙으로 "단일 책임 원칙" 이 있다.

단일 책임 원칙의 정의는 다음과 같다.
"A class should have only one reason to change"

예를 들어보자.
Student 라는 클래스가 있고, 이 클래스는 장학금 부서와 인증 부서에서 모두 사용된다.
두 부서에서 아래 두 프로퍼티를 추가했다.

- qualifiesForScholarship: 장학금을 받을 수 있는 자격이 되는지를 나타냄
- isPassing: 인증을 통과했는지를 나타냄

이 두 프로퍼티는 이전 학기 성적으로 계산이 된다.
다음과 같이 두 프로퍼티를 한 번에 계산하는 function 을 만들자.

```kotlin
class Student {
    // ..
    fun isPassing(): Boolean =
        calculatePointsFromPassedCourse() > 15

    fun qualifiesForScholarship(): Boolean =
        calculatePointsFromPassedCourse() > 30

    private fun calculatePointsFromPassedCourse(): Int {
        // ..
    }
}
```

장학금 관련해서, 덜 중요한 과목은 장학금 포인트를 줄여야하는 요구 사항이 생겼다.
calculatePointsFromPassedCourse 를 수정했다고 하자.

이렇게 되면, 인증을 통과할 수 있는 학생이 통과하지 못할 수 있다.
이런 문제가 발생한 이유는, calculatePointsFromPassedCourse 가 자신이 해야하는 책임 이이의 책임을 가지고 있기 때문이다.

이런 문제를 막기 위해서는, 처음부터 책임에 따라서 다른 클래스로 구분해서 만들어야한다.

- StudentIsPassingValidator
- StudentQualifiesForScholarshipValidator

또는, 확장 함수를 사용할 수 있다.

```kotlin
fun Student.qualifiesForScholarship(): Boolean {
    /// ...
}

fun Student.calculatePointsFromPassedCourse(): Boolean {
  /// ...
}
```

따라서,

- 서로 다른 곳에서 사용하는 knowledge 는 독립적으로 변경할 가능성이 있다. 비슷한 처리를 해도, 완전히 다른 knowledge 로 취급해라.
- 다른 knowledge 는 분리해서 둬라. 그렇지 않으면, 재사용하며 안되는데 재사용할 가능성이 있다.

## 일반적인 알고리즘을 반복해서 구현하지 마라

여기서의 알고리즘은,
비즈니스 로직을 포함하지 않는 수학적인 연산 같은 별도의 모듈 or 라이브러리로 분리할 수 있는 부분이다.

일반적인 알고리즘은 대부분 이미 정의되어 있다. 대표적으로 stdlib 가 있다.
stdlib 는 확장 함수를 활용해서 만들어진 유틸리티 라이브러리이다.

상황에 따라, 표준 라이브러리에 없는 알고리즘이 필요할 수 있다.
예를 들어, 모든 숫자의 곱을 계산하는 라이브러리가 필요하다고 하자.
아래 처럼, 범용 유틸리티 함수를 정의할 수 있다.

```kotlin
fun Iterable<Int>.product() =
    fold(1) { acc, i -> acc * i }
```

코틀린의 stdlib 처럼 위 코드도 확장 함수로 구현되어있다.
확장 함수로 정의하면,

1. 함수는 상태를 유지하지 않아서, 행위를 나타내기 좋다.
2. top-level 함수와 비교해서, 구체적인 타입의 객체에만 사용을 제한할 수 있다.
3. 수정할 객체를 argument 로 전달받아 사용하는 것 보다, 확장 리시버로 사용하는 것이 가독성에 좋다.
4. 자동 완성 제안으로 쉽게 찾을 수 있다. TextUtils.isEmpty("Text") 는 isEmpty 가 어디에 있는지 알아야한다. but not, "Text".isEmpty()

## 일반적인 프로퍼티 패턴은 프로퍼티 위임으로 만들어라

프로퍼티 위임을 사용하면, 일반적인 프로퍼티의 행위를 추출해서 재사용할 수 있다.
프로퍼티 위임을 어떻게 활용할 수 있는지 간단한 프로퍼티 delegate 를 만들어보자.

프로퍼티가 사용될 때마다, 간단한 로그를 출력해보자.

```kotlin
var token: String? = null
    get() {
        print("token returned value $field")
        return field
    }
    set(value) {
      print("token changed from $field to $value")
      field = value
    }

var attempts: Int = 0
  get() {
    print("attempts returned value $field")
    return field
  }
  set(value) {
    print("attempts changed from $field to $value")
    field = value
  }
```

두 프로퍼티는 타입이 다르다.
하지만, 내부적으로 같은 처리를 하고 프로젝트에서 자주 반복될 수 있는 패턴이다.
따라서, 프로퍼티 위임을 활용해서 추출할 수 있다.

프로퍼티 위임은 다른 객체의 메서드를 활용해서 프로퍼티 접근자 (getter, setter) 를 만드는 방식이다.
객체를 만들고, by 키워드로 프로퍼티 접근자 (getter, setter) 를 정의한 클래스를 연결하면 된다.

아래와 같이.

```kotlin
var token: String? by LoggingProperty(null)
var attempts: Int by LoggingProperty(0)

private class LoggingProperty<T>(var value: T) {

    operator fun getValue(
        thisRef: Any?,
        prop: KProperty<*>
    ): T {
      print("${prop.name} returned value $value")

      return value
    }

    operator fun setValue(
      thisRef: Any?,
      prop: KProperty<*>,
      newValue: T
    ){
      val name = prop.name
      print("$name changed from $value to $newValue")

      value = newValue
    }
}
```

객체를 프로퍼티 위임하려면, 이렇게 연산이 필요하다.

- val 은 getValue
- var 은 getValue, setValue

이런 연산은 위 예제와 같이 멤버 함수로 만들 수 있지만, 확장 함수로도 만들 수 있다.

```kotlin
val map: Map<String, Any> = mapOf(
    "name" to "jko",
    "programmer" to true
)

val name by map
println(name)
```

위 코드의 출력결과는 "jko" 이다.
stdlib 에 아래와 같은 확장 함수가 정의되어 있기 때문이다.

![](/image/effective-kotlin-property-delegate-map-stdlib.png)

---

이펙티브 코틀린 <마르친 모스칼라>
