---
layout: post
title: Bean Validation
date: 2022-04-09
categories: Spring
---

Bean Validation 은, JSR 303 에서 처음으로 제안된 어노테이션 기반의 Java Bean 검증 명세이다.
대표적인 구현체로 Hibernate Validator 가 있다.
spring-boot-starter-validation 의존성을 명시하면, hibernate-validator 의존성이 추가된다.

## Bean Validation

Bean Validation 은 JSR 303 에서 처음으로 제안되었다.
최초에 제안된 명세의 내용은 다음과 같다.

```text
Validating data is a common task that is copied in many different layers of an application, from the presentation tier to the persistence layer.
Many times the exact same validations will have to be implemented in each separate validation framework, proving time consuming and error-prone.
To prevent having to re-implement these validations at each layer, many developers will bundle validations directly into their classes, cluttering them with copied validation code that is, in fact, meta-data about the class itself.

This JSR will define a meta-data model and API for JavaBean validation.
The default meta-data source will be annotations, with the ability to override and extend the meta-data through the use of XML validation descriptors.
It is expected that the common cases will be easily accomplished using the annotations, while more complex validations or context-aware validation configuration will be available in the XML validation descriptors.
The validation API developed by this JSR will not be specific to any one tier or programming model.
It will specifically not be tied to either the web tier or the persistence tier, and will be available for both server-side application programming, as well as rich client Swing application developers.
This API is seen as a general extension to the JavaBeans object model, and as such is expected to be used as a core component in other specifications, such as JSF, JPA, and Bean Binding.
```

위 내용을 정리하면 다음과 같다.

- 데이터 검증은, 어플리케이션의 서로 다른 레이어에서 (from the presentation tier to the persistence layer) 반복적으로 수행되는 작업이다.
- 각 계층에서 이러한 데이터 검증이 재구현되는 것을 피하기 위해, 많은 개발자들은 검증 코드를 클래스 안에 포함시킨다.
- 이 검증 코드는 해당 클래스의 meta-data (데이터에 대한 정보를 제공하는 데이터) 이다.
- JSR 303은, JavaBean validation 을 위한 meta-data 모델과 API 를 정의한다.
- 기본적인 meta-data 는 오버라이드 및 확장이 가능한 어노테이션 기반이다.

## Hibernate Validator

Bean Validation 스팩은 현재 2.0 까지 제안되었다.

1. JSR 303 (Bean Validation)
   ![](/image/jsr303.png)
2. JSR 349 (Bean Validation 1.1)
   ![](/image/jsr349.png)
3. JSR 380 (Bean Validation 2.0)
   ![](/image/jsr380.png)

그런데 위 JSR 은 스팩일 뿐, 구현체는 아니다.
검증된 구현체로는 Hibernate Validator 가 있다.
각 JSR 에 따라, Hibernate Validator 의 버젼은 다음과 같다.

1. JSR 303 (Bean Validation) - 4.3.1.final
2. JSR 349 (Bean Validation 1.1) - 5.1.1.final
3. JSR 380 (Bean Validation 2.0) - 6.0.1.final

## Spring Boot with Hibernate Validator

Spring Boot 에서 bean validation 을 활용하는 간단한 예를 보자.
다음과 같이 spring-boot-starter-validation 의존성을 추가하자.

![](/image/depdendency-spring-boot-starter-validation.png)

그러면, 아래처럼 hibernate-validator 가 추가된는 것이 확인된다.

![](/image/depdendency-hibernate-validator.png)

다음과 같이 사용자를 등록하기 위한 end-point 를 작성해보자.

```kotlin
import org.springframework.web.bind.annotation.PostMapping
import org.springframework.web.bind.annotation.RequestBody
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.RestController

@RestController
@RequestMapping
class UserController {

  @PostMapping("/users")
  fun createUser(@RequestBody userSignUp: UserSignUp): String {

    return "success"
  }
}

data class UserSignUp(
  val name: String,
  val age: Long
)
```

다음과 같이 나이를 10000 으로 입력해도, 정상적으로 "success" 가 응답이 된다.

![](/image/without-bean-validation.png)

아래와 같이, 최대 나이를 100 살로 제한해보자.

```kotlin
import org.springframework.web.bind.annotation.PostMapping
import org.springframework.web.bind.annotation.RequestBody
import org.springframework.web.bind.annotation.RequestMapping
import org.springframework.web.bind.annotation.RestController
import javax.validation.Valid
import javax.validation.constraints.Max

@RestController
@RequestMapping
class UserController {

  @PostMapping("/users")
  fun createUser(@Valid @RequestBody userSignUp: UserSignUp): String {

    return "success"
  }
}

data class UserSignUp(
  val name: String,

  @field:Max(value = 100)
  val age: Long
)
```

다음과 같이 400 error 를 응답받게 된다.

![](/image/with-bean-validation.png)

validation 이 실패했고, age 가 100 이하이어야 한다고 WARN 로깅이 되고 있다.

![](/image/with-bean-validation-fail-log-01.png)
![](/image/with-bean-validation-fail-log-02.png)

---

- https://jcp.org/en/jsr/detail?id=303
- https://en.wikipedia.org/wiki/Metadata
- https://beanvalidation.org
