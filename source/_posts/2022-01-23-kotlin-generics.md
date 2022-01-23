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

## 제네릭 타입 파마리터

제네릭스를 사용하면, 타입 파마리터를 받는 타입을 정의할 수 있다.
제네릭 타입의 인스턴스를 만들려면, 타입 파라미터를 구체적인 타입 인자로 치환해야한다.

예를 들어, Map 클래스는 키 타입과 값 타입을 타입 파리미터로 받으므로 Map<K, V> 이다.
이 제네릭 클래스에 대해 제네릭 타입의 인스턴스를 만들려면, 
Map<String, Person> 처럼 구체적인 타입 인자로 치환해서 인스턴스화할 수 있다.

### 타입 파라미터 추론

코틀린 컴파일러는 보통의 타입과 마찬가지로, 타입 파라미터도 추론가능하다.
아래에서는, listOf 에 전달된 두 값이 문자열이므로 여기서 생기는 리스트가 `List<String>` 임을 추론한다.

```kotlin
val names = listOf("junhee", "ko")
```

아래에서는, 빈 리스트를 만든다면 타입 인자를 추론할수 없어서 직접 타입 인자를 명시해야한다.

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

val slice1 = letters.slice<Char>(0..2) // 타입 인자 명시적으로 지정
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
기반 타입의 제네릭 파라미터에 대해 타입 인자를 지정해야한다.
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

타입 파라미터 제약은 클래스나 함수에 사용하는 타입 인자를 제한한다.
제약을 가하려면 타입 파라미터 이름 뒤에 콜론(:) 을 표시하고 그 뒤에 상한 타입을 적으면 된다.

아래 예를 보자.

![](/image/kotlin-generics-constraint.png)

아래 호출은 실제 타입 인자인 Int 가 Number 를 확장하므로, 정상 호출된다.

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
하지만, 실제로 T 에 해당하는 타입 인자로 널이 될 수 있는 타입을 넘길 수 있다.

항상 널이 될 수 없는 타입만 타입 인자로 받기위해서는, 
Any 를 상한으로 지정하는 제약을 가하면 된다.

```kotlin
class Processor<T: Any> {   // 항상 널이 될 수 없는 타입만 인자로 받음
    
    fun process(value: T){
        value.hashCode()
    }
}

fun processorTest(){
    val processor = Processor<String>()
    processor.process("jko")
}
```

---
Kotlin In Action <드미트리 제메로프, 스베트라나 이사코바>
