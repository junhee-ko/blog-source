---
layout: post
title: "사용자 수에 따른 규모 확장성" 
date: 2023-09-02
categories: Design
---

한 명의 사용자를 지원하는 시스템에서, 몇백만 사용자를 지원하는 시스템을 설계해보자.

## 단일 서버

모든 컴포넌트가 한 대의 서버에서 실행되는 간단한 시스템부터 설계하자.

1. 사용자는 도메인 이름을 이용해서 웹 사이트에 접속한다. 
2. 이 접속을 위해서 DNS 에 질의해서 IP 주소로 변환하는 과정이 필요하다. DNS 조회 결과로 IP 주소가 반환된다.
3. 해당 IP 주소로 HTTP 요청이 전달된다.
4. 요청을 받은 웹 서버는 HTML 페이지나 JSON 형태의 응답을 반환한다.

## 데이터베이스

사용자가 늘면 서버 하나로 충분하지 않다.
웹/모바일 트래픽을 처리하는 서버 (웹 계층) 와 계층과 데이터베이스 서버 (데이터 계층) 을 분리해서 각각을 확장할 수 있도록 하자.

## 수직적 규모 확장, 수평적 규모 확장

scale up 이라고도 하는 수직적 규모 확장은 고사양 자원 (CPU, RAM ..) 을 추가하는 행위다.
scale out 이라고도 하는 수평적 규모 확장은 더 많은 서버를 추가해서 성능을 개선하는 행위다.

서버로 유입되는 트래픽의 양이 적을 때는 수직적 확장이 좋은 선택이다. 
가장 큰 장점은 단순함이다. 하지만,

1. 한대의 서버에 CPU 나 메모리를 무한대로 증설항 방법은 없다.
2. 자동 복구 (failover) 나 다중화가 불가능해서 서버 장애가 발생하면 완전히 중단된다.

### 로드 밸런서

부하 분산 집합에 속한 웹 서버들에게 트래픽 부하를 고르게 분산하는 역할을 한다.

사용자는 로드밸런서의 public IP address 로 접속한다. 그래서 웹 서버는 클라이언트의 접속을 직접 처리 하지 않는다.
보안을 위해서는 서버 간 통신에 private IP address 를 사용한다. 로드밸랜서는 웹 서버와 통신을 하기 위해 이 주소를 private IP address 를 사용한다.

부하 분산 집합에 웹 서버를 추가하면 자동 복구 (failover) 가 가능하며 가용성은 향상된다. 즉,

1. 서버 1이 다운되면 서버 2로 전송된다. 그래서 웹 사이트 전체가 다운되는 일이 방지된다.
2. 트래픽이 증가하면, 웹 서버 계층에 서버를 추가하면 로드밸런서가 자동으로 트래픽을 분산한다.

### 데이터베이스 다중화

서버 사이에 master-slave 관게를 설정하고 데이터 원본은 master 서버에, 사본은 slave 서버에 저정한다.
master 서버는 write operation 만 지원하고, slave 서버는 read operation 만 지원한다.
대부분의 애플리케이션은 읽기 연산의 비중이 높기 때문에 slave 서버의 수가 master 서버 수 보다 많다.
데이터베이스를 다중화하면,

1. 성능: 모든 데이터 변경 연산은 master 서버에, 읽기 연산은 slave 서버에 전달되기 때문에 병렬로 처리할 수 있는 query 수가 늘어나므로 성능이 좋아진다.
2. 안정성 (reliability): 데이터베이스를 지역적으로 떨어진 여러 장소에 다중화시켜놓으면, 서버 가운데 일부가 파괴되어도 데이터는 보존된다.
3. 가용성 (availability): 하나의 데이터베이스 서버에 장애가 발생해도 다른 서버에 있는 데이터를 가져와 계속 서비스할 수 있다.

데이터베이스 서버 가운데 하나가 다운되면,

1. slave 서버가 한 대인데, slave 서버가 다운되면: 읽기 연산은 모두 master 서버로 일시적으로 전달된다.
2. slave 서버가 여러 대인데, slave 서버가 다운되면: 읽기 연산은 나머지 slave 서버들에 분산된다.
3. slave 서버가 한 대인데, master 서버가 다운되면: slave 서버가 새로운 master 서버가 되며, 모든 연산을 일시적으로 새로운 master 서버에서 수행된다.

이제 응답시간 (latency) 를 cache 와 CDN 을 활용해서 개선해보자.

## 캐쉬

값 비싼 연산 결과나 자주 참조되는 데이터를 메모리 안에 두고, 요청이 더 빨리 처리될 수 있도록 하는 저장소이다.
데이터가 캐쉬에 있으면 캐쉬에서 읽고, 없으면 DB 에서 읽어 캐쉬에 쓴다.

캐쉬를 사용할 때는 다음 내용들을 고려하자.

1. 캐쉬 사용 상황: 데이터 갱신은 자주 일어나지 않지만, 참조는 빈번하게 일어날 때 캐쉬를 사용하기에 적절하다.
2. 데이터: 영속적으로 보관할 데이터는 캐쉬에 두는 것이 적절치 않다.
3. expire: 너무 짧으면 DB 를 자주 읽게 되고 너무 길면 원본과 차이가 날 가능성이 높다.
4. consistency: 저장소의 원본을 갱신하는 연산과 캐쉬를 갱신하는 연산이 단일 트랜잭션으로 처리되지 않으면 일관성이 깨질 수 있다.
5. 장애 대처: 캐쉬 서버가 한 대이면 SPOF 가 될 수 있다. 여러 지역에 걸쳐 캐쉬 서버를 분산해야한다.
6. 메모리 크기: 너무 작으면 데이터가 너무 자주 캐쉬에서 eviction 되어서 성능이 떨어진다. 메모리를 과할당 하면 보관할 데이터가 갑자기 늘어났을 때 생기는 문제를 방지할 수 있다.
7. eviction: 캐쉬가 꽉 차면 추가로 캐쉬에 데이터를 넣어야할 경우 기존 데이터를 내보내야한다. LRU, LFU, FIFO 등의 방법이 있다.

## CDN

정적 컨텐츠를 전송하기 위해 쓰이는, 지리적으로 분산된 서버의 네트워크이다. 이미지, 비디오, CSS, JavaScript 파일 등을 캐쉬할 수 있다.

1. 사용자가 이미지 URL 을 이용해서 image.png 에 접근한다.
2. CDN 서버의 캐쉬에 해당 이미지가 없으면, 서버는 원본 서버에 요청해서 파일을 가져온다.
3. 원본 서버가 파일을 CDN 서버에 반환한다.
4. CDN 서버는 파일을 캐쉬하고 사용자에게 반환한다. 이미지는 HTTP 헤더의 TTL 에 명시된 시간이 끝날 때까지 캐쉬된다.
5. 다시 사용자가 같은 이미지에 대한 요청을 CDN 서버에 전송하면, 만료되지 않은 이미지에 대한 요청은 캐쉬를 통해 처리된다. 

CDN 사용시 고려할 사항으로,

1. 비용: 자주 사용되지 않은 컨텐츠를 캐슁하면 이득이 크지 않다.
2. 만료 기간: time-sensitive 컨텐츠는 만료 시점을 잘 정해야한다. 너무 길면 신선도가 떨어지고, 너무 짧으면 원본 서버에 빈번히 접속해야한다.
3. 장애 대처: CDN 서버가 일시적으로 응답하지 않으면, 원본 서버로부터 직접 컨텐츠를 가져오도록 클라이언트를 구성해야한다.
4. 컨텐츠 무효화: 아직 만료되지 않은 컨텐츠여도 CDN 에서 제거할 수 있어야한다. (API or object versioning)

## 무상태 웹 계층

웹 계층을 수평적으로 확장하는 방법을 보자. 상태 정보 의존적 아키텍처와 무상태 아키텍를 비교하자.

상태 정보 의존적 아키텍처는, 

1. 사용자 A 의 세션 정보나 프로파일 이미지 같은 상태 정보를 서버 1에 저장한다.
2. 사용자 A 를 인증하기 위해서서 HTTP 요청이 반드시 서버 1로 전송되어야한다.

같은 클라이언트의 요청은 항상 같은 서버로 전송되어야하는데, 로드 밸랜서가 이를 지원하기 위해 sticky session 기능을 제공한다.
하지만, 이것은 로드밸런서에게 부담을 주고 뒷단 서버를 추가하거나 제거하기가 까다로워진다. 장애 처리도 복잡해진다.

무상태 아키텍처는, 사용자의 HTTP 요청이 어떤 웹 서버로도 전달될 수 있다.
웹 서버는 상태 정보가 필요하면 shared storage 로부터 가져온다.
상태 정보가 웹 서버로부터 물리적으로 분리되어 있기 때문에, 단순하고 안정적이며 규모 확장이 쉽다.

## 데이터센터

장애가 없는 상황에서 사용자는 가장 가까운 데이터 센터로 안내되는데, 이를 지리적 라우팅이라고 한다.
geoDNS 는 사용자의 위치에 따라 도메인 이름을 어떤 IP 주소로 반환할지 결정해주는 DNS 이다.
데이터 센터 중 하나가 장애가 발생하면, 모든 트래픽은 장애가 없는 데이터 센터로 전송된다.

데이터센터 다중화를 위해서는,

1. 트래픽 우회: 올바른 데이터 센터로 트래픽을 보내는 효과적인 방법을 찾아야한다.
2. 데이터 동기화: 트래픽이 다른 데이터베이스로 우회되어도 해당 데이터센터에는 데이터가 없을 수도 있다. 이를 위해, 데이터를 데이터센터 간에 다중화가 필요하다.

## 메시지 큐

시스템을 더 큰 규모로 확장하기 위해서는 시스템의 컴포넌트를 분리해서 독립적으로 확장할 수 있도록 해야한다.
메시지 큐는, 이 문제를 해결하기 위한 전략 가운데 하나이다.

메시지 큐를 이용하면, 서버 간 결합이 느슨해져셔 규모 확장성이 보장되어야하는 안정적 애플리케이션을 구성하기 좋다.
생산자는 소비자 프로세스가 다운되어도 메세지를 발행할 수 있고, 소비자는 생산자 서비스가 가용 상태가 아니더라도 메시지를 수신할 수 있다.

## 로그, 메트릭, 자동화

1. 로그: 시스템 오류와 문제들을 쉽게 찾아 내기 위해, 로그 모니터링을 해야한다.
2. 메트릭: 사업을 위한 유용한 정보나 시스템의 현재 상태를 얻을 수 있다.
3. 자동화: 생산성을 높이기 위해 자동화 도구를 사용할 수 있다. ex) CI 도구, 빌드/테스트/배포 자동화

## 데이터베이스 규모 확장

저장할 데이터가 많아지면 DB 에 대한 부하도 증가한다. 수직정 확장, 수평적 확장 두 가지 접근 방법이 있다.

### 수직정 확장

더 많은 또는 고성능의 자원 (CPU, RAM, Disk..) 를 증설하는 방법이다. 하지만,

1. DB 서버 하드웨어에는 한계가 있으므로 무한 증설할 수 없다.
2. SPOF 로 인한 위험성이 크다.
3. 비용이 비싸다.

### 수평적 확장

DB 의 수평적 확장을 sharding 이라고 하며, 더 많은 서버를 추가해서 성능을 향상시킬 수 있도록 한다.
모든 샤드는 같은 스키마를 쓰지만, 샤드에 보관된 데이터 사이에 중복은 없다.

1. sharding key: 데이터가 어떻게 분산될지 정하는 하나 이상의 칼럼으로 구성된다. 데이터를 고르게 분산하는 것이 중요하다. 
2. resharding: 데이터가 많아져 하나의 샤드로 더 이상 감당 못할 때나, 샤드 간 데이터 분포가 균등하지 못해 특정 샤드에 할당된 공간 소모가 다른 샤드에 비해 빨리 진행될때 필요하다.
3. celebrity 문제: hotspot key 문제라고도 부르는데, 특정 샤드에 질의가 집중되어 서버에 과부하가 걸리는 문제다.
4. join and de-normalization: 여러 샤드에 걸친 데이터를 조인하기가 힘들어진다. 비정규화하여 하나의 테이블에서 질의가 수행되도록 할 수 있다.

---

System Design Interview <알렉스 쉬>