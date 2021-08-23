---
layout: post 
title:  "Spring Web Flux : Reactive"
date:   2021-08-24
categories: Spring
---

스프링캠프 2017 에서 Toby 님이 발표한 Spring Web Flux 의 다음 내용을 정리한다.

- WebFlux 를 사용하며 개선할 포인트
- Data Access Repository
- Non Blocking API call

## Blocking IO

WebFlux 를 사용하는데, 서비스 로직에서 Blocking IO 가 사용되는 코드가 많으면 성능이 오히려 나빠질 수 있다.

개선할 blocking IO 는 다음과 같은 것들이 있다.
1. data access repository 를 호출
2. HTTP API 호출
3. 기타 네트워크를 이용하는 서비스

## JDBC 기반 RDB 연결

JDBC API 는 아쉽게도, Blocking 메서드로 점철되었다.
이 부분을 다음과 같이 개선해서, 서블릿 스레드를 점유하지 않도록 만들 수 있다.

- @Async 비동기를 적용해서, 
- CompletableFuture 리턴하게 하여 
- 서블릿 스레드만 빨리 스레드풀에 리턴하게 함으로써 가용한 자원으로 만들 수 있다.

예를 들어,
repository 의 메서드에 다음과 같이 @Async 를 붙인다.

```kotlin
@Async
fun findOneByFirstname(firstname: String): CompletableFuture<User>
```

위 메서드를 호출하는 쪽에서는,

```kotlin
fun findUser(name: String): Mono<User> = 
    Mono.fromCompletionStage(userRepository.findOneByFirstname(name))
```

## Reactive Data Access 기술

Spring Data 2.0 부터 추가된 다음 기술을 통해서, 
DB access 하는 코드를 완벽하게 Non blocking 으로 만들 수 있다.

1. ReactiveCrudRepository 확장
2. Spring Data 의 Reactive Repository 이용 (MongoDB, Casandra, Redis...)

```kotlin
interface ReactivePersonRepository : ReactiveCrudRepository<Person, String> {
    fun findByLastname(lastname: Mono<String>): Flux<Person>

    @Query("{ 'firstname': ?0, 'lastname': ?1}")
    fun findByFirstnameAndLastname(firstname: String, lastname: String): Mono<Person>
}
```

## WebClient

Non Blocking API Call 은 WebClient 를 이용하면 된다.
WebClient 는 AsyncRestTemplate 의 Reactive version 이다.
요청을 Mono/Flux 로 전달할 수 있고, 응답을 Mono/Flux 형태로 가져온다.

다음 코드를 보자.

```kotlin
@GetMapping("hello/webclient")
fun helloWithWebClient(): Mono<String> =
    WebClient.create("http://localhost:8080")
        .get()
        .uri("/hello/{name}", "Spring")
        .accept(MediaType.TEXT_PLAIN)
        .exchange()
        .flatMap { it.bodyToMono(String::class.java) }
        .map { it.toUpperCase() }
        .flatMap { userRepository.save(it) }
```

위 코드는, 
1. 요청을 받아 
2. API 호출을 한 뒤에,
3. 그 결과를 대문자로 바꿔서
4. DB 에 저장하고
5. 리턴한다.

그리고, exchange 는 요청을 응답으로 바꾸는 메서드이다.
즉, 서버에 실제적으로 API 호출을 하고 그 결과를 받아와 리턴한다.

## 성능 향상

Async + Non blocking 리팩티브 웹 애플리케이션의 효과를 얻으려면

1. 코드에서 blocking 이 발생하지 않도록 한다.
Flux or Mono 에 데이터를 넣어서 전달하면 된다.

2. WebFlux 와 다음 기술들을 결합하여 사용한다
Reactive Repository, Reactive 원격 API 호출, Reactive 지원 외부 서비스, @Async Blocking IO

---
https://www.youtube.com/watch?v=2E_1yb8iLKk&t=443s
