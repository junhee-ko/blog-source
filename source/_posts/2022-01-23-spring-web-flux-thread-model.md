---
layout: post 
title: Spring WebFlux Thread Model
date: 2022-01-23
categories: Spring
---

Spring Webflux 에서는 어떤 thread 들이 어떻게 request 를 처리하는지 정리한다.

## Threads: reactor-http-nio

```text
On a “vanilla” Spring WebFlux server (for example, no data access nor other optional dependencies), 
you can expect one thread for the server and several others for request processing (typically as many as the number of CPU cores). 
Servlet containers, however, may start with more threads (for example, 10 on Tomcat), 
in support of both servlet (blocking) I/O and servlet 3.1 (non-blocking) I/O usage.
```

Spring 공식 문서에 따르면, 
부가적인 dependency 가 없는 Spring WebFlux 서버는, 

1. 서버를 위한 thread 하나와
2. 요청 처리를 위한 여러 thread 들로 구성된다.

`spring-boot-starter-webflux` dependency 만 추가해서 project 하나를 만들어보자.

![](/image/spring-webflux-init.png)

그리고, 아래와 같이 '1,2,3' 을 응답하는 간단한 endpoint 를 추가하자.

```kotlin
@RestController
@RequestMapping
class NumbersController {

    private val logger = LoggerFactory.getLogger(this::class.java)

    @GetMapping("/local")
    fun getNumbers(): Flux<Int> {
        logger.info("-- getNumbers")

        return Flux.fromIterable(listOf(1, 2, 3))
    }
}
```

App 을 실행시키고, 해당 endpoint 를 호출해보자.

```shell
curl http://localhost:8080/local
```

요청 처리를 reactor-http-nio-2 스레드가 담당하는 것을 알 수 있다.

![](/image/spring-webflux-worker-thread.png)

## Worker Tread Count

Thread Pool 의 worker thread 개수는, availableProcessors 의 개수와 동일하다.
availableProcessors 의 개수를 출력해보자.

```kotlin
@SpringBootApplication
class Application

private val logger = LoggerFactory.getLogger(Application::class.java)

fun main(args: Array<String>) {
    runApplication<Application>(*args)

    logger.info("CPU: ${Runtime.getRuntime().availableProcessors()}")
}
```

출력 결과는 다음과 같다.

![](/image/spring-webflux-available-processor.png)

위에서 만든 endpoint 로 여러 요청을 해보자.
아래와 같이, 16개의 worker thread 가 요청 처리를 하고 있는 것을 알 수 있다.

![](/image/spring-webflux-worker-thread-test.png)

## WebClient Threads

```text
The reactive WebClient operates in event loop style. 
So you can see a small, fixed number of processing threads related to that (for example, reactor-http-nio- with the Reactor Netty connector). 
However, if Reactor Netty is used for both client and server, the two share event loop resources by default.
```

Spring 공식 문서에 따르면, WebClient 는 이벤트 루프 스타일로 동작한다.
다음 endpoint 를 추가해보자.

```kotlin
@GetMapping("/local/remote")
fun getNumbersMergedWithRemote(): Flux<Int> {
    logger.info("-- Start getNumbersMergedWithRemote")

    val numbers: Flux<Int> = Flux.fromIterable(listOf(1, 2, 3))
    val numbersFromRemote: Flux<Int> = webClient.get()
        .uri("/remote")
        .retrieve()
        .bodyToFlux<Int>()
        .log()

    return numbers.concatWith(numbersFromRemote)
}
```

위 endpoint 를 호출했을 대, 출력되는 결과는 다음과 같다.

![](/image/spring-webflux-worker-thread-non-blocking.png)

요청 처리는 reactor-http-nio-4 가 담당하고 있다.
그리고 reactor-http-nio-4 에서 subscribes 를 한다.
하지만, 결과는 다른 스레드인 reactor-http-nio-2 에 publish 되고 있다.

---
- https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-concurrency-model
- https://piotrminkowski.com/2020/03/30/a-deep-dive-into-spring-webflux-threading-model/
- https://github.com/junhee-ko/spring-webflux-thread-model
