---
layout: post 
title:  "Spring Web Flux : @MVC"
date:   2021-08-23
categories: Spring
---

스프링캠프 2017 에서 Toby 님이 발표한 Spring Web Flux 의 다음 내용을 정리한다.

- annotation 방식의 @MVC 와 유사한 WebFlux 개발 방법

## @MVC WebFlux : 01

annotation 방식의 @MVC 방식과 유사하면서
비동기 + 논블러킹 리액티브 스타일의 코드 작성이 가능하다.

```kotlin
@RestController
class MyController {

    @GetMapping("hello/{name}")
    fun hello(request: ServerRequest): Mono<ServerResponse> =
        ok().body(fromObject(request.pathVariable("name")))
}
```

- 요청 정보가 미리 바인딩되지 않아, ServerRequest.pathVariable() 로 가져온다.
- 응답은, Mono 에 감싸진 ServerResponse 를 응답한다.
- 상태 코드와 바디를 명시적으로 선언한다.

## @MVC WebFlux : 02

위 코드는, 더 이전 스타일로 표현 가능하다.
가장 대표적인 @MVC WebFlux 작성 방식이다.

```kotlin
@GetMapping("hello/{name}")
fun hello(@PathVariable name: String): Mono<String> =
    Mono.just("Hello $name")
```

- 파라미터 바인딩을 MVC 방식 그대로 한다.
- 그리고, 핸들러 로직의 결과를 Mono/Flux 타입으로 리턴한다.

웹 요청의 바디를 받으려면,

```kotlin
@GetMapping("hello/{name}")
fun hello(@RequestBody user: User): Mono<String> =
    Mono.just("Hello ${user.name}")
```

- MVC 방식과 동일하게, 웹 요청의 바디를 MessageConverter 에서 바인딩한다.

---
https://www.youtube.com/watch?v=2E_1yb8iLKk&t=443s
