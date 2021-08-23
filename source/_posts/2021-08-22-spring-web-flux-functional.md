---
layout: post 
title:  "Spring Web Flux : Functional"
date:   2021-08-22 
categories: Spring
---

스프링캠프 2017 에서 Toby 님이 발표한 Spring Web Flux 의 다음 내용을 정리한다.

- 함수형 스타일의 WebFlux 가 웹 요청을 처리하는 방식
- 함수형 스타일의 WebFlux 장/단점

## Spring 이 웹 요청을 처리하는 방식

다음 순서로 진행된다.

1. 요청 매핑
웹 요청을 어느 handler 에게 보낼지 결정한다. MVC 에서는 @RequestMapping 을 이용한다.

2. 요청 바인딩
handler 에 전달할 웹 요청을 준비한다.
(URL Path, Header, Cookie 정보를 가져옴, body 의 json 을 자바 오브젝트로 바인딩...)

3. 핸들러 실행
전달 받은 요청 정보를 이용해 로직을 수행한다.

4. 핸들러 결과 처리
handler 의 return 값으로 웹 응답으로 변환한다.

전통적인 방식의 다음 코드를 보자.

```kotlin
@RestController
class MyController {

    @GetMapping("hello/{name}")
    fun hello(@PathVariable name: String): String {
        return "Hello $name"
    }
}
```

@GetMapping 을 통해 요청을 매핑하고, @PathVariable 을 통해 요청을 바인딩한다.
그리고, return "Hello $name" 에서 핸들러를 실행하고 결과를 처리한다.  

참고로, 
- 기존에는 @RequestMapping 과 메서드 타입을 사용했는데, Spring 4.xx 에서 @GetMapping, @PostMapping.. 이 등장했다.
- 또한, @RestController 이므로 response body 에 문자열이 그대로 들어간다. 
- 특별한 Content-Type 을 지정하지 않으면, text/plain 으로 지정된다.

## 함수형 WebFlux 가 웹 요청을 처리하는 방식

다음 순서로 진행된다.

1. 요청 매핑 : RouterFunction
2. 요청 바인딩 : HandlerFunction
3. 핸들러 실행 : HandlerFunction
4. 핸들러 결과 처리 : HandlerFunction

요청 매핑에, RouterFunction 이라는 Functional Interface 가 사용된다.
요청 바인딩, 핸들러 실행, 결과 처리에는 HandlerFunction 이 사용된다.

RouterFunction 은 다음과 같이 정의되어 있다.

```java
@FunctionalInterface
public interface RouterFunction<T extends ServerResponse> {

    Mono<HandlerFunction<T>> route(ServerRequest request);
}
```

Functional Interface 이기 때문에, 람다식으로 표현이 가능하다.

route 메서드에서, 
- ServerRequest 는 Webflux 의 웹 요청이고,
- Mono<HandlerFunction<T>> 는 ServerResponse 를 리턴하는 HandlerFunction 이다.

결국 이 route 는, 서버 요청을 받아서 HandlerFunction 을 찾아준다.
여기서 이 HandlerFunction 은 Controller 의 method 에 대응된다.

## 함수형 스타일로 변형

아래 코드는 전통적인 @MVC 방식이다.

```kotlin
@RestController
class MyController {

    @GetMapping("hello/{name}")
    fun hello(@PathVariable name: String): String {
        return "Hello $name"
    }
}
```

이제 위 코드를, Webflux 함수형 스타일로 바꿔보자.
함수 2개를 작성하면 된다. 

- HandlerFunction
- RouterFunction

### HandlerFunction

HandlerFunction 을 먼저 작성해보자.

```kotlin
val helloHandler: HandlerFunction<*> = HandlerFunction { request: ServerRequest ->
    val name: String = request.pathVariable("name")
    val result: Mono<String> = Mono.just("Hello $name")
    val response: Mono<ServerResponse> = ServerResponse.ok().body(result, String::class.java)

    response
}
```

1. ServerRequest.pathVariable() 로 {name} 을 추출하고
2. 로직 적용 후 결과 값을 모노에 담아,
3. 웹 응답을 ServerResponse 의 Builder 를 활용해서 만든다.
4. 그리고, Mono 에 단긴 ServerResponse 타입으로 리턴한다.

위 코드는 다음과 같이 개선 가능하다.

```kotlin
val helloHandler = HandlerFunction { request ->
    ok().body(fromObject("Hello ${request.pathVariable("name")}"))
}
```

BodyInserters.fromObject() 의 도움을 받아 Mono 에 담고 있다.

위의 코드에서 볼 수 있듯이, 우리가 직접 request.pathVariable() 로 값을 꺼내오고 있다. 
@MVC 에서는 @PathVariable 이 붙어 있으면, 스프링이 관례에 따라 값을 넣어서 메스드를 호출해준다.
우리가 직접 path 에서 값을 꺼내오는 코드를 만들지 않는다.
함수형에서는 명시적으로 가져와야하는 코드가 필요하다.

### RouterFunction

RouterFunction 을 작성해보자.

```kotlin
val routerFunction: RouterFunction<ServerResponse> = RouterFunction { request: ServerRequest ->
    if (RequestPredicates.path("hello/{name}").test(request))
        Mono.just(helloHandler)
    else
        Mono.empty()
}
```

hello/{name} 과 호환되는 요청인지 테스트해보고 결과 true 이면, 
위에서 만든 helloHandler 를 Mono 넣어서 리턴하고 있다.

### HandlerFunction + RouterFunction

HandlerFunction 과 RouterFunction 을 조합할 수 있다.

```kotlin
val routerFunctionWithHandler: RouterFunction<ServerResponse> = RouterFunctions.route(
    RequestPredicates.path("hello/{name}"),
    HandlerFunction { request: ServerRequest ->
        ok().body(fromObject("Hello ${request.pathVariable("name")}"))
    }
)
```

RouterFunctions.route static method 의 
- 첫 번째 파리미터에는, 매핑 조건을 체크한는 RequestPredicate 을 전달하고,
- 두 번재 파라미터에는, HandlerFunction 을 그대로 전달하고 있다.

위 코드는, 아래와 같이 조금 더 개선 가능하다.

```kotlin
val routerFunctionWithHandler = RouterFunctions.route(RequestPredicates.path("hello/{name}")) { request ->
    ok().body(fromObject("Hello ${request.pathVariable("name")}"))
}
```

## RouterFunction 등록

그런데, 스프링 컨테이너는 요청이 들어왔을 때 RouterFunction 을 거쳐 HandlerFunction 을 어떻게 실행하게 할까 ?
RouterFunction 을 @Bean 으로 등록하면 된다.

```kotlin
@Configuration
class HelloRouter {

    @Bean
    fun route(): RouterFunction<ServerResponse> =
        RouterFunctions
            .route(RequestPredicates.path("hello/{name}")) { request ->
                ServerResponse
                    .ok()
                    .body(BodyInserters.fromObject("Hello ${request.pathVariable("name")}"))
            }
}
```

handler 의 로직이 복잡하다면, 아래와 같이 handler 를 추출할 수 있다.

```kotlin
@Configuration
class HelloRouter {

    @Bean
    fun route(): RouterFunction<ServerResponse> {
        // Here !!
        val helloHandler = HandlerFunction { request ->
            ServerResponse.ok().body(BodyInserters.fromObject("Hello ${request.pathVariable("name")}"))
        }

        return RouterFunctions
            .route(RequestPredicates.path("hello/{name}"), helloHandler)
    }
}
```

Handler 를 별개의 클래스로 정의하면 더 간결해질 수 있다.

```kotlin
@Component
class HelloHandler(val helloService: HelloService) {

    fun hello(request: ServerRequest): Mono<ServerResponse> {
        val res = helloService.hello(request.pathVariable("name"))
        
        return ok().body(fromObject(res))
    }
}
```

이렇게, HelloHandler 를 별개의 클래스로 정의하면,
HelloHandler 클래스 안에 HandlerFunction 에 mapping 될 수 있는 메서드들을 정의할 수 있다.

그리고 이제, Router 에서는 정의한 Handler 를 호출한다.

```kotlin
@Configuration
class HelloRouter {

    @Bean
    fun route(@Autowired helloHandler: HelloHandler): RouterFunction<ServerResponse> =
        RouterFunctions
            .route(path("hello/{name}"), helloHandler::hello) // Here !!
}
```

### RouterFunction 중첩

하나의 Bean 에 n 개의 RouterFunction 선언이 가능하다.
and(), andRoute() 를 사용하면 된다.

또한, 
RouterFunction.nest() 사용하면, @RequestMapping 처럼 공통의 조건을 정의할 수 있다.

아래 코드를 보자.

```kotlin
    @Bean
    fun routeMany(@Autowired personHandler: PersonHandler): RouterFunction<ServerResponse> {
        return nest(
            path("/person"),
            nest(
                accept(APPLICATION_JSON),
                route(GET("/{id}"), personHandler::getPerson) // 1
                    .andRoute(method(HttpMethod.GET), personHandler::listPeople) //2
            ).andRoute(
                POST("/").and(contentType(APPLICATION_JSON)), personHandler::createPerson // 3
            )
        )
    }
```

세 개의 HandlerFunction 을 매핑하고 있다.

1. getPerson : /person prefix && accept Header 가 APPLICATION_JSON && /person/{id}
2. listPeople : /person prefix && accept Header 가 APPLICATION_JSON && GET HTTP method
3. createPerson: /person prefix && POST HTTP Method && content type 이 APPLICATION_JSON

## 함수형 스타일의 WebFlux 장/단점

### 장점

1. 정확한 타입 체크, 잘못된 코드 작성 오류 최소화
기존 @MVC 에서는, @RequestMapping 같은 annotation 을 달아 선언에 따른 관례들이 조합이 되어서 바인딩되는 결과를 얻는다.
함수형 스타일의 WebFlux 에서는, 모든 웹 요청 처리 작업을 명시적으로 코드로 작성한다.
그래서, 정확한 타입 체크가 가능하고 관례를 혼동해서 작성되는 잘못된 코드 작성을 막는다.

2. 추상화와 확장에 유리
함수 형태로 메서드를 채이닝하기 때문에, 추상화하거나 프레임워크화가 가능하고, 확장에 유리하다.

3. 테스트 작성 편리함
기존 @MVC 에서는, return 되는 body 의 JSON 을 테스트하기 위해, 컨네이너를 띄어 사실상 통합 웹 테스트 발생한다. 그래서, 테스트하는데 시간이 오래 걸린다.
반면, 함수형 스타일에서는 컨테이너를 띄우지 않고, handler 로직은 물론이고 요청 매핑과 리턴 값 처리까지 단위 테스트로 가능하다.

### 단점

함수형 스타일의 코드 작성이 편하지 않으면, 코드 작성과 이해가 어렵다.

---
https://www.youtube.com/watch?v=2E_1yb8iLKk&t=443s
