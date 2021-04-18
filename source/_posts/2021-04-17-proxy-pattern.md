---
layout: post
title:  "Proxy Pattern"
date:   2021-04-17
categories: Java
---

Proxy Pattern 에 대해 정리한다.

## Proxy Server

프록시 패턴에 대해 정리하기 전에, 프록시 서버가 무엇인지 보자.

프록시의 사전적 의미는 "대리, 대리권, 대리 투표, 대리인" 이다.
프록시 서버는, 위키에 따르면 다음과 같다.

> 프록시 서버는 클라이언트가 자신을 통해서 다른 네트워크 서비스에 간접적으로 접속할 수 있게 해 주는 컴퓨터 시스템이나 응용 프로그램을 가리킨다. 
> 서버와 클라이언트 사이에 중계기로서 대리로 통신을 수행하는 것을 가리켜 '프록시', 그 중계 기능을 하는 것을 '프록시 서버' 라고 부른다.

![](/image/proxy-server.png)

정리하면 프로시 서버는,

1. 클라이언트의 요청을 서버에게 대신 전달하고
2. 서버의 응답을 클라이언트에게 대신 전달한다.


## Proxy Pattern

그렇다면, 프록시 패턴은 무엇일까 ? 위키에 따르면 다음과 같다.

> 프록시 패턴(proxy pattern)은 컴퓨터 프로그래밍에서 소프트웨어 디자인 패턴의 하나이다. 
> 일반적으로 프록시는 다른 무언가와 이어지는 인터페이스의 역할을 하는 클래스이다. 
> 프록시는 어떠한 것 (이를테면 네트워크 연결, 메모리 안의 커다란 객체, 파일, 또 복제할 수 없거나 수요가 많은 리소스) 과도 인터페이스의 역할을 수행할 수 있다.


프록시 패턴은 다음과 같은 클래스 다이어그램으로 나타난다.

![](/image/proxy-pattern.png)

Proxy 클래스가

1. Subject 인터페이스를 구현하면서,
2. Subject 를 구현하고 있는 RealSubject 에 의존하고 있다.

## Example

프록시 패턴의 간다한 예제를 살펴보자.
예제의 클래스 다이어그램은 다음과 같다.

![](/image/proxy-pattern-example.png)

ProxyImage 클래스가 

1. Image 인터페이스를 구현하면서,
2. Image 를 구현하고 있는 RealImage 에 의존하고 있다.

이제 코드로 보자. 아래는, 클라이언트가 의존하는 interface 이다.

```java
interface Image {
    public void displayImage();
}
```

Image 인터페이스를 구현하는 구현 클래스이다.

```java
class RealImage implements Image {
    private String filename;
    public RealImage(String filename) {
        this.filename = filename;
        loadImageFromDisk();
    }

    private void loadImageFromDisk() {
        System.out.println("Loading   " + filename);
    }

    @Override
    public void displayImage() {
        System.out.println("Displaying " + filename);
    }
}
```

Image 인터페이스를 구현하면서, RealImage 클래에 의존하는 클래스이다.

```java
class ProxyImage implements Image {
    private String filename;
    private Image image; // <------- Here

    public ProxyImage(String filename) {
        this.filename = filename;
    }

    @Override
    public void displayImage() {
        if (image == null)
           image = new RealImage(filename);

        image.displayImage();
    }
}
```

위 코드들을 수행하는 client code 는 아래와 같다.

```java
class ProxyExample {
    public static void main(String[] args) {
        Image image1 = new ProxyImage("HiRes_10MB_Photo1");
        Image image2 = new ProxyImage("HiRes_10MB_Photo2");

        image1.displayImage();
        image2.displayImage();
    }
}
```

실행 결과는,

```java
Loading    HiRes_10MB_Photo1
Displaying HiRes_10MB_Photo1
Loading    HiRes_10MB_Photo2
Displaying HiRes_10MB_Photo2
```

## 정리

실행 흐름은 다음과 같다.

1. 클라이언트가 displayImage() 를 호출하면
2. ProxyImage 는 RealImage 생성한 뒤에
3. RealImage 의 displayImage() 을 호출한다.

위 Example 의 핵심적인 부분은 여기다.

```java
class ProxyImage implements Image {
    private String filename;
    private Image image;
    
    // ...
}
```

ProxyImage 클래스는 Image 를 구현하면서, 동시에 Image 클래스를 필드로 가지고 있다.
클라이언트는 ProxyImage 클래스의 displayImage 메서드를 호출했는데, 실제로는 RealImage 의 displayImage 메서드가 호출되고 있다.
ProxyImage 클래스가 RealImage 대신에 대리, 즉 프록시 역할을 하고 있는 것이다.

## 언제 사용 ?

프록시 패턴을 언제 사용할 수 있을끼 ?
Real Target 을 호출하기 전,후로 추가적인 기능을 실행하기 위해 사용될 수 있다.

예를 들어, RealImage 를 생성전이나 RealImage 의 displayImage() 를 호출 후에 log 를 남기고 싶다면 
ProxyImage 클래스에 logging 을 추가해서 RealImage 클래스 변경 없이 구현가능하다.

---
https://en.wikipedia.org/wiki/Proxy_pattern
https://github.com/junhee-ko/proxy-pattern
https://github.com/junhee-ko/demo-jpa-proxy
