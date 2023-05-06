---
layout: post
title: "쿠버네티스 스케쥴링"
date: 2020-12-15
categories: Infrastructure
---

스케쥴링이란, 컨테이너나 가상 머신 같은 인스턴스를 새로 생성할 때 어느 서버에 생성할 것인지 결정하는 것이다.

## 여러 핵심 컴포넌트

k8s 에는 여러 핵심 컴포넌트들이 kube-system namespace 에 실행되고 있다.

1. kubectl 로 상호 통신할 수 있는 kube-apiserver
2. 스케쥴링에 관여하는 kube-scheduler
3. 쿠버네티스 클러스터의 전반적인 상태 데이터를 저장하는 etcd

### etcd 

etcd 는 분산 코디네이터의 일종으로, 현재 생성된 디플로이먼트나 포드의 목록과 정보, 클러스터 자체 정보 등의 데이터를 저장한다.
분산 코디네이터에는, Zookeeper, Consul 등도 있다.

etcd 에 저장된 데이터는 무조건 kube-apiserver 를 통해서만 접근 가능하다.
즉, kubectl get pods 명령어를 실행하면,

1. API server 에 요청이 전달되고
2. API server 는 etcd 의 데이터를 읽어와 kubectl 사용자에게 반환한다.

## 포드가 노드에 생성되는 과정

사용자가 kubectl 이나 API 서버로 포드 생성 요청을 하면,

1. ServiceAccount, RoleBinding 등의 기능으로 포드 생성을 요청한 사용자의 인증 및 인가 작업 수행
2. ResourceQuata, LimitRange 같은 어드미션 컨트롤러가 해당 포드 요청을 mutate 하거나 validate 
3. 포드 생성이 승인되면, 해당 포드를 워커 노드 중 하나에 생성

여기서, 스케쥴링은 3번에서 수행된다.

## 스케쥴링 과정

1. 포드 생성 요청이 최종 승인되면, API 서버는 etcd 에 포드의 데이터를 저장한다. 이 때, nodeName 항목을 설정하지 않은 상태로 etcd 에 저장한다.
2. kube-scheduler 컴포넌트는 API 서버의 Watch 를 통해 nodeName 이 비어있는 포드 데이터가 저장되어 있다는 사실을 전달받는다.
3. kube-scheduler 는 해당 포드를 스케쥴링 대상으로 판단하고, 할당할 적절한 노드를 선택해서 API 서버에 해당 노드와 포드를 바인딩 할 것을 요청한다.
4. 그리고 포드의 nodeName 항목의 값에 선택된 노드의 이름이 설정된댜.
5. 클러스터의 각 노드에 실행 중인 kubelet 은 API 서버에 걸어둔 Watch 를 통해 포드의 nodeName 이 설정되었다는 것을 전닫받는다.
6. 그리고, nodeName 에 해당하는 노드의 kubelet 이 컨테이터 런타임을 통해 포드를 생성한다.

## 포드가 생성될 노드를 선택하는 스케쥴링 과정

스케쥴러가 적절한 노드를 선택하는 과정은, 크게 두 순서이다.

1. 노드 필터링: 포드를 할당할 수 있는 노드와 그렇지 않은 노드를 filtering
2. 노드 스코어링: k8s 의 소스코드에 미리 정의된 알고리즘의 가중치 기반으로 노드의 점수를 계산

## Node Selector, Node Affinity

### Node Selector

특정 워커 노드에 포드를 할당하는 가장 확실한 방법은, 포드의 YAML 파일에 노드의 이름을 명시하는 것이다.
노드의 이름을 고정으로 설정하기 때문에, 이 방식은 YAML 파일을 보편적으로 사용할 수 없다. 
또한, 노드에 장애가 발생하면 유여한게 대처하기 힘들다.

그래서, 노드의 라벨을 이용하는 방법이 있다.
라벨을 이용해서 특정 라벨이 존재하는 노드에만 포드를 할당하는 것이다.
노드에 특정 라벨을 추가하고, 포드의 YAML 파일에 nodeSelector 을 정의하면 된다.

### Node Affinity

nodeSelector 에서 더 확장된 라벨 선택 기능을 제공하고, 반드시 충족되어햐는 조건과 선호하는 조건을 별도 정의 가능하다.
예를 들면, YAML 파일에 "operator:In" 항목에 key 의 라벨이 values 값 중 하나를 만족하는 노드에 포드를 스케쥴링한다라는 방식으로 사용 가능하다. 

## Taints, Tolerations

라벨 이외에도, Taints 라는 방법으로 포드를 할당할 노드를 선택할 수 있다.
특정 노드에 얼룩 (Taints) 을 지정해서 해당 노드에 포드가 할당되는 것을 막는 기능이다.
Taints 에 대응하는 Tolerations 을 포드에 설정하면 Taints 가 설정된 노드에도 포드를 할당 할 수 있다. 

## Cordon, Drain, PodDistributionBudget

### Cordon

Taints, Tolerations 를 이용해서 노드에 포드가 스케쥴링되는 것을 막을 수도 있지만, 더 명시적인 방법을 사용할 수 있다.
아래처럼, kubectl cordon 명령어로 해당 노드에 더 이상 포드가 스케쥴리 되지 않도록 할 수 있다.

```bash
$ kubectl cordon {node name}
```

### Drain

drain 은 cordon 처럼 해당 노드에 스케쥴링을 금지하는 것은 동일하다.
하지만, drain 은 노드에서 기존에 실행중이던 포드를 다른 노드로 eviction 을 수행한다는 것이 다르다.

### PodDistributionBudget

drain 은, eviction 과정 중에 애플리케이션이 중단될 수 있다.
PodDistributionBudget 은, 특정 개수 또는 비율만큼의 포드가 반드시 정상적인 상태를 유지하기 위해 사용된다.

maxUnavailable, minAvailable 두 가지 중 하나를 사용하면 된다.

1. maxUnavailable: kubectl drain 에 의해 노드의 포드가 종료될 때, 최대 몇 개 까지 동시에 종료될 수 있는지를 의미한다.
2. minAvailable: 최소 몇개의 포드가 정상 상태를 유지해야하는지를 의미한다.

---

시작하세요! 도커/쿠버네티스 <용찬호>