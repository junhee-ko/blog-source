---
layout: post
title: "[gRPC 시작에서 운영까지] 1장_gRPC 소개"
date: 2020-12-19
categories: gRPC
---

## gRPC

gRPC 는 로컬 함수를 호출하는 것 만큼 쉽게 분산된 이기종 애플리케이션을 연결, 호출, 운영, 디버깅할 수 있는 프로세스 간 통신 기술이다.
gRPC 애플리케이션을 개발할 때 가장 먼저 해야할 일은, 서비스 인터페이스를 정의하는 것이다.
이 때, 인터페이스 정의 언어 (==IDL) 을 사용한다.
서비스 정의를 사용해 Server Skeleton 이라는 server side code 를 생성할 수 있고, Stub 이라는 client side code 도 생성할 수 있다.

![](/image/grpc-server-client-implementation.png)

위 예시를 통해 살펴보자.
서비스 정의는 ProductInfo.proto 파일에 정의하고, 서버 측과 클라이언트 측 코드를 생성하는데 사용된다.
서비스 정의부터 자세히 보자.

## 서비스 정의

gRPC 는 프로토콜 버퍼를 IDL 로 사용해서 서비스 인터페이스를 정의한다.

```protobuf
package ecommerce;

service ProductInfo {
    rpc addProduct(Product) returns (ProductID);
    rpc getProduct(ProductID) returns (Product);
}

message Product {
    string id = 1;
    string name = 2;
    string description = 3;
    float price = 4;
}

message ProductID {
    string value = 1;
}
```

## gRPC Server

위와 같이 서비스 정의가 완료되면, 이를 사용해 프로토콜 버퍼 컴파일러인 protoc 를 사용해 서버 측이나 클라이언트 측 코드를 생성할 수 있다.
그리고, 서버는 아래 처리가 필요하다.

1. 상위 서비스 클래스를 overriding 함으로써 생성된 서버 스켈레톤의 서비스 로직을 구현
2. gRPC 서버를 실행해 클라이언트 요청을 수신하고 응답

## gRPC Client

서버 측과 마찬가지로 서비스 정의를 사용해 클라이언트 스텁을 생성한다.
스텁은 서버와 동일한 메서드를 제공하는데, 클라이언트 코드에서 메서드의 호출을 네트워크 상 원격 함수 호출로 변환해준다.

---

gRPC 시작에서 운영까지 <카순 인드라시리, 다네쉬 쿠루푸>