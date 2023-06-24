---
layout: post
title: "쿠버네티스 애플리케이션 상태와 배포"
date: 2020-12-15
categories: Infrastructure
---

쿠버네티스는 애플리케이션을 안정적으로 배포하기 위한 몇 가지 기능을 제공한다.

## ReCreate

일시적으로 사용자의 요청을 처리하지 못해도 괜찮은 애플리케이션이면, ReCreate 방법을 사용할 수 있다.
기존 버전의 포드를 모두 삭제한 뒤, 새로운 버젼의 포드를 생성하는 방식이다.

## 디플로이먼트를 통해 롤링 업데이트

디플로이먼트를 업데이트하는 도중에 사용자의 요청을 처리하기 위해서는, 롤링 업데이트 기능을 사용하면 된다.
롤링 업데이트 도중에 기존 버전의 포드를 몇 개씩 삭제할지, 새로운 버전의 포드를 몇 개식 생성할지는 직접 설정 가능하다.

### 블루 그린 배포

기존 버전의 포드를 그대로 놔둔 상태에서, 새로운 버전의 포드를 미리 생성 한 뒤 서비스의 라우팅만 변경해서 배포하는 방식이다.
롤링 업데이트 방식과 다르게, 두 버젼의 애플리케이션이 공존하지 않으며 ReCreate 전략 처럼 중단 시간이 발생하지도 않는다.
하지만, 특정 순간에는 디플로이먼트에 설정된 replicas 개수의 두 배에 해당하는 포드가 실행되어, 일시적으로 전체 자원을 많이 소모할 수 있다.

## 포드의 Lifecycle

새로운 포드가 생성되어 Running 상태가 되었어도, 애플리케이션의 초기화 작업 등으로 사용자의 요청을 아직 처리할 준비가 되지 않은 상태일 수 있다.
또한, 기존의 포드를 종료할 때, 처리중인 요청을 전부 완료한 뒤에 포드를 종료시켜야한다.

이를 위해 알아야 할 포드의 라이프사이클에 대해 정리한다.

### 포드의 상태와 생애주기

1. Pending: 포드 생성 요청이 API 서버에 승인되었지만, 아직 실제로 생성되지 않은 상태. ex) 아직 노드에 스케쥴링이 안됨
2. Running: 포드가 정상적으로 실행된 상태
3. Completed: 포드가 정상적으로 종료된 상태
4. Error: 포드가 정상적으로 실행되지 않은 상태로 종료된 상태
5. Terminating: 포드가 삭제 또는 eviction 되기 위해 삭제 상태에 머물러 있는 상태

### Completed, Error, CrashLoopBackOff

함수가 종료되면 특정 값을 반환하듯이, 리눅스의 프로세스 또한 종료될 때 종료 코드를 반환한다. 
컨테이너 내부의 프로세스 또한 종료될 때 종료 코드를 반환하는데, 컨테이너의 init 프로세스가 어떤 값을 반환하냐에 따라 포드의 상태가 Completed or Error 상태가 된다.

1. 0 을 종료 코드로 반환하면 Completed 상태
2. 1 을 반환하면 Error 상태

k8s 에서는 어떠한 작업이 실패하면, 일정 간격을 두고 해당 작업을 재시도한다. 실패 횟수가 늘어날수록 재시도 간격이 지수 형태로 늘어난다.
이 때, 중간 상태가 CrashLoopBackOff 이다.

### Running 상태

포드가 Running 상태라고 해서, 컨테이너 내부의 애플리케이션이 제대로 동작함을 보장하지 않는다. 
이를 위해, k8s 에서는 다음 기능을 제공한다.

1. Init Container
2. postStart
3. livenessProbe, readinessProbe

#### Init Container

포드의 컨테이너 내부에서 애플리케이션이 실행되기 전에 초기화를 수행하는 컨테이너이다.
애플리케이션 컨테이너보다 먼저 실행되어, 특정 작업을 미리 수행하는 용도로 사용될 수 있다.

#### postStart

포드의 컨테이너가 실행되거나 삭제될 때, 특정 작업을 수행하도록 Hook 을 YAML 파일에 정의할 수 있다.

1. postStart: 컨테이너가 실행될 때 실행됨
2. preStop: 컨테이너가 종료될 때 실행됨

postStart 는 두 가지 방식으로 사용 가능하다.

1. HTTP: 컨테이너가 시작된 직후, 특정 주소로 HTTP 요청을 전송
2. Exec: 컨테이너가 시작된 직후, 컨테이너 내부에서 특정 명령어를 실행

명령어나 HTTP 요청이 제대로 실행되지 않으면 컨테이너는 Running 상태로 전환되지 않는다.

#### livenessProbe, readinessProbe

Init Container 나, postStart 훅이 정상 실행되었어도 애플리케이션이 제대로 동작한다는 보장이 없다.
다른 이유로, 사용자의 요청을 처리하지 못하는 상태일 수 있기 때문이다.

사용자의 요청을 처리하는 상태인지 판별하기 위해, 아래 두 방법을 사용할 수 있다.

1. livenessProbe: 컨테이너 내부의 애플리케이션이 liveness 인지 검사한다. 실패하면, 컨테이너는 restartPolicy 에 따라 재시작한다.
2. readinessProbe: 컨테이너 내부의 애플리케이션이 사용자 요청 처리 준비가 되었는지 인지 검사한다. 실패하면, 서비스의 라우팅 대상에서 제외된다.

readinessProbe 상태 검사에 실패했어도, 시간에 지나면 초기화 작업이 완료되어 준비 상태가 될 수 있기 때문에 일시적으로  포드를 서비스의 라우팅에서 제외하는 작업만 수행한다. 

livenessProbe, readinessProbe 는 아래 세 가지 중 하나로 검사할 수 있다.

1. httpGet: HTTP 요청을 전송해 상태 검사
2. exec: 컨테이너 내부에서 명령어를 실행해 상태 검사
3. tcpSocket: TCP 연결이 수립될 수 있는지 체크해서 상태 검사

---

시작하세요! 도커/쿠버네티스 <용찬호>