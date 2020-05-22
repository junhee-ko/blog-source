---
layout: post
title:  "Spring MVC 요청 처리 과정"
date:   2020-05-22
categories: Spring
---

Spring MVC 는 어떻게 Client 의 요청을 처리하는지에 대해 정리한다.

## DispatcherServlet

DispatchServlet 은 Client 의 요청에 대해 실제 처리하는 메서드를 호출해주는 Front Controller 역할을 하는 클래스이다.

계층 구조는 다음과 같다.

![](/image/spring_mvc_01.png)

이제, DispatchServlet 이 Client 의 요청을 받은 뒤에, 어떻게 요청을 처리할 Controller 를 찾고 응답하는지 알아보자.

## Client 요청 처리 흐름

전체 흐름은 다음과 같다.

1. Application 이 구동될 때, Spring Boot **Auto Configure** 로 Spring MVC 의 Bean 들이 자동 등록된다.
2. 사용자의 요청이 들어오면, DispatcherServlet 의 **doService()** 가 실행된다.
3. doService() 에서 여러 설정을 한 뒤, **doDispatch()** 가 실행이 된다.
4. **HandlerMapping** 이 요청을 처리할 Handler 를 찾는다.
5. 등록되어 있는 **HandlerAdapter** 중에 해당 Handler 를 실행할 Adapter 를 찾는다.
6. HandlerAdapter 는 Handler 의 응답을 처리한다.
7. 최종적으로 응답을 보낸다.



디버거를 이용해서, 위 흐름을 코드로 상세히 살펴보자.

1. Client 가 GET 요청을 하면, FrameworkServlet 의 doGet() 이 실행된다.

   ![](/image/spring_mvc_02.png)

2.  processRequest() 의 doService() 에 들어가자.  

   ![](/image/spring_mvc_03.png)

3. DispatcherServlet 의 doService 메서드임을 알 수 있다.

   여기서, handlerMappings 나 hadnlerAdapterts 클래스들은 아래와 같이 주입된 것이 확인된다.

   ![](/image/spring_mvc_04.png)

   이 클래스들은, Spring Boot Auto Configure 로 인해서, WebMvcAutoConfiguration 에서 Bean 으로 등록된다. 

   다음과 같이 WebMvcAutoConfiguration 에서 여러 Bean 들이 등록되는 것을 알 수 있다.

   ![](/image/spring_mvc_05.png)

4. 이제 다시, doSerivce() 에서 doDispatch() 에 들어가보자.

   ![](/image/spring_mvc_06.png)

   다음과 같이,

   - 요청을 처리할 핸들러를 찾고 (by HandlerMappings)
   - 해당 핸드러를 실행할 수 있는 HandlerAdapter 를 찾는다.
   - 찾아낸 HandlerAdapter 를 이용해서 handle 한다.

   ![](/image/spring_mvc_07.png)
