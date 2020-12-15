---
layout: post
title:  "Sprint Boot Bean"
date:   2020-06-11
categories: Spring
---

Bean 이 무엇이고, Spring Boot 는 어떤 과정으로 Bean 을 등록되는지 정리한다.

## Bean 이란

IoC (Inversion of Control) Container 가 관리하는 객체이다.
IoC Container 는 객체의 생성을 책임지고 의존성을 관리한다. Spring 에서는 ApplicaitonContext 가 IoC Container 역할을 한다. ApplicaitonContext 는 BeanFactory 를 확장한다.

![](/image/spring-boot-bean-01.png)

## Bean 등록 순서

다음 순으로 Bean 이 등록된다. 

1. @ComponentScan
   최상위 패키지 이하의 모든 패키지에 있는 클래스에, @Component 가 붙어있으면 Bean 으로 등록한다.
   예를 들어, 다음과 같이 repository package 와 service package 에 각각 @Repository, @Service 가 붙은 클래스는 Bean 으로 등록된다. @Repository, @Service 도 결국 @Component 가 붙어있기 때문이다.
   ( Bean 등록 대상은, IDE 에서 클래스 옆에 콩 모양의 이미지를 보여준다. )

   ![](/image/spring-boot-bean-05.png)

2. @EnableAutoConfiguration
   **spring-boot-autoconfigure** 프로젝트의  META-INF 에 spring.factories 라는 파일이 있다.
   이 파일 안의 key 값 중에 **EnableAutoConfiguratoin** 이라는 키가 있는데, 이 키에 여러 Configuration 클래스가 정의되어있다.
   
   ![](/image/spring-boot-bean-02.png)

   그리고 각 Configuration 클래스에는 @Configuration 이 붙어있다.
   예를 들어, WebMvcAutoConfigurration 를 따라가보면 결국에는 @Component 가 붙어 있기 때문에 Bean 으로 등록된다.

   ![](/image/spring-boot-bean-03.png)

   ![](/image/spring-boot-bean-04.png)