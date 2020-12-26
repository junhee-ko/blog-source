---
layout: post
title: "[gRPC 시작에서 운영까지] 3장_gRPC 통신 패턴"
date: 2020-12-26
categories: gRPC
---

gRPC 기반 애플리케이션에서 사용되는 네 가지 통신 패턴을 정리한다.

1. Unary RPC
2. Server Streaming RPC
3. Client Streaming RPC
4. Bidirectional Streaming RPC 

## Unary RPC

![](/image/grpc-pattern-unary.png)

위 방식이 Unary RPC 패턴을 따른 것이다.
클라이언트는 orderId 로 단일 요청을 보내고, 서비스는 주문 정보가 포함된 단일 응답을 돌려준다.
이 패턴을 구현해보자.

### Service

우선, 프로토콜 버퍼를 사용해 아래와 같이 서비스를 정의한다.

```protobuf
syntax = "proto3";

import "google/protobuf/wrappers.proto";

package ecommerce;

service OrderManagement {
    rpc getOrder(google.protobuf.StringValue) returns (Order);
}

message Order {
    string id = 1;
    repeated string items = 2; // 한 번 이상 반복되는 필드를 나타냄. 즉, 하나의 주문 메세지는 여러 아이템이 있을 수 있음.
    string description = 3;
    float price = 4;
    string destination = 5;
}
```

### Server

gRPC 서비스 정의 프로토 파일을 사용해서 서버 스켈레톤 코드를 생성한 뒤에, 아래와 같이 getOrder 메서드의 로직을 구현할 수 있다.

```java
public class OrderMgtServiceImpl extends OrderManagementGrpc.OrderManagementImplBase {
  
    @Override
    public void getOrder(StringValue request, StreamObserver<OrderManagementOuterClass.Order> responseObserver) {
        OrderManagementOuterClass.Order order = orderMap.get(request.getValue());
        if (order != null) {
            System.out.printf("Order Retrieved : ID - %s", order.getId());
            responseObserver.onNext(order);
            responseObserver.onCompleted();
        } else  {
            logger.info("Order : " + request.getValue() + " - Not found.");
            responseObserver.onCompleted();
        }
        // ToDo  Handle errors
        // responseObserver.onError();
    }
}
```

### Client

이제, getOrder 메서드를 원격으로 호출하는 클라이언트 로직을 구현하자. 
사용하는 언어에 대한 코드를 생성해 클라이언트 스텁을 만들고, 스텁을 사용해서 서비스를 호출한다.

```java
public class OrderMgtClient {

  public static void main(String[] args) {
    ManagedChannel channel = ManagedChannelBuilder.forAddress("localhost", 50051).usePlaintext().build();
    OrderManagementGrpc.OrderManagementBlockingStub stub = OrderManagementGrpc.newBlockingStub(channel);
    OrderManagementGrpc.OrderManagementStub asyncStub = OrderManagementGrpc.newStub(channel);

    // Get Order
    StringValue id = StringValue.newBuilder().setValue("101").build();
    OrderManagementOuterClass.Order orderResponse = stub.getOrder(id);
  }
}
```
## Server Streaming RPC

![](/image/grpc-pattern-server-streaming.png)

서버가 클라이언트의 요청을 받은 후에, 일련의 응답을 다시 보낸다.
모든 서버 응답을 보낸 뒤에는 서버가 서버의 상태 정보를 후행 메타데이터로 클라이언트에 전송해서 스트림의 끝을 알린다.
OrderManagement 서비스가 일치하는 모든 주문을 한 번에 발송하는 것이 아니라, 주문을 발견하는 대로 보낼 수 있는 기능을 구현해보자.

### Service

우선, 프로토콜 버퍼를 사용해 아래와 같이 서비스 정의를 해야한다.

```protobuf
syntax = "proto3";

import "google/protobuf/wrappers.proto";

package ecommerce;

service OrderManagement {
    rpc searchOrders(google.protobuf.StringValue) returns (stream Order);
}

message Order {
    string id = 1;
    repeated string items = 2;
    string description = 3;
    float price = 4;
    string destination = 5;
}
```

### Server

그리고 아래와 같이, searchOrders 메서드의 로직을 구현한다.

```java
public class OrderMgtServiceImpl extends OrderManagementGrpc.OrderManagementImplBase {
  
  @Override
  public void searchOrders(StringValue request, StreamObserver<OrderManagementOuterClass.Order> responseObserver) {

    for (Map.Entry<String, OrderManagementOuterClass.Order> orderEntry : orderMap.entrySet()) {
      OrderManagementOuterClass.Order order = orderEntry.getValue();
      int itemsCount = order.getItemsCount();
      for (int index = 0; index < itemsCount; index++) {
        String item = order.getItems(index);
        if (item.contains(request.getValue())) {
          logger.info("Item found " + item);
          responseObserver.onNext(order); // 스트림을 통해 일치하는 주문을 보냄.
          break;
        }
      }
    }
    responseObserver.onCompleted();
  }
}
```

### Client

그리고 아래와 같이, searchOrders 메서드를 원격으로 호출하는 클라이언트 로직을 구현하자.

```java
public class OrderMgtClient {

  public static void main(String[] args) {
    ManagedChannel channel = ManagedChannelBuilder.forAddress("localhost", 50051).usePlaintext().build();
    OrderManagementGrpc.OrderManagementBlockingStub stub = OrderManagementGrpc.newBlockingStub(channel);
    OrderManagementGrpc.OrderManagementStub asyncStub = OrderManagementGrpc.newStub(channel);

    // Search Orders
    StringValue searchStr = StringValue.newBuilder().setValue("Google").build();
    Iterator<OrderManagementOuterClass.Order> matchingOrdersItr;
    matchingOrdersItr = stub.searchOrders(searchStr);
    while (matchingOrdersItr.hasNext()) {
      OrderManagementOuterClass.Order matchingOrder = matchingOrdersItr.next();
    }
  }
}
```

## Client Streaming RPC

![](/image/grpc-pattern-client-streaming.png)

클라이언트가 하나의 요청이 아니라, 여러 메세지를 서버로 보내고 서버는 클라이언트에게 단일 응답을 보낸다.
이 때, 서버는 클라이언트가 모든 메세지를 수신해서 응답을 보낼 때 까지 기다릴 필요는 없다.
클라이언트가 updateOrders 메서드로 여러 주문을 업데이트 한다고 하자.

### Service

우선, 프로토콜 버퍼를 사용해 아래와 같이 서비스 정의를 해야한다.

```protobuf
syntax = "proto3";

import "google/protobuf/wrappers.proto";

package ecommerce;

service OrderManagement {
  rpc updateOrders(stream Order) returns (google.protobuf.StringValue);
}

message Order {
    string id = 1;
    repeated string items = 2;
    string description = 3;
    float price = 4;
    string destination = 5;
}
```

### Server

그리고 아래와 같이, updateOrders 메서드의 로직을 구현한다.

```java
public class OrderMgtServiceImpl extends OrderManagementGrpc.OrderManagementImplBase {

  @Override
  public StreamObserver<OrderManagementOuterClass.Order> updateOrders(StreamObserver<StringValue> responseObserver) {
    return new StreamObserver<OrderManagementOuterClass.Order>() {
      StringBuilder updatedOrderStrBuilder = new StringBuilder().append("Updated Order IDs : ");

      @Override
      public void onNext(OrderManagementOuterClass.Order value) {
        if (value != null) {
          orderMap.put(value.getId(), value);
          updatedOrderStrBuilder.append(value.getId()).append(", ");
          logger.info("Order ID : " + value.getId() + " - Updated");
        }
      }

      @Override
      public void onError(Throwable t) {
        logger.info("Order ID update error " + t.getMessage());
      }

      @Override
      public void onCompleted() {
        logger.info("Update orders - Completed");
        StringValue updatedOrders = StringValue.newBuilder().setValue(updatedOrderStrBuilder.toString()).build();
        responseObserver.onNext(updatedOrders);
        responseObserver.onCompleted();
      }
    };
  }
}
```

### Client

그리고 아래와 같이, updateOrders 메서드를 원격으로 호출하는 클라이언트 로직을 구현하자.

```java
public class OrderMgtClient {

  public static void main(String[] args) {
    ManagedChannel channel = ManagedChannelBuilder.forAddress(
        "localhost", 50051).usePlaintext().build();
    OrderManagementGrpc.OrderManagementBlockingStub stub = OrderManagementGrpc.newBlockingStub(channel);
    OrderManagementGrpc.OrderManagementStub asyncStub = OrderManagementGrpc.newStub(channel);
    
    // Update Orders
    invokeOrderUpdate(asyncStub);
  }
  
  private static void invokeOrderUpdate(OrderManagementGrpc.OrderManagementStub asyncStub) {
    OrderManagementOuterClass.Order updOrder1 = OrderManagementOuterClass.Order.newBuilder()
        .setId("102")
        .addItems("Google Pixel 3A").addItems("Google Pixel Book")
        .setDestination("Mountain View, CA")
        .setPrice(1100)
        .build();
    OrderManagementOuterClass.Order updOrder2 = OrderManagementOuterClass.Order.newBuilder()
        .setId("103")
        .addItems("Apple Watch S4").addItems("Mac Book Pro").addItems("iPad Pro")
        .setDestination("San Jose, CA")
        .setPrice(2800)
        .build();
    OrderManagementOuterClass.Order updOrder3 = OrderManagementOuterClass.Order.newBuilder()
        .setId("104")
        .addItems("Google Home Mini").addItems("Google Nest Hub").addItems("iPad Mini")
        .setDestination("Mountain View, CA")
        .setPrice(2200)
        .build();

    final CountDownLatch finishLatch = new CountDownLatch(1);

    StreamObserver<StringValue> updateOrderResponseObserver = new StreamObserver<StringValue>() {
      @Override
      public void onNext(StringValue value) {
        logger.info("Update Orders Res : " + value.getValue());
      }

      @Override
      public void onError(Throwable t) {

      }

      @Override
      public void onCompleted() {
        logger.info("Update orders response  completed!");
        finishLatch.countDown();
      }
    };

    StreamObserver<OrderManagementOuterClass.Order> updateOrderRequestObserver = asyncStub.updateOrders(updateOrderResponseObserver);
    updateOrderRequestObserver.onNext(updOrder1);
    updateOrderRequestObserver.onNext(updOrder2);
    updateOrderRequestObserver.onNext(updOrder3);
    updateOrderRequestObserver.onNext(updOrder3);


    if (finishLatch.getCount() == 0) {
      logger.warning("RPC completed or errored before we finished sending.");
      return;
    }
    updateOrderRequestObserver.onCompleted();

    // Receiving happens asynchronously

    try {
      if (!finishLatch.await(10, TimeUnit.SECONDS)) {
        logger.warning("FAILED : Process orders cannot finish within 10 seconds");
      }
    } catch (InterruptedException e) {
      e.printStackTrace();
    }

  }
}
```

## BidirectionalStreaming RPC

![](/image/grpc-pattern-bidirectional-streaming.png)

클라이언트는 메세지 스트림으로 서버에 요청하고, 서버는 메세지 스트림으로 응답한다.
클라이언트는 연속된 주문 스트림을 전송하고, 배송 위치를 기준으로 주문들을 결합하여 발송으로 처리하는 주문 처리 기능을 구현하자.

### Service

우선, 프로토콜 버퍼를 사용해 아래와 같이 서비스 정의를 해야한다.

```protobuf
syntax = "proto3";

import "google/protobuf/wrappers.proto";

package ecommerce;

service OrderManagement {
  rpc processOrders(stream google.protobuf.StringValue) returns (stream CombinedShipment);
}

message Order {
    string id = 1;
    repeated string items = 2;
    string description = 3;
    float price = 4;
    string destination = 5;
}

message CombinedShipment {
  string id = 1;
  string status = 2;
  repeated Order ordersList = 3;
}
```

### Server

그리고 아래와 같이, processOrders 메서드의 로직을 구현한다.

```java
public class OrderMgtServiceImpl extends OrderManagementGrpc.OrderManagementImplBase {

  // Bi-di Streaming
  @Override
  public StreamObserver<StringValue> processOrders(StreamObserver<OrderManagementOuterClass.CombinedShipment> responseObserver) {

    return new StreamObserver<StringValue>() {
      int batchMarker = 0;
      @Override
      public void onNext(StringValue value) {
        logger.info("Order Proc : ID - " + value.getValue());
        OrderManagementOuterClass.Order currentOrder = orderMap.get(value.getValue());
        if (currentOrder == null) {
          logger.info("No order found. ID - " + value.getValue());
          return;
        }
        // Processing an order and increment batch marker to
        batchMarker++;
        String orderDestination = currentOrder.getDestination();
        OrderManagementOuterClass.CombinedShipment existingShipment = combinedShipmentMap.get(orderDestination);

        if (existingShipment != null) {
          existingShipment = OrderManagementOuterClass.CombinedShipment.newBuilder(existingShipment).addOrdersList(currentOrder).build();
          combinedShipmentMap.put(orderDestination, existingShipment);
        } else {
          OrderManagementOuterClass.CombinedShipment shipment = OrderManagementOuterClass.CombinedShipment.newBuilder().build();
          shipment = shipment.newBuilderForType()
              .addOrdersList(currentOrder)
              .setId("CMB-" + new Random().nextInt(1000)+ ":" + currentOrder.getDestination())
              .setStatus("Processed!")
              .build();
          combinedShipmentMap.put(currentOrder.getDestination(), shipment);
        }

        if (batchMarker == BATCH_SIZE) {
          // Order batch completed. Flush all existing shipments.
          for (Map.Entry<String, OrderManagementOuterClass.CombinedShipment> entry : combinedShipmentMap.entrySet()) {
            responseObserver.onNext(entry.getValue());
          }
          // Reset batch marker
          batchMarker = 0;
          combinedShipmentMap.clear();
        }
      }

      @Override
      public void onError(Throwable t) {

      }

      @Override
      public void onCompleted() {
        for (Map.Entry<String, OrderManagementOuterClass.CombinedShipment> entry : combinedShipmentMap.entrySet()) {
          responseObserver.onNext(entry.getValue());
        }
        responseObserver.onCompleted();
      }

    };
  }
}
```

### Client

그리고 아래와 같이, updateOrders 메서드를 원격으로 호출하는 클라이언트 로직을 구현하자.

```java
public class OrderMgtClient {

  public static void main(String[] args) {
    ManagedChannel channel = ManagedChannelBuilder.forAddress(
        "localhost", 50051).usePlaintext().build();
    OrderManagementGrpc.OrderManagementBlockingStub stub = OrderManagementGrpc.newBlockingStub(channel);
    OrderManagementGrpc.OrderManagementStub asyncStub = OrderManagementGrpc.newStub(channel);
    
    invokeOrderProcess(asyncStub);
  }


  private static void invokeOrderProcess(OrderManagementGrpc.OrderManagementStub asyncStub) {
    final CountDownLatch finishLatch = new CountDownLatch(1);

    StreamObserver<OrderManagementOuterClass.CombinedShipment> orderProcessResponseObserver = new StreamObserver<OrderManagementOuterClass.CombinedShipment>() {
      @Override
      public void onNext(OrderManagementOuterClass.CombinedShipment value) {
        logger.info("Combined Shipment : " + value.getId() + " : " + value.getOrdersListList());
      }

      @Override
      public void onError(Throwable t) {

      }

      @Override
      public void onCompleted() {
        logger.info("Order Processing completed!");
        finishLatch.countDown();
      }
    };

    StreamObserver<StringValue> orderProcessRequestObserver =  asyncStub.processOrders(orderProcessResponseObserver);

    orderProcessRequestObserver.onNext(StringValue.newBuilder().setValue("102").build());
    orderProcessRequestObserver.onNext(StringValue.newBuilder().setValue("103").build());
    orderProcessRequestObserver.onNext(StringValue.newBuilder().setValue("104").build());
    orderProcessRequestObserver.onNext(StringValue.newBuilder().setValue("101").build());

    if (finishLatch.getCount() == 0) {
      logger.warning("RPC completed or errored before we finished sending.");
      return;
    }
    orderProcessRequestObserver.onCompleted();


    try {
      if (!finishLatch.await(120, TimeUnit.SECONDS)) {
        logger.warning("FAILED : Process orders cannot finish within 60 seconds");
      }
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
  }
}
```
---

gRPC 시작에서 운영까지 <카순 인드라시리, 다네쉬 쿠루푸>