---
layout: post 
title:  "지연 계산 컬렉션"
date:   2021-08-15 
categories: Kotlin
---

lazy collection 연산에 대해 정리한다.

## 지연 계산 (lazy) 켈렉션 연산

다음 예를 보자.

```kotlin
people
    .map(Person::name)
    .filter { it.startsWith("A") }
```

filter 와 map 은 리스트를 반환한다. 
즉 위 코드는, 한 리스트는 filter 의 결과를 담고 다른 리스트는 map 의 결과를 담는다. 
원소의 개수가 수백만 개라면 효율이 떨어질 수 있다.

그래서, 각 연산이 컬렉션을 직접 사용하는 대신 시퀀스를 사용하게 만들어 효율을 높일 수 있다.
```kotlin
people.asSequence()     // 원본 컬렉션을 시퀀스로 변환
    .map(Person::name)
    .filter { it.startsWith("A") }
    .toList()           // 결과 시퀀스를 다시 리스트로 변환  
```

## 시퀀스 연산 실행 : 중간 연산, 최종 연산

자바 8의 스트림과 코틀린의 시퀀스의 개념은 같다.

중간 연산은 다른 시퀀스를 반환한다. 
최종 연산은 결과를 반환한다.

다음 코드를 다시 보자.

```kotlin
people.asSequence()
    .map(Person::name)              // 중간 연산
    .filter { it.startsWith("A") }  // 중간 연산
    .toList()                       // 최종 연산
```

중간 연산은 항상 지연 계산된다. 
즉, 최종 연산이 호출될 때 적용 된다는 뜻이다.

다음 코드를 보자. 
실행하면, 아무 내용도 출력되지 않는다.

```kotlin
listOf(1, 2, 3, 4).asSequence()
    .map {
        println("map $it")
        it * it
    }
    .filter {
        println("filter $it")
        it % 2 == 0
    }
```

아래와 같이, toList() 를 추가하면 모든 계산이 수행된다.

```kotlin
listOf(1, 2, 3, 4).asSequence()
    .map {
        println("map $it")
        it * it
    }
    .filter {
        println("filter $it")
        it % 2 == 0
    }
    .toList()

```

다른 예를 보자.
1. map 으로 리스트의 각 숫자를 제곱하고, 
2. 제곱한 숫자 중에 find 로 3 보다 큰 첫 번째 원소를 찾자.

```kotlin
val lazy = listOf(1, 2, 3, 4).asSequence()
    .map { it * it }
    .find { it > 3 }
println(lazy)

val eager = listOf(1, 2, 3, 4)
    .map { it * it }
    .find { it > 3 }
println(eager)
```

위 코드의 결과는 모두 4 이지만, 즉시 계산 (컬렉션 사용) 과 지연 계산 (시퀀스 사용) 의 차이를 보여준다.

컬렉션을 사용하면,
1. 리스트가 다른 리스트로 일단 모두 변환 된다. 
2. 그래서 map 연산의 결과는 9, 16 을 포함한다. 
3. 그리고, find 가 술어를 만족하는 첫 번 째 원소인 4 를 찾는다.

시퀀스를 사용하면, 
1. find 호출이 원소를 하나씩 처리하기 시작한다. 
2. 1 을 가져와서 map 에 지정된 변환을 수행하고 find 에 지정된 술어를 만족하는지 확인한다. 
3. 2 를 가져와서 map 에 지정된 변환을 수행하고 find 에 지정된 술어를 만족하는지 확인한다. 
4. 만족한다. 그래서 결과로 반한한다. 
5. 그러면, 답을 찾았기 때문에 3, 4 를 처리할 필요가 없다.

---
Kotlin In Action <드미트리 제메로프, 스베트라나 이사코바>
