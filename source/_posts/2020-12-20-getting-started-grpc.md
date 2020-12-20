---
layout: post
title: "[gRPC 시작에서 운영까지] 2장_gRPC 시작"
date: 2020-12-20
categories: gRPC
---

![](/image/grpc-server-client.png)

위와 같은 온라인 판매 시스템을 만들어보자.

## 서비스 정의 작성

gRPC 애플리케이션을 개발할 때, 가장 먼저 할 일은 클라이언트가 원격으로 호출할 수 있는 메서드와 메서드의 파라미터, 사용자 메시지 포맷 등을 포함하는 서비스 인터페이스를 정의하는 것이다.
서비스 정의는 gRPC 에서 사용되는 인터페이스 정의 언어 (==IDL) 인 프로토콜 버퍼 정의로 작성된다.

### 메세지 정의

```protobuf
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

위에서, 각 메세지 필드에 정의된 번호는 메세지에서 필드를 고유하게 식별하는데 사용된다. 

### 서비스 정의

서비스는 클라이언트에게 제공되는 원격 메서드의 모임이다.
위에서 작성한 Product message, ProductID message 와 같이 작성하자.

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

## 구현

![](/image/grpc-server-client-implementation.png)

이제, 서비스 정의에서 설정된 원격 메서드들로 gRPC 서비스를 구현한다.
원격 메서드들은 서버에 의해 제공되며, gRPC 클라이언트는 서버에 연결해 해당 원격 메서드를 호출한다.
먼저, ProductInfo 서비스 정의를 컴파일하고 선택한 언어로 소스 코드를 생성해야한다.
컴파일은, 

1. 프로토콜 버퍼 컴파일러를 사용해 프로토 파일을 수동으로 컴파일 하거나,
2. Maven 이나 Gradle 같은 빌드 자동화 도구를 사용할수도 있다.

## 서비스 개발

### gRPC 서비스 구현

```java
package ecommerce;

import io.grpc.Status;
import io.grpc.StatusException;

import java.util.HashMap;
import java.util.Map;
import java.util.UUID;

public class ProductInfoImpl extends ProductInfoGrpc.ProductInfoImplBase { // 1

    private Map productMap = new HashMap<String, ProductInfoOuterClass.Product>();

    @Override
    public void addProduct(ProductInfoOuterClass.Product request, // 2
                           io.grpc.stub.StreamObserver<ProductInfoOuterClass.ProductID> responseObserver) { // 3
        UUID uuid = UUID.randomUUID();
        String randomUUIDString = uuid.toString();
        request = request.toBuilder().setId(randomUUIDString).build();
        productMap.put(randomUUIDString, request);
        ProductInfoOuterClass.ProductID id
                = ProductInfoOuterClass.ProductID.newBuilder().setValue(randomUUIDString).build();
        responseObserver.onNext(id);
        responseObserver.onCompleted();
    }

    @Override
    public void getProduct(ProductInfoOuterClass.ProductID request,
                           io.grpc.stub.StreamObserver<ProductInfoOuterClass.Product> responseObserver) { 
        String id = request.getValue();
        if (productMap.containsKey(id)) {
            responseObserver.onNext((ProductInfoOuterClass.Product) productMap.get(id));
            responseObserver.onCompleted();
        } else {
            responseObserver.onError(new StatusException(Status.NOT_FOUND));
        }
    }
}
```

1. 플러그인에 의해 생성된 추상 클래스를 확장한다.
2. Product 클래스는 서비스 정의에 의해 생성된 ProductInfoOuterClass 에 선언되어 있다.
3. responseObserver 객체는 클라이언트에게 응답을 보내고 스트림을 닫기 위해 사용된다.

### 서버 생성

서비스를 위부에 제공하려면, gRPC 서버 인스턴스를 생성할 때 ProductInfo 서비스를 서버에 등록하면 된다.

```java
package ecommerce;

import io.grpc.Server;
import io.grpc.ServerBuilder;

import java.io.IOException;
import java.util.logging.Logger;

public class ProductInfoServer {

    private static final Logger logger = Logger.getLogger(ProductInfoServer.class.getName());
    private Server server;

    private void start() throws IOException {
        /* The port on which the server should run */
        int port = 50051;
        server = ServerBuilder.forPort(port)
                .addService(new ProductInfoImpl())
                .build()
                .start();
        logger.info("Server started, listening on " + port);
        Runtime.getRuntime().addShutdownHook(new Thread(() -> {
            // Use stderr here since the logger may have been reset by its JVM shutdown hook.
            logger.info("*** shutting down gRPC server since JVM is shutting down");
            ProductInfoServer.this.stop();
            logger.info("*** server shut down");
        }));
    }

    private void stop() {
        if (server != null) {
            server.shutdown();
        }
    }

    /**
     * Await termination on the main thread since the grpc library uses daemon threads.
     */
    private void blockUntilShutdown() throws InterruptedException {
        if (server != null) {
            server.awaitTermination();
        }
    }

    /**
     * Main launches the server from the command line.
     */
    public static void main(String[] args) throws IOException, InterruptedException {
        final ProductInfoServer server = new ProductInfoServer();
        server.start();
        server.blockUntilShutdown();
    }

}
```

### Client 개발

proto 서비스 정의 파일은 이미 여러 번 사용한 proto 파일을 그대로 사용한다.
그리고, Gradle Build Tool 을 사용해 프로젝트에 대한 클라이언트 스텁 코드를 생성한다.
그리고, ecommerce 패키지에 아래 클래스를 작성한다.

```java
package ecommerce;

import io.grpc.ManagedChannel;
import io.grpc.ManagedChannelBuilder;

import java.util.logging.Logger;

/**
 * gRPC client sample for productInfo service.
 */
public class ProductInfoClient {

    private static final Logger logger = Logger.getLogger(ProductInfoClient.class.getName());

    public static void main(String[] args) throws InterruptedException {
        ManagedChannel channel = ManagedChannelBuilder.forAddress("localhost", 50051) // 1
                .usePlaintext()
                .build();

        ProductInfoGrpc.ProductInfoBlockingStub stub =
                ProductInfoGrpc.newBlockingStub(channel); // 2

        ProductInfoOuterClass.ProductID productID = stub.addProduct(
                ProductInfoOuterClass.Product.newBuilder()
                        .setName("Samsung S10")
                        .setDescription("Samsung Galaxy S10 is the latest smart phone, " +
                                "launched in February 2019")
                        .setPrice(700.0f)
                        .build());
        logger.info("Product ID: " + productID.getValue() + " added successfully.");

        ProductInfoOuterClass.Product product = stub.getProduct(productID);
        logger.info("Product: " + product.toString());
        channel.shutdown();
    }
}
```

1. 현재 local 에서 실행되고 포트 50051 로 수신을 대기하는 서버에 연결한다.
2. 채널을 사용해 클라이언트 스텁을 만드는데, 아래 두 가지 유형을 사용할 수 있다.

> BlockingStub : 서버의 응답을 받을 때 까지 대기
> NonBlockingStub : 서버 응답을 기다리지 않고 Observer 를 등록해 응답을 받음


## 빌드와 실행

이제 gRPC 서버와 클라이언트 애플리케이션을 빌드하고 실행하자.

### 서버 빌드

```bash
gradle build
```

### 클라이언트 빌드

```bash
gradle build
```

### 서버와 클라이언트 실행

```bash
java -jar build/libs/product-info-service.jar
java -jar build/libs/product-info-client.jar
```

---

gRPC 시작에서 운영까지 <카순 인드라시리, 다네쉬 쿠루푸>