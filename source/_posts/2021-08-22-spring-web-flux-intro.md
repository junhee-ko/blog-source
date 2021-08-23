---
layout: post 
title:  "Spring Web Flux : 소개"
date:   2021-08-22 
categories: Spring
---

스프링캠프 2017 에서 Toby 님이 발표한 Spring Web Flux 의 다음 내용을 정리한다.

- WebFlux 사용 이유
- WebFlux 개발 방식
- WebFlux 의 주요 특징

## 사용 이유

Thread, CPU, Memory 등 자원을 낭비하지 않고 더 많은 요청을 처리할 수 있는 고성능 Web Application 을 만들 수 있다.
이런 부분에서 효율성을 극대화할 수 있는 경우는, 서비스 간 호출이 많은 Microservice Architecture 가 있다.

## 개발 방식

두 가지 개발 방식을 지원한다.

1. 기존의 @MVC 방식 : @Controller, @RequestMapping 등 annotation 을 사용한다.
2. 함수형 모델 : annotation 에 의지하지 않고, RouterFunction 과 HandlerFunction 를 사용한다.

## 주요 특징

1. Servlet 기반이 아니다. (서블릿 지원하는 컨테이너에서 동작할 수 있게 호환성은 가지고 있음)
2. ServerRequest, ServerResponse 을 사용한다. (HTTP Request, Response 를 추상화한 새로운 모델) 

## 지원하는 웹 서버 컨테이너

- Tomcat, Jetty : 서블릿의 기존 동기-블로킹 방식을 사용하지 않고, 서블릿  3.1+ 의 비동기 논블로킹 요청 처리 방식을 이용한다.
- Netty, Undertow : 서블릿과 상관 없는 비동기-논블로킹 IO 웹 서버에서 동작한다.

---
https://www.youtube.com/watch?v=2E_1yb8iLKk&t=443s
