---
layout: post
title:  "[전문가를 위한 스프링 5] 4장_스프링 구성 상세와 스프링 부트"
date:   2020-07-22
categories: Spring
---
## 1. Bean Life Cycle

IoC 컨테이너가 제공하는 주요 기능 중 하나는, 빈의 생성이나 소멸 같은 `라이프 사이클의 특정 시점에 통지를 받을 수 있도록 빈을 생성`하는 것이다.
일반적으로 두 가지 라이프사이클 이벤트가 있다.

1. post-initialization event : 스프링이 개발자가 구성한 대로 빈에 모든 프로퍼티 값을 설정하고 의존성 점검을 마치자 마자 발생 
2. pre-destruction event : 스프링이 빈 인스턴스를 소멸시키기 바로 전에 발생

> 요청을 받을 때 마다 스프링 컨테이너가 매번 빈을 생성하는 Prototype 빈에는, 스프링이 소멸 전 이벤트를 통지하지 않는다.

빈이 이런 라이프사이클 이벤트를 받을 수 있는 메너니즘은 세 가지가 있다.

1. inteface 기반
2. method 기반
3. annotation 기반

## 2. 빈 생성 시점에 통지 받기

1. 빈 생성 시 메서드 실행
   초기화 메서드로 제대로 빈이 구성되었는지 확인할 수 있다.
   다음은, xml 파일에 default-init-method 애트리뷰트에 초기화 메서드를 지정하였다.

   ```java
   public class Singer {
       private static final String DEFAULT_NAME = "Eric Clapton";
   
       private String name;
       private int age = Integer.MIN_VALUE;
   
       public void setName(String name) {
           this.name = name;
       }
   
       public void setAge(int age) {
           this.age = age;
       }
   
       private void init() {
           System.out.println("빈 초기화");
   
           if (name == null) {
               System.out.println("기본 이름 사용");
               name = DEFAULT_NAME;
           }
   
           if (age == Integer.MIN_VALUE) {
               throw new IllegalArgumentException(
                       Singer.class +" 빈 타입에는 반드시 age 프로퍼티를 설정해야 합니다.");
           }
       }
   }
   ```

   ```java
   <?xml version="1.0" encoding="UTF-8"?>
   
   <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd"
       default-lazy-init="true" default-init-method="init">
   
       <bean id="singerOne"
           class="com.apress.prospring5.ch4.Singer"
             p:name="John Mayer" p:age="39"/>
   
       <bean id="singerTwo"
           class="com.apress.prospring5.ch4.Singer"
            p:age="72"/>
   
       <bean id="singerThree"
           class="com.apress.prospring5.ch4.Singer"
            p:name="John Butler"/>
   </beans>
   ```

2. InitializingBean 인터페이스 구현하기
   위 코드의 init() 메서드와 같은 역할을 하는 afterPropertiesSet() 메서드를 정의한다.

   ```java
   public class SingerWithInterface implements InitializingBean {
       private static final String DEFAULT_NAME = "Eric Clapton";
   
       private String name;
       private int age = Integer.MIN_VALUE;
   
       public void setName(String name) {
           this.name = name;
       }
   
       public void setAge(int age) {
           this.age = age;
       }
   
       public void afterPropertiesSet() throws Exception {
           System.out.println("빈 초기화");
   
           if (name == null) {
               System.out.println("기본 가수 이름 설정");
               name = DEFAULT_NAME;
           }
   
           if (age == Integer.MIN_VALUE) {
               throw new IllegalArgumentException(
                       SingerWithInterface.class 
                       +" 빈 타입에는 반드시 age 프로퍼티를 설정해야 합니다.");
           }
       }
   }
   ```

3. JSR-250 @PostConstruct 어노테이션 사용하기
   빈 클래스 내에서 스프링이 호출할 메서드를 지정한다.

   ```java
   public class SingerWithJSR250 {
       private static final String DEFAULT_NAME = "Eric Clapton";
   
       private String name;
       private int age = Integer.MIN_VALUE;
   
       public void setName(String name) {
           this.name = name;
       }
   
       public void setAge(int age) {
           this.age = age;
       }
   
       @PostConstruct
       private void init() throws Exception {
           System.out.println("빈 초기화");
   
          if (name == null) {
               System.out.println("기본 가수 이름 설정");
               name = DEFAULT_NAME;
           }
   
           if (age == Integer.MIN_VALUE) {
               throw new IllegalArgumentException(
                   SingerWithJSR250.class 
                   +" 빈 타입에는 반드시 age 프로퍼티를 설정해야 합니다.");
           }
       }
   }
   ```

4. @Bean 으로 초기화 메서드 선언하기
   자바 구성 클래스에서 빈을 선언할 때 사용한다. @Lazy 는 xml 의 default-lazy-init="true" 와 동일하다. 

   ```java
   public class SingerConfigDemo {
   
       @Configuration
       static class SingerConfig {
           @Lazy
           @Bean(initMethod = "init")
           Singer singerOne() {
               Singer singerOne = new Singer();
               singerOne.setName("John Mayer");
               singerOne.setAge(39);
               return singerOne;
           }
   
           @Lazy
           @Bean(initMethod = "init")
           Singer singerTwo() {
               Singer singerTwo = new Singer();
               singerTwo.setAge(72);
               return singerTwo;
           }
   
           @Lazy
           @Bean(initMethod = "init")
           Singer singerThree() {
               Singer singerThree = new Singer();
               singerThree.setName("John Butler");
               return singerThree;
           }
       }
   }
   ```

빈 생성은 다음과 같은 단계를 거친다.

1. 빈 생성 위해 생성자 호출
2. 의존성 주입 (수정자 호출)
3. 사전 초기화 담당하는 BeanPostProcessor 기반 빈들에게 호출해야하는 메서드 있는지 확인 요청한다. 여기서 BeanPostProcessor 는 빈이 생성된 이후에 빈 조작을 수행하는 스프링에 특화된 인터페이스다.
4. @PostContruct 어노테이션은 CommonAnnotationBeanPostProcessor 빈에 등록되므로, CommonAnnotationBeanPostProcessor 빈이 @PostContruct 어노테이션이 적용된 메서드 호출
5. InitializingBean 의 afterPropertiesSet 메서드 실행
6. init-method 애트리뷰트로 지정한 빈의 실제 초기화 메서드를 마지막에 실행

## 3. 빈 소멸 시점에 통지 받기

1. 빈이 소멸될 때 메서드를 실행
   다음은, xml 파일에 destroty-method 애트리뷰트에 소멸 콜백 메서드를 지정하였다.

   ```java
   public class DestructiveBean {
       private File file;
       private String filePath;
       
       public void afterPropertiesSet() throws Exception {
           System.out.println("빈을 초기화합니다.");
   
           if (filePath == null) {
               throw new IllegalArgumentException(
                   DestructiveBean.class + "에 filePath 프로퍼티를 지정해야 합니다.");
           }
   
           this.file = new File(filePath);
           this.file.createNewFile();
   
           System.out.println("파일 존재여부: " + file.exists());
       }
   
       public void destroy() {
           System.out.println("빈을 소멸합니다.");
   
           if(!file.delete()) {
               System.err.println("에러: 파일 삭제에 실패했습니다.");
           }
   
           System.out.println("파일 존재여부: " + file.exists());
       }
   }
   ```

   ```java
   <?xml version="1.0" encoding="UTF-8"?>
   
   <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">
   
       <bean id="destructiveBean" 
           class="com.apress.prospring5.ch4.DestructiveBean"
           destroy-method="destroy"
           init-method="afterPropertiesSet"
           p:filePath=
           "#{systemProperties['java.io.tmpdir']}#{systemProperties['file.separator']}test.txt"/>
   </beans>
   ```

2. DisposableBean 인터페이스 구현하기
   destroy() 메서드를 정의한다. 빈 소멸 직전에 호출된다.

   ```java
   public class DestructiveBeanWithInterface implements InitializingBean, DisposableBean {
       private File file;
       private String filePath;
       
       @Override
       public void afterPropertiesSet() throws Exception {
           System.out.println("빈을 초기화합니다.");
   
           if (filePath == null) {
               throw new IllegalArgumentException(
                   DestructiveBeanWithInterface.class
                   + "에 filePath 프로퍼티를 지정해야 합니다.");
           }
   
           this.file = new File(filePath);
           this.file.createNewFile();
   
           System.out.println("파일 존재 여부: " + file.exists());
       }
   
       @Override
       public void destroy() {
           System.out.println("빈을 소멸합니다.");
   
           if(!file.delete()) {
               System.err.println("에러: 파일 삭제에 실패했습니다.");
           }
   
           System.out.println("파일 존재 여부: " + file.exists());
       }
   }
   ```

3. JSR-250 @PreDestroy 어노테이션 사용

   ```java
   public class DestructiveBeanWithJSR250 {
       private File file;
       private String filePath;
       
       @PostConstruct
       public void afterPropertiesSet() throws Exception {
           System.out.println("빈을 초기화합니다.");
   
           if (filePath == null) {
               throw new IllegalArgumentException(
                   DestructiveBeanWithJSR250.class + "에 filePath 프로퍼티를 지정해야 합니다.");
           }
   
           this.file = new File(filePath);
           this.file.createNewFile();
   
           System.out.println("파일 존재 여부: " + file.exists());
       }
   
       @PreDestroy
       public void destroy() {
           System.out.println("빈을 소멸합니다.");
   
           if(!file.delete()) {
               System.err.println("에러: 파일 삭제에 실패했습니다.");
           }
   
           System.out.println("파일 존재 여부: " + file.exists());
       }
   }
   ```

4. @Bean 을 사용해 소멸 메서드 정의하기
   자바 구성 클래스에서 빈을 선언할 때 사용한다.

   ```java
   public class DestructiveBeanConfigDemo {
     
       @Configuration
       static class DestructiveBeanConfig {
           @Lazy
           @Bean(initMethod = "afterPropertiesSet", destroyMethod = "destroy")
           DestructiveBeanWithJSR250 destructiveBean() {
               DestructiveBeanWithJSR250 destructiveBean = new DestructiveBeanWithJSR250();
               destructiveBean.setFilePath(System.getProperty("java.io.tmpdir") +
                       System.getProperty("file.separator") + "test.txt");
               return destructiveBean;
           }
   
       }
   }
   ```

## 4. 빈이 스프링을 알게하기 (Spring Aware)

1. BeanNameAware 인터페이스 사용하기
   자신에게 부여된 이름을 알고자 하는 빈이 구현해야할 BeanNameAware 인터페이스는 setBeanName() 메서드를 가지고 있다.
   스프링은 빈 구성을 마친 뒤 라이프사이클 콜백 (초기화 or 소멸) 을 호출하기 전에 setBeanName() 메서드를 호출한다.

   ```java
   public class NamedSinger implements BeanNameAware {
       private String name;
   
       /** @Implements {@link BeanNameAware#setBeanName(String)} */
       public void setBeanName(String beanName) {
           this.name = beanName;
       }
   
       public void sing() {
           System.out.println("Singer [" + name + "] - sing()");
       }
   }
   ```

2. ApplicatoinContextAware 인터페이스 사용하기
   빈은 자신을 관리하는 ApplicatoinContext 인스턴스의 참조를 얻을 수 있다.

   ```java
   public class ShutdownHookBean implements ApplicationContextAware {
       private ApplicationContext ctx;
   
       /** @Implements {@link ApplicationContextAware#setApplicationContext(ApplicationContext)}  }*/
       public void setApplicationContext(ApplicationContext ctx)
           throws BeansException {
   
           if (ctx instanceof GenericApplicationContext) {
               ((GenericApplicationContext) ctx).registerShutdownHook();
           }
       }
   }
   ```

---

전문가를 위한 스프링 5 <율리아나 코스미스>
