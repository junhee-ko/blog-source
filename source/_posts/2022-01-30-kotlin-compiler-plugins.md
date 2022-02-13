---
layout: post 
title: Kotlin Compiler Plugins
date: 2022-02-13
categories: Kotlin
---

kotlin compiler plugins 중에 plugin-spring 과 plugin-jpa 에 대해 정리한다.

## 프로젝트 생성

Spring Initializer 를 이용해서 프로젝트를 생성해보자.
kotlin + gradle 기반에 의존성으로 Spring Data JPA 만 추가한다.
생성된 build.gradle.kt 파일을 보자.

![](/image/kotlin-project-init-gradle.png)

plugin 에 위와 같이 "plugin.spring", "plugin.jpa" 가 추가된 것을 볼 수 있다.
왜 이 두 개의 plugin 이 자동으로 추가된 걸까 ?

## plugin.spring

"org.jetbrains.kotlin.plugin.spring" 는 org.jetbrains.kotlin.plugin.allopen" 의 wrapper plugin 이다. "org.jetbrains.kotlin.plugin.allopen" 와 동일하게 동작한다.

"org.jetbrains.kotlin.plugin.spring" compiler plugin 은, 아래 annotation 이 붙은 클래스와 그 클래스의 맴버에 open 을 자동으로 추가한다.

- @Component
- @Async
- @Transactional
- @Cacheable
- @SpringBootTest

직접 확인해보자. 다음과 같이 프로젝트에 두 개의 클래스를 추가해보자.

```kotlin
class Book(
    private val name: String,
    private val number: Long
)

@Component
class BookComponent(
    private val name: String = "kotlin",
    private val number: Long = 84219
)
```

컴파일된 결과를 보면, Book 에는 open 키워드가 붙어있지 않지만, BookComponent 에는 붙어있다.

![](/image/kotlin-build-open.png)
![](/image/kotlin-build-not-open.png)

BookComponent 클래스에 @Component 가 붙어있기 때문에, 
all-open compiler plugin 이 BookComponent 클래스와 BookComponent 의 맴버인 name 과 number 에 open 을 자동으로 추가한다.

물론, @SpringBootApplication 이 붙은 클래스의 컴파일 결과에도 open 키워드가 붙어있다.

![](/image/kotlin-open-application-classs.png)


## plugin.jpa

"org.jetbrains.kotlin.plugin.jpa" 는 "org.jetbrains.kotlin.plugin.noarg" 의 wrapper plugin 이다.

"org.jetbrains.kotlin.plugin.jpa" compiler plugin 은, 아래 annotation 이 붙은 클래스에 zero-argument constructor 를 생성한다.
 
- @Entity
- @Embeddable
- @MappedSuperclass

직접 확인해보자. 다음과 같이 프로젝트에 두 개의 클래스를 추가해보자.

```kotlin
class User

@Entity
class UserEntity(
    @Id
    private val id: Long
)
```

compile 된 결과를 decompile 해보자. 
UserEntity 에는 zero-argument constructor 가 생성되었지만, User 에는 없다.

![](/image/kotlin-build-zero-argument-constructor.png)
![](/image/kotlin-build-not-zero-argument-constructor.png)

---
- https://kotlinlang.org/docs/all-open-plugin.html
- https://kotlinlang.org/docs/no-arg-plugin.html#jpa-support

