---
layout: post
title:  "카프카란 무엇인가"
date:   2019-12-21
categories: Kafka
---

카프카 : 대용량, 대규모 메세지 데이터를 빠르게 처리하도록 개발된 메시징 플랫폼

## 탄생 배경

end-to-end 아키텍쳐의 문제점은,

1. 통합된 전송 영역이 없어서 복잡도 증가하고
2. 데이터 파이프라인의 관리가 어렵다.

하지만 카프카는 아래 그림 처럼,

1. 모든 시스템으로 데이터 전송이 가능하고
2. 실시간 처리 가능하고
3. 확장이 용이하다.

![](/image/kafka_structure.png)

## 동작 방식과 원리

메시징 시스템을 먼저 살펴보자.
중앙에 메시징 시스템 서버를 두고 메시지를 Publish 하고 Subscribe 하는 형태의 통신을 Pub/Sub 모델이라고 한다.

![](/image/message-system-pub-sub.png)

프로듀서가 메시지를 컨슈머에게 직접 전달하는 것이 아니라, 중간의 메시징 시스템으로 전달한다. 그래서,

1. 개체가 하나 빠지거나 수신 불능 상태가 되어도, 메세징 시스템이 살아있으면 프로듀서에서 전달된 메세지가 유실이 되지 않는다.
2. 메세징 시스템을 중심으로 연결되기 때문에, 확장성이 용이하다.

카프카의 메세지 전달 순서는,

1. 프로듀서가 메세지를 카프카에게 보낸다.
2. 메세지가 Topic 에 도착해 저장된다.
3. 컨슈머는 카프카 서버에 접속해 메세지를 가져간다. 

## 특징

1. 프로듀서와 컨슈머의 분리
   어느 한쪽 시스템에서 문제가 발생해도 연쇄 작용이 발생할 확률이 낮다.
   서버 추가에 대한 부담이 준다.

2. 멀티 프로듀서, 멀티 컨슈머
   데이터 분석 및 처리 프로세스에서 하나의 데이터를 다양한 용도로 사용하는 요구 사항을 충족할 수 있다.

3. 디스크에 메세지 저장
   컨슈머가 메세지를 읽어가도 보관 주기 동안 디스크에 메세지를 저장한다.
   ( 일반적인 메세지 시스템은 컨슈머가 메세지를 읽어가면 큐에서 바로 메세지 삭제 )

4. 확장성
   온라인 상태에서 확장 작업 가능

5. 높은 성능

---

카프카, 데이터 플랫폼의 최강자 <고승범, 공용준>