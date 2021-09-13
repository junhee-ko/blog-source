---
layout: post 
title:  컬렉션과 배열 
date:   2021-09-13 
categories: Kotlin
---

코틀린의 컬렉션과 자바의 컬렉션 간의 관계에 대해 정리한다.

## 널 가능성과 컬렉션

타입 인자로 쓰인 타입에 ? 표시를 붙이면 널을 저장할 수 있다. 
아래 예제에서는, List<Int?> 는 Int 나 null 을 저장할 수 있다.

```kotlin
fun readNumbers(reader: BufferedReader): List<Int?> {

    val result = ArrayList<Int?>()

    for (line in reader.lineSequence()) {
        try {
            val number = line.toInt()
            result.add(number)
        } catch (e: NumberFormatException) {
            result.add(null)
        }
    }

    return result
}
```

"어떤 변수 타입의 널 가능성" 과 "타입 파라미터로 쓰인 타입의 널 가능성" 은 차이가 있다. 
예를 들어,

1. List<Int?> : 리스트 안의 각 값이 널이 될 수 잇다.
2. List<Int>? : 전체 리스트가 널이 될 수 있다.

## 읽기 전용과 변경 가능한 컬렉션

코틀린 컬렉션과 자바 컬렉션의 중요한 차이 중 하나는, 
코틀린에서는 컬렉션 안의 데이터에 접근하는 인터페이스와 변경하는 인터페이스가 분리되었다는 점이다.

1. kotlin.collections.Collection : 컬렉션 안의 데이터에 접근
2. kotlin.collections.MutableCollection : 데이터 변경 (kotlin.collections.Collection 를 확장)

아래 코드에서, target 에 해당하는 인자로 읽기 전용 컬랙션을 넘길 수 없다.

```kotlin
fun <T> copyElements(source: Collection<T>, target: MutableCollection<T>) {
    for (item in source) {
        target.add(item)
    }
}
```

## 코를린 컬렉션과 자바

자바는 읽기 전용 컬렉션과 변경 가능 컬렉션을 구분하지 않는다. 
그래서, 코틀린에서 읽기 전용 컬렉션으로 선언된 객체여도 자바에서는 그 컬렉션의 내용을 변경할 수 있다.

다음은, 자바 코드이다.

```java
public class CollectionUtils {
    public static List<String> uppercaseAll(List<String> items) {
        for (int i = 0; i < items.size(); i++) {
            items.set(i, items.get(i).toUpperCase());
        }

        return items;
    }
}
```

다음은, 위 자바 코드를 호출하는 코틀린 코드이다.

```kotlin
fun printlnUppercase(list: List<String>) {
    println(CollectionUtils.uppercaseAll(list))
    println(list.first())
}
```

읽기 전용 list 임에도 불구하고, 컬렉션을 변경하는 자바 함수를 호출하면 컬렉션이 변경이 된다.

## 배열

코틀린 배열은 타입 파라미터를 받는 클래스이다.
배열의 원소 타입은 타입 파라미터에 의해 정해진다.

배열은 생성하는 다음 예제를 보자.

```kotlin
val letters = Array<String>(26){i -> ('a'+i).toString()}
```

람다는 배열의 인덱스를 인자로 받아, 해당 위치에 들어갈 원소를 반환한다.

코틀린은 원시 타입의 배열을 표현하는 별도 클래스를, 각 원시 타입마다 제공한다.
( IntArray, ByteArray, CharArray, BooleanArray... )
이 모든 타입은 자바 원시 타입 배열인 int[], byte[], char[] 로 컴파일된다.

```kotlin
val fiveZeros1 = IntArray(5)
val fiveZeros2 = intArrayOf(0, 0, 0, 0, 0)
```

---
Kotlin In Action <드미트리 제메로프, 스베트라나 이사코바>
