---
layout: post
title: Effective Kotlin - Reusability 1
date: 2022-04-12
categories: Kotlin
---

코틀린의 재사용성을 활용하는 방법을 정리한다.

- knowledge 를 반복해서 사용하지 마라

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

---

이펙티브 코틀린 <마르친 모스칼라>
