---
layout: post
title:  "[전문가를 위한 스프링 5] 3장_Spring IoC 와 DI"
date:   2020-06-27
categories: Spring
---
## 1. IoC 종류

IoC (Inversion of Control ) 은 두 가지로 나뉜다 : DI, DL

DI 는 `IoC 컨테이너가 컴포넌트에 의존성을 주입`시켜 준다. 반면, DL 은 컴포넌트 `스스로 의존성 참조`를 가져온다.

- DI (Dependency Injection)

  - 생성자 주입 : IoC 컨테이너는 해당 컴포넌트를 초기화할 때, 컴포넌트에 필요한 의존성을 전달

    ```java
    public ConstructorInjection(Dependency dp){
       this.dp = dp;
    }
    ```

  - 세터 주입 : setter method 를 호출해서, 의존성을 나중에 제공 가능

    ```java
    public void setDependecy(Dependency dp){
       this.dp = dp;
    }
    ```

  - 필드 주입 : 스프링 컨테이너가 reflection 을 이용해 필요한 의존성을 주입

    ```java
    @Service
    public class Singer { 
       @Autowired
       private Inspiration inspirationBean;
       ...
    ```
  
- DL (Dependency Lookup)

  - Depenency Pull : 중앙 registry 에서 의존성을 직접 가져오는 방식 ( register -> container )

    ```java
    context.getbean();
    ```

  - Contextualized Dependency Lookup : 특정 중앙 registry 에서 의존성을 가져오는 것이 아니라, 자원을 관리하는 컨테이너에서 의존성을 가져오는 방식

    ```java
    publicv void lookup(Container ct){
    	this.dp = (Dependency) ct.getDependency("myDependency");
    }
    ```



그렇다면, 의존성 주입 vs 의존성 룩업 ? `의존성 주입을 사용해라.` 주입을 이용하면, 

- 사용자 클래스는 IoC 컨테이너와 완전리 분리된다. 
- 직접 테스트용 의존성을 주입하기 쉬우므로 테스트 하기 쉽니다.

그렇다면, 생성자 주입 vs 세터 주입 vs 필드 주입? 상황에 따라 선택해라.

- 생성자 주입 : `컴포넌트에 의존성 주입을 보장`해야할 때
- 세터 주입 : `새로운 객체를 생성하지 않고 의존성을 교체`할 때
- 필드 주입 : 다음 이유로, 권장하지 않는다.
  - 클래스가 비대해지는 상황을 생성자 주입이나 수정자 주입에서는 쉽게 알아 챌 수 있지만, 필드 주입은 아니다.
  - 클래스는 public interface 의 메서드나 생성자로 필요한 의존성 타입을 명확히 전달해야하는데, 필드 주입을 이용하면 어떤 타입의 의존성이 필요한지 명확하지 않다.
  - final 필드에 사용할 수 없다.
  - 의존성을 수동으로 주입해야하므로, 테스트 코드 작성이 어렵다.

## 2. BeanFactory, ApplicationContext

1. Bean : 컨테이너가 관리하는 모든 컴포넌트

2. BeanFactory interface : 컴포넌트의 라이프사이클과 의존성을 관리

3. ApplicationContext interface (extends BeanFactory) : DI 외에도, 트랜잭션, AOP, 애플리케이션 이벤트 처리 기능 제공


## 3. Application Context 구성

1. Component 선언

   AppliceionContext 를 부트스트랩할 때, 컴포넌트를 찾아서 빈 인스턴스를 생성한다.

   ```java
   @Component("provider")
   public class HelloWorldMessageProvider implements MessageProvider {
       ...
   }
   ```

   ```java
   @Service("renderer")
   public class StandardOutMessageRenderer implements MessageRenderer {
       private MessageProvider messageProvider;
     
       @Override
       @Autowired
       public void setMessageProvider(MessageProvider provider) {
           this.messageProvider = provider;
       }
     	
       ...
   }
   ```

2. Java Configuration

   Configuration 클래스에는, IoC 컨테이너가 빈 인스턴스를 만들 때 호출하는 @Bean 어노테이션이 적용된 메서드가 있다.

   ```java
   @Configuration
   public class HelloWorldConfiguration {
     
     @Bean
     public MessageProvider provider() {
         return new HelloWorldMessageProvider();
     }
   }
   ...
   ```

## 4. 주입 인자

스프링은 다른 컴포넌트나 단순 값 이외에 자바 컬렉션, 외부에 정의된 프로퍼티를 주입할 수 있도록 주입 인자에 많은 옵션을 지원한다.

1. 단순 값 주입

   ```java
   @Service
   public class InjectSimple {
       @Value("John Mayer")
       private String name;
       @Value("40")
       private int age;
       @Value("1.92")
       private float height;
       @Value("false")
       private boolean programmer;
       @Value("1241401112")
       private Long ageInSeconds;
       ...
   ```

2. SpEL (Spring Expression Language)

   ```java
   @Service
   public class InjectSimpleSpel {
       @Value("#{injectSimpleConfig.name}")
       private String name;
   
       @Value("#{injectSimpleConfig.age + 1}")
       private int age;
   
       @Value("#{injectSimpleConfig.height}")
       private float height;
   
       @Value("#{injectSimpleConfig.programmer}")
       private boolean programmer;
   
       @Value("#{injectSimpleConfig.ageInSeconds}")
       private Long ageInSeconds;
       ...
   ```

   ```java
   @Component("injectSimpleConfig")
   public class InjectSimpleConfig {
       private String name = "John Mayer";
       private int age = 40;
       private float height = 1.92f;
       private boolean programmer = false;
       private Long ageInSeconds = 1_241_401_112L;
       ...
   ```

3. 컬렉션 주입

   ```java
   @Service
   public class CollectionInjection {
   
       /**
        * @Resource(name="map") is equivalent with @Autowired @Qualifier("map")
        */
       @Autowired
       @Qualifier("map")
       private Map<String, Object> map;
   
       @Resource(name="props")
       private Properties props;
   
       @Resource(name="set")
       private Set set;
       
       @Resource(name="list")
       private List list;
   ```

## 5. 빈 명명 규칙

모든 빈은 ApplicationContext 내에서 고유한 하나 이상의 이름을 가져야한다.

id 나 이름이 없는 같은 타입의 빈이 여러개 선언되면, 스프링은 ApplicationContext 를 초기화하는 과정에서 빈을 주입할 때 예외 (NoSuchBeanDefinitionException) 를 던진다.

다음 Singer 클래스의 경우, 클래스명 자체로 빈을 명명한다 : singer

```java
@Component
public class Singer {
    ...
}
```

다음 Singer 클래스의 경우, @Component 어노테이션의 인수가 빈의 이름이다 : johnMayer

```java
@Component("johnMayer")
public class Singer {
    ...
}
```

별칭을 선언하려면 ? @Bean 의 name attribute 를 사용한다. 첫 번째 값이 id, 다른 값이 별칭. 즉, johnMayer 가 id

```java
@Configuration
static class AliasBeanConfig {
		
    @Bean(name = {"johnMayer", "john", "jonathan", "johnny"})
    public Singer singer() {
        return new Singer();
    }
}
```

## 6. 빈 생성 방식

기본적으로 스프링의 모든 빈은 `싱글톤`이다. 즉, 스프링은 빈의 단일 인스턴스를 유지하고 관리한다. ApplicationContext.getBean() 에 대한 모든 호출은 동일한 인스턴스를 반환한다.

`빈의 인스턴스를 가져올 때마다 새로운 빈 인스턴스`를 가져오려면 ? 다음과 같이, `prototype` 으로 변경해라.

```java
@Component("nonSingleton")
@Scope("prototype")
public class Singer {
      private String name = "unknown";

      public Singer(@Value("John Mayer") String name) {
            this.name = name;
      }

      @Override public String toString() {
            return  name;
      }
}
```

## 7. Autowiring

스프링이 Autowiring 을 수행하는 다섯 가지 방식이 있다.

1. byName : ApplicationContext 에서 각 프로퍼티와 `이름이 같은` 빈을 찾아서 연결 시도
2. byType : ApplicationContext 에서 `동일한 타입`의 빈을 대상 빈의 각 프로퍼티에 연결 시도
3. Constructor : 주입이 수정자가 아닌 생성자를 이용해서 이루어진다는 점을 제외하면 byType 과 동일
4. default : 빈에 기본 생성자가 있으면 byType 방식, 없으면 Constructor 방식
5. no : 기본값

어노테이션을 이용해서 구성할 때, 기본 Autowiring 방식은 byType 이다. 

이름을 기반으로 Autowiring 을 하기 위해서는 @Autowired 와 주입되어야 하는 빈의 이름을 인자로 전달하는 @Qualifier 을 적용해라.

다음 코드로 예를 들어보자. 


```java
@Component
@Lazy // 처음 접근이 일어날 때 인스턴스가 생상될 빈을 정의
public class TrickyTarget {

	Foo fooOne;
	Foo fooTwo;
	Bar bar;

	public TrickyTarget() {
		System.out.println("Target.constructor()");
	}

	public TrickyTarget(Foo fooOne) {
		System.out.println("Target(Foo) 호출");
	}

	public TrickyTarget(Foo fooOne, Bar bar) {
		System.out.println("Target(Foo, Bar) 호출");
	}

	@Autowired
	public void setFooOne(Foo fooOne) {
		this.fooOne = fooOne;
		System.out.println("프로퍼티 fooOne 설정");
	}

	@Autowired
	public void setFooTwo(Foo foo) {
		this.fooTwo = foo;
		System.out.println("프로퍼티 fooTwo 설정");
	}

	@Autowired
	public void setBar(Bar bar) {
		this.bar = bar;
		System.out.println("프로퍼티 bar 설정");
	}

	public static void main(String... args) {
		GenericXmlApplicationContext ctx = new GenericXmlApplicationContext();
		ctx.load("classpath:spring/app-context-05.xml");
		ctx.refresh();
		TrickyTarget t = ctx.getBean(TrickyTarget.class);
		ctx.close();
	}
}
```

```java
@Component
public class Foo {

}
```

```java
@Component
public class Bar {

}
```

그리고, TrickyTarget 클래스를 실행하면 다음과 같이 출력된다.

```java
Target.constructor()
프로퍼티 fooOne 설정
프로퍼티 fooTwo 설정
프로퍼티 bar 설정
```

그런데 만약에, 빈 타입들이 서로 연관되면 상황이 복잡해진다. Foo 를 인터페이스로 변경하고 이 인터페이스를 구현하는 빈 타입 두개를 선언해보자.

```java
public interface Foo {

}
```

```java
@Component
public class FooImplOne implements Foo {

}
```

```java
@Component
public class FooImplTwo implements Foo{

}
```

그리고 다시 TrickyTarget 클래스를 실행하면, UnsatisfiedDependencyExpcetion 이 발생한다. 

이 메세지는 스프링이 setFoo 메서드를 사용해서 주입해야하는 빈이 어떤 빈인지 알지 못한다는 것이다.

이를 해결 하는 두 가지 방법이 있다 : @Primary, @Qualifier

1. @Primary

   타입을 기반으로 Autowiring 할 때, 스프링에게 자신의 우선순위를 높게 지정한다. 

   @Primary 는 정확히 두 개의 관련 빈 타입이 있을 경우에 유용하다.

   ```java
   @Component
   @Primary
   public class FooImplOne implements Foo {
   
   }
   ```

   다시 TrickyTarget 클래스를 실행하면, 결과는 다음과 같다.

   ```
   Target.constructor()
   프로퍼티 fooOne 설정
   프로퍼티 fooTwo 설정
   프로퍼티 bar 설정
   ```

2. @Qualifier

   모호한 setter 였던 setFooOne() 과 setFooTwo() 에 @Autowired 와 함께 적용한다.
   
   ```java
   @Component
   @Lazy
   public class TrickyTarget {
   
   	Foo fooOne;
   	Foo fooTwo;
   	Bar bar;
   
   	...
   
   	@Autowired
   	@Qualifier("fooImplOne")
   	public void setFooOne(Foo fooOne) {
   		this.fooOne = fooOne;
   		System.out.println("프로퍼티 fooOne 설정");
   	}
   
   	@Autowired
   	@Qualifier("fooImplTwo")
   	public void setFooTwo(Foo foo) {
   		this.fooTwo = foo;
   		System.out.println("프로퍼티 fooTwo 설정");
   	}
   
   	...
   ```
   
   다시 TrickyTarget 클래스를 실행하면, 결과는 다음과 같다.
   
   ```
   Target.constructor()
   프로퍼티 fooOne 설정
   프로퍼티 fooTwo 설정
   프로퍼티 bar 설정
   ```

---

전문가를 위한 스프링 5 <율리아나 코스미스>
