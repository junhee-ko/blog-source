---
layout: post
title: Prometheus > PromQL
date: 2023-05-13
categories: Infrastructure
---

## 집계

PromQL 로 처리할 수 있는 집계 쿼리들을 정리한다.

### Gauge

게이지 매트릭은 상태의 스냅샷이다. 
보통 합계, 평균, 최솟값, 최댓값을 구하기 위해 게이지를 집계한다.

다음과 같은 시계열 데이터가 있다고 하자.

```text
node_filesystem_size_bytes{device="/dev/sda1",fstype="vfat",instance="localhost:9100",job="node",mountpoint="/boot/efi"} 124214
node_filesystem_size_bytes{device="/dev/sda5",fstype="ext4",instance="localhost:9100",job="node",mountpoint="/"} 43543534
node_filesystem_size_bytes{device="tempfs",fstype="tempfs",instance="localhost:9100",job="node",mountpoint="/run"} 565465465
node_filesystem_size_bytes{device="tempfs",fstype="tempfs",instance="localhost:9100",job="node",mountpoint="/run/lock"} 3213545
node_filesystem_size_bytes{device="tempfs",fstype="tempfs",instance="localhost:9100",job="node",mountpoint="/run/user/1000"} 67354
```

아래 수식은, 각 머신의 전체 파일 시스템 크기를 계산한다.

```text
sum without(device, fstype, mountpoint)(node_filesystem_size_bytes) 
```

이 표현식은, 

1. without 을 통해 세 개의 레이블을 무시하고 
2. 레이블이 같은 모든 메트릭을 합치도록 sum 집계 연산자에 지시한다.

결과는 다음과 같다

```text
{instance="localhost:9100",job="node"} 128402194
```

device, fstype, mountpoint 레이블이 없어졌다. 
계산이 수행된 결과는 더 이상 node_filesystem_size_bytes 메트릭이 아니기 때문에, 메트릭 이름도 없다.

### Counter

카운터는 이벤트의 개수나 크기를 추적하며, /metrics 경로에 표시되는 값은 애플리케이션이 시작된 이후의 총합이다.
시간에 따라 카운터가 얼마나 빨리 증가하는지의 여부가 주로 중요하기 때문에, rate 함수를 많이 사용한다.

예를 들어, 초 단위로 수신된 네트워크 트래픽의 양을 계산해보자.

```text
rate(node_network_receive_bytes_total[5m])
```

5m 은 5분동안의 데이터에 대해 rate 가 제공된다는 것을 의미하기 때문에 결과는 지난 5분동안의 평균이다.

```text
{device="lo",instance="localhost:9100",job="node"} 18120.12323213123
{device="wlan0",instance="localhost:9100",job="node"} 13123.12323213123
```

rate 의 결과는 게이지이기 때문에, 게이지와 동일한 집계 연산을 적용할 수 있다.
머신별로 1초당 수신한 전체 바이트 수를 얻을 수 있다.

```text
sum without(device)rate(node_network_receive_bytes_total[5m])
```

결과는,

```text
{instance="localhost:9100",job="node"} 31243.12323213123
```

### Summary

서머리 매트릭에는 _sum 과 _count 점미어가 포함된다. 
_sum 과 _count 는 모두 카운터이다.

프로메테우스는 HTTP API 가 반환하는 데이터의 양에 대해 http_response_size_bytes 서머리 메트릭을 expose 한다.
http_response_size_bytes_count 는 요청의 개수를 추적한다.

아래 수식은 초당 HTTP 요청 비율의 총합이다.

```text
sum without(handler)(rate(http_response_size_bytes_count)[5m])
```

마찬가지로, http_response_size_bytes_sum 은 각 핸들이 반환한 바이트 수에 대한 카운터이다.

```text
sum without(handler)(rate(http_response_size_bytes_sum)[5m])
```

서머리의 강점은, 이벤트의 평균 크기를 계산할 수 있단는 것이다. 
크기가 1,4,7 인 세개의 응답이 있으면 평균은 12 를 3으로 나눈 4가 된다.

```text
sum without(handler)(rate(http_response_size_bytes_sum)[5m])
/
sum without(handler)(rate(http_response_size_bytes_count)[5m])
```

## Selector

레이블에 의한 제한은 Selector 를 사용해 수행된다. 예를 들어,

```text
process_resident_memory_bytes{job="node"}
```

process_resident_memory_bytes 이름과 "node" job 레이블을 가지는 모든 시계열을 반환하는 실렉터이다.
이 특정 실렉터는, 주어진 시점에 주어진 시계열의 값을 반환하므로, instant vector selector 이다.

job="node" 는 matcher 라고 하며, 하나의 selector 에 AND 로 처리된 여러 matcher 가 있을 수 있다.

### Matcher

매처에는 네 종류가 있다.

1. = : 동등 매처. 반환된 시계열이 주어진 레이블 값과 동일한 레이블 이름을 갖는지 확인
2. != : 부정 동등 매처. 반환된 시계열이 주어진 레이블 값과 동일한 레이블 이름을 갖지 않도록 지정
3. =~ : 정규 표현식 매처. 반환된 시계열에 대한 레이블 값이 해당 정규 표헌식과 일치하도록 지정
4. !~ : 부정 정규 표현식 매처. 정규 표현식을 기반으로 레이블 값 제외

### Instant Vector Selector

인스턴트 백터 실렉터는 쿼리 평가 시점 전 가장 최근 샘플의 instant vector 를 반환하며, 0 개 이상의 시계열 목록으로 표현된다.
각 시계열에는 샘플이 하나씩 있고, 샘플에는 값과 타임스탬프가 포함된다. 

### Range Vector Selector

범위 백터 실렉터는, 인스턴트 백터 실렉터와 다르게 각 시계열에 대해 많은 샘플을 반환한다.
항상 rate 함수와 같이 사용된다.

```text
rate(process_cpu_seconds_total[1m])
```

1m 은 인스턴트 백터 실랙터를 범위 벡터 실랙터로 전환하고, 쿼리 평가 시점까지 1분 동안 실렉터와 일치하는 모든 시계열을 반환하도록 PromQL 에 지시한다.
범위 벡터를 직접 보는 경우는 거의 없다. 그래서, 대부분 범위 벡터를 인수로 취하는 rate 나 avg_over_time 같은 함수와 함께 범위 벡터를 사용한다.

### Offset

오프셋은 쿼리에 대한 평가 시간을 가질 수 있게 하며, 시간을 되돌린다.
아래 수식은, 쿼리 평가 시점보다 1시간 전에 메모리 사용량을 얻는다.

```text
process_resident_memory_bytes{job="node"} offset 1h
```

## HTTP API

프로메테우스는 많은 API 를 제공한다. 모든 엔드포인트는 /api/v1 경로 아래에 있다.

### Query

/api/v1/query 는 PromQL 표현식을 주어진 시간에 실행하고 결과를 반환한다.

ex) http://localhost:9090/api/v1/query?query=process_resident_memory_bytes

### Query Range

/api/v1/query_range 는 그래프 작성에 사용할 엔드포인트이다.
query 엔드포인트에 대한 다중 호출을 위한 편의 문법이다.

start 시간, end 시간, step 을 제공한다.
/api/v1/query_range 는 start 시간에 처음 시작되고, step 초 동안 실행된다. 
그 다음 시작 후 두 번째 step 초에 실행되고, 계속 반복 수행되다가 쿼리 해석 시간이 end 시간을 넘기면 종료된다.

---

프로메테우스, 오픈소스 모니터링 시스템 <브라이언 브라질>
