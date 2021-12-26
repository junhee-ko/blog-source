---
layout: post 
title: map, flatMap
date:   2021-12-26
categories: Reactor
---

Project Reactor 의 "map, flatMap" transform operator 에 대해 정리한다.

## map

하난의 element 를 1-to-1 방식으로 변형한다.
공식 문서의 Mono 의 map 정의는 다음과 같다.

```
Transform the item emitted by this Mono 
by applying a synchronous function to it.
```

공식 문서의 Flux 의 map 정의는 다음과 같다.

```
Transform the items emitted by this Flux 
by applying a synchronous function to each item.
```

다응 코드를 보자.

```kotlin
fun namesFluxMapAndFilter(strLen: Long): Flux<String> {
    val names = listOf("ko", "jun", "hee")

    return Flux.fromIterable(names)
        .map { it.uppercase(Locale.getDefault()) } // HERE !!
        .filter { it.length > strLen }
}

@Test
fun namesFluxMapAndFilter() {
    // given
    val strLen = 2L

    // when
    val names: Flux<String> = fluxMonoGenerator.namesFluxMapAndFilter(strLen)

    // then
    StepVerifier.create(names)
        .expectNext("JUN", "HEE")
        .verifyComplete()
}
```

위 코드는,

1. 각 element 를 대문자로 변형한다. (map)
   "ko" -> "KO"
   "jun" -> "JUN"
   "hee" -> "HEE"
2. 길이가 2 보다 큰 element 만 추출한다. (filter)
   "JUN", "HEE"

## flatMap

하나의 element 를 1-to-n 방식으로 변형한다.
공식 문서의 Flux 의 flatMap 정의는 다음과 같다.

```
Transform the elements emitted by this Flux asynchronously into Publishers, 
then flatten these inner publishers into a single Flux through merging, 
which allow them to interleave.
```

정리하면,
1. 각 element 를 Publisher(Mono or Flux) 로 asynchronous 하게 변형한다.
2. 그리고, 각 Publisher 를 합쳐서 하나의 Flux 로 펼친다.

다음 코드를 보자.

```kotlin
fun namesFluxFlatMap(strLen: Long): Flux<String> {
    val names = listOf("ko", "jun", "hee")

    return Flux.fromIterable(names)
            .map { it.uppercase(Locale.getDefault()) }
            .filter { it.length > strLen }
            .flatMap { splitString(it) } // HERE !!
}

// "KO" -> Flux "K", "O"
private fun splitString(name: String): Flux<String> {
    val split: List<String> = name.split("").filter { it.isNotEmpty() }

    return Flux.fromIterable(split)
}

@Test
fun namesFluxFlatMap() {
    // given
    val strLen = 2L

    // when
    val names: Flux<String> = fluxMonoGenerator.namesFluxFlatMap(strLen)

    // then
    StepVerifier.create(names)
        .expectNext("J", "U", "N", "H", "E", "E")
        .verifyComplete()
}
```

위 코드는,

1. 각 element 를 대문자로 변형한다. (map)
   "ko" -> "KO"
   "jun" -> "JUN"
   "hee" -> "HEE"
2. 길이가 2 보다 큰 element 만 추출한다. (filter)
   "JUN", "HEE"
3. 각 element 를 split 해서 list 를 생성한 뒤 Flux 를 생성한다.
   "JUN" -> Flux "J", "U", "N"
   "HEE" -> Flux "H", "E", "E"
4. 3번의 결과를 하나의 Flux 로 생성한다.
   Flux "J", "U", "N", Flux "H", "E", "E" -> Flux "J", "U", "N", "H", "E", "E"

---
- https://projectreactor.io/
- Udemy, "Build Reactive MicroServices using Spring WebFlux/SpringBoot"
