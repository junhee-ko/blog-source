---
layout: post
title:  "클래스"
date:   2021-08-03
categories: Kotlin
---

다음 내용을 정리한다.

- 클래스 계층을 정의하는 방식
- 상속 제어 변경자
- 가시성 변경자
- sealed 변경자

## 인터페이스

간단한 인터페이스를 정의해보자.

```kotlin
interface Clickable {
    fun click()
}
```

위 간단한 인터페이스를 구현해보자.

```kotlin
class Button : Clickable {
    override fun click() = println("Button was clicked")
}
```

extends 와 implements 키워드를 사용하는 Java 와 다르다.
클래스 이름 뒤에 콜론을 붙여서 클래스 확장과 인터페이스 구현을 모두 처리한다.

인터페이스 메서드도 디폴트 구현을 제공할 수 있다.
자바처럼 default 키워드를 붙일 필요는 없다.

```kotlin
interface Clickable {
    fun click()
    fun showOff() = println("I am clickable") // 디폴트 구현이 있는 메서드
}
```

showOff() 메서드를 정의하는, 다른 인터페이스를 정의해보자.

```kotlin
interface Focusable {
    fun setFocus(b: Boolean) = println("Set Foucs")
    fun showOff() = println("I am focusable")
}
```

한 클래스가 Clickable 과 Focusable 인터페이스를 같이 구현하면 어떻게 될까 ?
두 인터페이스 모두가 showOff 디폴트 메서드가 있는데, 어느 메서드가 선택될까 ?
정답은, showOff 구현을 대체할 오버라이딩 메서드가 없으면, 컴파일 에러가 발생한다.

![](/image/kotlin-interface-many-default-method.png)

따라서, 아래와 같이 직접 구현해야한다.

```kotlin
class Button : Clickable, Focusable {
    override fun click() = println("Button was clicked")

    override fun showOff() {
        super<Clickable>.showOff()
        super<Focusable>.showOff()
    }
}
```

## 상속 제어 변경자 : final

자바의 클래스와 메서드는 기본적으로 상속에 대해 열려있다.
하지만, 코틀린에서는 닫여있다. 즉, final 이다.
어떤 클래스의 상속을 허용하려면, open 변경자를 붙여야한다.
그리고, 오버라이드를 허용하고 싶은 메서드나 프로퍼티 앞에도 open 변경자를 붙여야한다.

```kotlin
open class RichButton : Clickable {
    fun disable() {}        // final: 하위 클래스가 오버라이드 불가능
    open fun animate() {}   // open: 하위 클래스가 오버라이드 가능
    override fun click() {} // open: 하위 클래스가 오버라이드 가능
}
```

기반 클래스나 인터페이스를 오버라이드하면, 그 메서드는 open 이다.
오버라이드한 메서드의 구현을 하위 클래스에서 오버라이드 못하게 하려면, 아래와 같이 final 을 붙이자.

```kotlin
open class RichButton : Clickable {
    fun disable() {}
    open fun animate() {}
    final override fun click() {} // final 이 없으면, 열려있음
}
```

추상 멤버는 항상 열려있다. 그래서 open 변경자를 붙일 필요 없다.
하위 클래스에서 이 추상 멤버를 오버라이드 해야하기 때문이다.

```kotlin
abstract class Animated {
    abstract fun animate()
    fun animateTwice() {} // 추상 클래스에 속해도, 비추상 함수는 기본적으로 final
    open fun stopAnimating() {}
}
```

## 가시성 변경자 : public

가시성 변경자는, 클래스의 외부 접근을 제어한다.
자바의 기본 가시성은 package-private 이다. ( visible only from the same package )
코틀린에서는, public 이다.

코틀린에서는, 패키지를 namespace 관리 목적으로만 사용해서 패키지를 가시성 제어에 사용하지 않는다.
대신, 모듈 내부에서만 볼 수 있는 internal 이라는 가시성 변경자가 있다.
모듈이란, 한 번에 컴파일 되는 코틀린 파일들을 의미한다.

## 봉인된 클래스 : 계층 확장 제한

상위 클래스인 Expr 과 숫자를 표현하는 Num, 덧셈을 표현하는 Sum 두 하위 클래스가 있다.

```kotlin
interface Expr
class Num(val value: Int) : Expr
class Sum(val left: Expr, val right: Expr) : Expr
```

when 을 사용해 Expr 의 타입을 검사할 때, 디폴트 분기인 else 가 반드시 있어야한다.

```kotlin
fun eval(e: Expr): Int =
    when (e) {
        is Num -> e.value
        is Sum -> eval(e.right) + eval(e.left)
        else -> throw IllegalArgumentException()
    }
```

새로운 클래스를 추가하고 위 분기에 추가를 하지 않으면, 디폴트 분기가 선택되어서 버그가 발생할 수 있다.

이 문제는, sealed 클래스로 해결할 수 있다.
상위 클래스에 sealed 변경자를 붙이면, 하위 클래스 정의를 제한한다.

```kotlin
sealed class Expr { // sealed 로 봉인

    // 하위 클래스를 중첩 클래스로 나열
    class Num(val value: Int) : Expr()
    class Sum(val left: Expr, val right: Expr) : Expr()
}
```

그러면, 아래와 같이 else 분기가 필요없다.

```kotlin
fun eval(e: Expr): Int =
    when (e) {
        is Expr.Num -> e.value
        is Expr.Sum -> eval(e.right) + eval(e.left)
    }
```

sealed 클래스의 하위 클래스가 추가된다면,
when 식이 컴파일 되지 않아 when 식을 고쳐야한다는 사실을 쉽게 알 수 있다.

---
Kotlin In Action <드미트리 제메로프, 스베트라나 이사코바>
