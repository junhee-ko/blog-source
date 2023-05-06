---
layout: post
title: Prometheus > Label
date: 2023-05-06
categories: Infrastructure
---

## 레이블의 정의

레이블은 시계열과 연관된 key-value 의 쌍으로, 시계열을 고유하게 식별한다.
경로별로 분류된 HTTP 요청에 관한 매트릭을 매트릭의 이름에 경로를 넣으면,

- http_requests_login_total
- http_requests_logout_total
- http_requests_adduser_total
- http_requests_comment_total
- ...

이것은 전체 요청을 계산하기 위해 모든 경로를 알고 있어야하는 안티패턴이다.
그래서, 프로메테우스는 레이블을 아래와 같이 사용할 수 있다.

- http_requests_total{path="/login"}
- http_requests_total{path="/logout"}
- http_requests_total{path="/adduser"}
- http_requests_total{path="/comment"}
- ...

이제 path 레이블을 포함하는 하나의 http_requests_total 매트릭을 사용해 작업할 수 있다.
PromQL 을 통해 다음과 같은 것들을 얻을 수 있다.

1. 전체 집계된 요청 비율
2. 전체 경로 중 단 한 경로의 비율
3. 전체 요청 대비 각 요청의 비율

## 계측 레이블과 대상 레이블

레이블에는 계측 레이블과 대상 레이블이 있다.

### 계측 레이블

계측 레이블은 애플리케이션이나 라이브러리 내부에서 알 수 있는 내용을 담고 있다. 예를 들어, 

- 애플리케이션이나 라이브러리가 수신하는 HTTP 요청 타입
- 통신 대상 데이터베이스

### 대상 레이블

프로메테우스가 데이터를 수집하는 특정 모니터링 대상을 식별한다. 예를 들어,

- 애플리케이션의 유형
- 데이터 센터 유형
- 개발 환경이나 운영 환경 여부
- 애플리케이션의 인스턴스

## 집계

다음 예를 보자.

```python
REQUESTS = Counter('hello_words_total', 'Hello Wolrds requested.', labelnames=['path', 'method'])

class MyHandler(http.server.BaseHTTPRequestHandler):
    def do_GET(self):
        REQUESTS.labels(self.path, self.command).inc()
        self.send_response(200)
```

"rate(hello_words_total[5m])" 의 결과는,

```text
{job="myJob", instance="localhost:1234",path="/foo", method="GET"} 1
{job="myJob", instance="localhost:1234",path="/foo", method="POST"} 2
{job="myJob", instance="localhost:1234",path="/bar", method="GET"} 4
{job="myJob", instance="localhost:5678",path="/foo", method="GET"} 8
{job="myJob", instance="localhost:5678",path="/foo", method="POST"} 16
{job="myJob", instance="localhost:5678",path="/bar", method="GET"} 32
```

"sum without(path)(rate(hello_words_total[5m]))" 의 결과는,

```text
{job="myJob", instance="localhost:1234", method="GET"} 5
{job="myJob", instance="localhost:1234", method="POST"} 2
{job="myJob", instance="localhost:5678", method="GET"} 40
{job="myJob", instance="localhost:5678", method="POST"} 16
```

"sum without(path, instance)(rate(hello_words_total[5m]))" 의 결과는,

```text
{job="myJob", method="GET"} 45
{job="myJob", method="POST"} 18
```

---

프로메테우스, 오픈소스 모니터링 시스템 <브라이언 브라질>
