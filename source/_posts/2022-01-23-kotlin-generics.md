---
layout: post 
title: Generics
date: 2022-01-23
categories: Kotlin
---

다음 내용을 정리한다.

- 제네릭 타입 파라미터
- 제네릭 함수
- 제네릭 클래스
- 제네릭 타입 파라미터 제약
- 제네릭스 동작 원리

## 제네릭 타입 파마리터

제네릭스를 사용하면, 타입 파마리터를 받는 타입을 정의할 수 있다.
제네릭 타입의 인스턴스를 만들려면, 타입 파라미터를 구체적인 타입 파라미터로 치환해야한다.

예를 들어, Map 클래스는 키 타입과 값 타입을 타입 파리미터로 받으므로 Map<K, V> 이다.
이 제네릭 클래스에 대해 제네릭 타입의 인스턴스를 만들려면, 
Map<String, Person> 처럼 구체적인 타입 파라미터로 치환해서 인스턴스화할 수 있다.

### 타입 파라미터 추론

코틀린 컴파일러는 보통의 타입과 마찬가지로, 타입 파라미터도 추론가능하다.

코틀린 Collections 의 listOf function 에는 아래 처럼, 타입 파라미터 T 가 정의되어 있다.

![](/image/kotlin-generics-listof.png)

아래에서는, listOf 에 전달된 두 값이 문자열이므로 여기서 생기는 리스트가 `List<String>` 임을 추론한다.

```kotlin
val names = listOf("junhee", "ko")
```

아래에서는, 빈 리스트를 만든다면 타입 파라미터를 추론할수 없어서 직접 타입 파라미터를 명시해야한다.

```kotlin
val names = listOf<String>()
```

## 제네릭 함수

컬렉션을 다루는 라이브러리의 함수는 대부분 제네릭 함수다.
slice 함수의 정의를 보자.

![](/image/kotlin-slice-function.png)

이 slice 제네릭 함수를 호출해보자.

```kotlin
val letters = ('a'..'z').toList()

val slice1 = letters.slice<Char>(0..2) // 타입 파라미터 명시적으로 지정
val slice2 = letters.slice(10..12)     // 컴파일러가 T 를 Char 로 추론
```

다른 예로, filter 함수의 정의를 보자.

![](/image/kotlin-slice-filter.png)

이 filter 제네릭 함수를 호출해보자.

```kotlin
val names = listOf("james", "kobe", "jko", "messi")
val me = listOf("jko", "junhee", "ko")

names.filter { it in me } // 여기서 it 의 타입은 T 라는 제네릭 타입
```

위에서, 컴파일러는 T 가 String 이라는 사실을 다음 사실로 추론한다.
1. filter 가 `List<T>` 타입의 리스트에 대해 호출될 수 있다는 사실
2. filter 의 수신 객체인 names 의 타입이 `List<String>` 이란 사실

## 제네릭 확장 프로퍼티

리스트의 마지막 원소 바로 앞에 있는 원소를 반환하는 확장 프로퍼티를 만들어보자.

```kotlin
val <T> List<T>.penultimate: T
    get() = this[size - 2]

listOf(1,2,3).penultimate // 타입 파라미터 T 가 Int 로 추론된다.
```

## 제네릭 클래스

타입 파라미터를 이름 뒤에 붙이면, 클래스 본문에서 타입 파리미터를 다른 일반 타입처럼 사용 가능하다.
표준 자바 인터페이스인 List 를 코틀린으로 정의해보자.

```kotlin
interface List<T>{
    
    operator fun get(index: Int): T
}
```

위 인터페이스 안에서 T 를 일반 타입처럼 사용가능하다.

제네릭 클래스를 확장하는 클래스를 정의하기 위해서는,
기반 타입의 제네릭 파라미터에 대해 타입 파라미터를 지정해야한다.
이 때,

1. 구제적인 타입을 넘길 수도 있고
```kotlin
interface List<T>{
    
    operator fun get(index: Int): T
}

class StringList : List<String> {   // 구체적인 타입
    
    override fun get(index: Int): String {
        TODO("Not yet implemented")
    }
}
```
2. 타입 파라미터로 받은 타입을 넘길수도 있다.
```kotlin
interface List<T>{
    
    operator fun get(index: Int): T
}

class ArrayList<T> : List<T> {  // 타입 파라미터로 받은 타입
    
    override fun get(index: Int): T {
        TODO("Not yet implemented")
    }
}
```

## 타입 파라미터 제약

타입 파라미터 제약은 클래스나 함수에 사용하는 타입 파라미터를 제한한다.
제약을 가하려면 타입 파라미터 이름 뒤에 콜론(:) 을 표시하고 그 뒤에 상한 타입을 적으면 된다.

아래 예를 보자.

![](/image/kotlin-generics-constraint.png)

아래 호출은 실제 타입 파라미터인 Int 가 Number 를 확장하므로, 정상 호출된다.

```kotlin
listOf(1,2,3).sum()
```

### 타입 파리미터를 널이 될수 없는 타입으로 한정

다음 예를 보자.

```kotlin
class Processor<T> {
    
    fun process(value: T){  // ? 가 붙어있지 않음
        value?.hashCode()
    }
}

val processor = Processor<String?>()
processor.process(null)
```

value 파라미터의 타입 T 에는 ? 가 붙어있지 않다.
하지만, 실제로 T 에 해당하는 타입 파라미터로 널이 될 수 있는 타입을 넘길 수 있다.

항상 널이 될 수 없는 타입만 타입 파라미터로 받기위해서는, 
Any 를 상한으로 지정하는 제약을 가하면 된다.

```kotlin
class Processor<T: Any> {   // 항상 널이 될 수 없는 타입만 파라미터로 받음
    
    fun process(value: T){
        value.hashCode()
    }
}

fun processorTest(){
    val processor = Processor<String>()
    processor.process("jko")
}
```

## 제레릭스 동작 원리

실행 시점에는, Type Erasure 로 인해 제네릭 클래스의 인스턴스에는 타입 파리미터의 정보가 없다.
inline function 에 reified 타입 파라미터를 사용하면 타입 파라미터를 지워지지 않게 할 수 있다.
이 내용을 구체적으로 정리해보자.

### Type Erasure

다음 코드를 보자.

```kotlin
fun <T> print(list: List<T>) {

    if (list is List<String>) {
        println("list is List<String>")
    }
}
```

위 코드는 컴파일 에러가 발생한다.

```text
Cannot check for instance of erased type: List<String>
```

이러한 에러가 발생하는 이유는,
제네릭 파라미터에 대한 정보가 런타임에 지워지기 때문이다. (== type erasure)
즉, List<String> 객체를 만들고 문자열을 넣어도 실행 시점에는 List 로만 볼 수 있다.

다음을 보자.

```text
fun main(){
    val list = listOf<String>("jko", "junhee")
}
```

위 코드를 컴파일한 결과를, 다시 디컴파일 해보자.


![](/image/kotlin-generics-type-erase-decompile.png)

위와 같이, List 객체가 어떤 타입의 원소를 저장하는지 알 수 없게 되었다.
메모리 사용량이 줄어든다는 장점은 있다. 저장해야하는 타입의 정보가 줄어들었기 때문이다. 

어떤 값이 List 인지는 Star Projection (*) 을 통해서 알 수 있다.

```kotlin
if (list is List<*>) {  }
```

###  reified 타입 파라미터

```kotlin
fun <T> isA(value: Any) = value is T
```

역시 컴파일 에러가 발생한다.

```text
Cannot check for instance of erased type: T
```

제레닉 타입의 파라미터 정보가 실행 시점에 없기 때문이다.
이런 제약을 해결하기 위해, inline 함수에 타입 파라미터를 reified 로 지정하면 된다.

```kotlin
inline fun <reified T> isA(value: Any) = value is T

fun isATest(){
    val isAString1 : Boolean = isA<String>("abc")
    val isAString2 : Boolean = isA<String>(123)
}
```

reified 타임 파라미터를 활용한 표준 라이브러리 함수의 예로, filterIsInstance 가 있다.

```kotlin
val list = listOf("junhee", 123)
val filteredList: List<String> = list.filterIsInstance<String>()
```

filterIsInstance 가 어떻게 작성되었는지 보자.

![](/image/kotlin-generics-filterIsInstance1.png)
![](/image/kotlin-generics-filterIsInstance2.png)

`<reified T>` 는 타입 파라미터가 실행 시점에 지워지지 않는다는 것을 의미한다.
디컴파일된 결과를 보면, 아래와 같이 String 이 남아있다.

![](/image/kotlin-generics-filterIsInstance-decompile.png)

---
Kotlin In Action <드미트리 제메로프, 스베트라나 이사코바>
