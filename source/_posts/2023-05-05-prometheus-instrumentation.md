---
layout: post
title: Prometheus > Instrumentation
date: 2023-05-05
categories: Infrastructure
---

## 매트릭 유형

프로메테우스의 매트릭 유형에는 다음이 있다.

1. Counter
2. Gauge
3. Summary
4. Histogram

### Counter

계측시에 가장 자주 사용되는 매트릭 유형이다. 사용 목적은,

1. 이벤트 개수나 크기를 추적
2. 특정 경로의 코드가 얼마나 자주 실행되는지 추적

예를 들어, 

```python
REQUESTS = Counter('hello_words_total', 'Hello Wolrds requested.')

class MyHandler(http.server.BaseHTTPRequestHandler):
    def do_GET(self):
        REQUESTS.inc()
        self.send_response(200)
```

메트릭은 0 에서 시작해서, 애플리케이션의 기본 URL 에 접속할 때마다 1씩 증가한다.
PromQL 표현식 rate(hello_words_total[1m]) 을 사용하면, 초당 얼마나 많은 HTTP 요청이 발생했는지 확인할 수 있다.

### Gauge

카운터의 관심이 "얼마나 빨리 증가하는가" 이면, 게이지는 실제 값에 관심을 둔다. 
값은 증가하거나 감소할 수 있다.

게이지의 예로,

1. 대기열 내의 항목 개수
2. 캐시의 메모리 사용량
3. 활성 스레드 개수
4. 레코드가 처리된 마지막 시간

예를 들어, 진행 중인 호출 개수를 추적하고 마지막 호출이 언제 완료되었는지를 확인하기 위해 게이지를 사용할 수 있다.

```python
INPROGRESS = Gauge('hello_words_inprogress', 'Number of Hello Wolrds in progress.')
LAST = Gauge('hello_words_last_time_seconds', 'The last time a Hello World was served.')

class MyHandler(http.server.BaseHTTPRequestHandler):
    def do_GET(self):
        INPROGRESS.inc()
        
        self.send_response(200)
        
        LAST.set(time.time())
        INPROGRESS.dec()
```

마지막 Hello World 가 언제 서비스되었는지 확인하기 위해, hello_words_last_time_seconds 가 사용될 수 있다.
또한, PromQL 표헌식 "time() - hello_words_last_time_seconds" 는 마지막 요청 후 얼마나 오랜 시간이 지났는지 확인 가능하다.

### Summary

시스템의 성능을 파악할 때, 애플리케이션이 요청에 응답하는데 걸리는 시간이나 대기 시간은 중요한 매트릭이다.
예를 들어, Hello World 핸들러 실행에 얼마나 걸리는지 추적할 수 있다.

```python
LATENCY = Summary('hello_words_latency_seconds', 'Time for a request Hello World.')

class MyHandler(http.server.BaseHTTPRequestHandler):
    def do_GET(self):
        start = timer.time()
        
        self.send_response(200)
        
        LATENCY.observe(time.time() - start)
```

/metrics 를 보면, hello_words_latency_seconds 매트릭은 두 개의 시계열을 참조한다.

1. hello_words_latency_seconds_count
2. hello_words_latency_seconds_sum

hello_words_latency_seconds_count 는 수행된 observe 호출의 개수이다. 
"rate(hello_world_latency_seconds_count[1m])" 는 Hello World 요청의 초당 비율을 반환한다.

hello_words_latency_seconds_sum 은 observe 로 전달된 값의 합이다. 
"rate(hello_world_latency_seconds_sum[1m])" 은, 1초당 요청에 대한 응답에 소요된 시간의 양을 반환한다.

이 두 표현식을 나누면, 마지막 1분에 대한 평균 대기시간을 얻을 수 있다.
"rate(hello_world_latency_seconds_sum[1m]) / rate(hello_world_latency_seconds_count[1m])"

예를 들어, 마지막 1분 동안 2초, 4초, 9초가 걸리는 세 개의 요청을 받았다고 하자.
카운트는 3, 합계는 15초, 평균 대시 시간은 5초가 된다.

## 계측 대상

계측 대상으로는 크게 계측하고자 하는 서비스나 라이브러리가 있을 수 있다.

### 서비스 계측

자체적인 매트릭을 가지는 서비스 타입에는 세 가지가 있다.

1. 온라인 서비스 시스템
2. 오프라인 서비스 시스템
3. 일괄처리 작업

#### 온라인 서비스 시스템

온라인 서비스 시스템은 사람이나 다른 서비스가 응답을 기다리는 시스템이다.
서비스 계측에 포함되는 주요 메트릭에는,

1. Request Rate (요청 비율)
2. Latency (대기시간)
3. Error Rate (오류 비율)

#### 오프라인 서비스 시스템

오프라인 서비스 시스템에는 응답을 기다리는 대기자가 없다.
보통 작업을 일괄처리하며, 여러 단계를 가지는 파이프라인이 있고 각 단계 사이에 대기열이 있을 수 있다.

로그 처리 시스템은 오프라인 서비스 시스템의 일종이다. 
각 단계별로 대기 중인 작업의 총량, 진행 중인 작업량, 항목의 처리 속도, 발생 오류에 대한 매트릭이 있다.
이러한 메트릭들은, 

1. Utilisation (활용도)
2. Saturation (포화도)
3. Error (오류)

#### 일괄 처리 작업 (Batch Job)

일괄처리 작업은 오프라인 서비스 시스템과 다르게 정기적으로 실행된다.
항상 실행되지는 않기 때문에 데이터 수집이 잘 동작하지 않을 수 있어서 "Pushgateway" 같은 기법들이 사용된다.
일괄처리 작업의 마지막에 실행하는데 걸린 시간, 작업의 각 단계에 걸리는 시간, 작업이 마지막으로 성공한 시간이 기록되어야한다.

## 매트릭 명명 규칙

1. 문자: 문자로 시작해야하며 그 뒤에 문자, 숫자, 밑줄 등 아무 형태나 붙여쓸 수 있다.
2. snake case: 이름의 각 부분은 소문자이고 _ 로 분리된다.
3. 접미어: 카운터/서머리/히스토그램 매트릭에는 _total, _count, _sum, _bucket 접미어가 사용되기 때문에, 이러한 접미어는 피해야한다.
4. 단위: 매트릭의 단위를 붙인다. ex) mymetric_seconds_total 은 초 단위를 갖는 카운터
5. 이름: 의미를 파악할 수 이름이어야하고, 왼쪽에서 오른쪽으로 갈 수록 구체적이어야한다.

---

프로메테우스, 오픈소스 모니터링 시스템 <브라이언 브라질>
