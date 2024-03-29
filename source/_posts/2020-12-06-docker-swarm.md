---
layout: post
title: 도커 스웜
date: 2020-12-06
categories: Infrastructure
---

## 도커 스웜을 사용하는 이유

하나의 호스트 머신에서 도커 엔진을 구동하다, CPU, 메모리, 디스크 용량과 같은 자원이 부족해질 수 있다.
이를 해결하기 위한 방법으로, 가장 많이 사용하는 방법이 여러 대의 서버를 클러스터로 만들어 자원을 병렬로 확장하는 것이다.

그러나, 여러 대의 서버를 하나의 자원 풀로 만드는 것은 쉽지 않다.
에를 들면,

- 새로운 서버나 컨테이너가 추가되었을 때 이를 발견 하는 작업 (Service Discovery)
- 어떤 서버에 컨테이너를 할당할 것인가에 대한 스케쥴러
- 클러스터 내의 서버가 다운 되었을 때 고가용성 보장

이러한 문제를 쉽게 해결할 수 있는 솔루션으로 도커에서 공식적으로 제공하는 Docker Swarm 과 Swarm Mode 를 사용할 수 있다.

## 스웜 클래식과 도커 스웜 모드

도커 스웜에는 두 가지 종류가 있다.

1. 스웜 크래식: 도커 버젼 1.6 이후부터 사용할 수 있는 컨테이너로서의 스웜
2. 스웜 모드: 도커 버젼 1.2 이후부터 사용할 수 있는 도크 스웜 모드

위 둘의 차이는, 목적이다.

1. 스웜 클래식: 여러 대의 도커 서버를 하나의 지점에서 사용하도록 단일 접근점을 제공한다.
2. 스웜 모드: 마이크로서비스 아키텍처의 컨테이너를 다루기 위한 클러스터링 기능에 초점을 둔다.

또 다른 차이로 분산 코디네이터, 에이전트와 같은 클러스터 툴이 별도로 구도되느냐이다.
여러 도커 서버를 하나의 클러스터로 구성하려면, 각종 정보를 저장하고 동기화하는 분산 코디네이터, 클러스터 내의 서버를 관리하고 제어하는 메니저, 각 서버를 제어하는 에이전트가 필요하다.

1. 스웜 클래식: 이들이 별도로 실행되야아한다.
2. 스웜 모드: 도커 엔진 자체에 내장되어 있다.

대규모 클러스터에서 서비스를 운영한다면, 스웜 클래식보다 스웜 모드를 사용하자.
스웜 모드를 사용한다면 더 많은 기능을 쉽게 사용할 수 있다.
또한, 스웜 클래식은 도커 공식 문서에서도 legacy 로 언급하며 유지 보수가 활발하지 않다.

## 도커 스웜모드의 구조

스웜 모드는 메니저 노드와 워커 노드로 구성되어 있다.
워커 노드는 실제로 컨테이너가 생성되고 관리되는 도커 서버이고, 메니저 노드는 워커 노드를 관리하기 위한 도커 서버이다.

운영 환경에서는, 메니저 노드를 다중화하자.
메니저의 부하를 분산하고, 특정 메니저 노드가 다운되었을 때 정상적으로 스웜 클러스터를 유지할 수 있기 때문이다.

## 스웜 모드 서비스

도커 명령어의 제어 단위는 컨테이너이다.
docker run 명령어는 컨테이너를 생성하고 docker rm 명령어는 컨테이너를 삭제한다.

스웜 모드에서 제어하느 단위는, Service 이다.
서비스는 같은 이미지에서 생성된 컨테이너의 집합이다.
서비스를 제어하면, 해당 서비스 내의 컨테이너에 같은 명령어가 수행된다.
서비스 내 컨테이너는 한 개 이상 존재할 수 있으며, 컨테이너들은 각 워커 노드와 메니저 노드에 할당된다.
이러한 컨테이너들을 Task 라고 부른다.

함께 생성된 컨테이너는 Replica 라고 한다.
서비스에 설정된 레플리카의 수 만큼 컨테이너가 스웜 클러스터 내에 존재해야한다.
스웜은 서비스의 컨테이너들에 대한 상태를 계속 확인하고 서비스 내에 정의된 레플리카의 수 만큼, 컨테이너가 스웜 클러스터 내에 존재하지 않으면, 새로운 컨테이너 레플리카를 생성한다.

---

시작하세요! 도커/쿠버네티스 <용찬호>
