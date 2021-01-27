---
layout: post
title: "[OAuth 2.0 마스터] 7장_엑세스 토큰 이용하기"
date: 2021-01-28
categories: OAuth
---

암시적 그랜트 플로우나 인가 코드 그랜트 플로우를 이용해서,
서비스 제공자에게 엑세스 토큰을 요청할 수 있게 되었다.
이제, 엑세스 토큰을 이용해서 서비스 제공자가 제공하는 API 를 호출할 차례이다.

## 엑세스 토큰 이용해서 API 호출

API 를 호출할 때 엑세스 토큰을 전달하는 방법으로는 세 가지가 있다.

1. 인가 요청 Header Field 에 담아서 전달
2. 인코딩된 Form 의 Parameter 로 전달
3. URI Query Parameter 로 전달

### 인가 요청 Header Field 에 담아서 전달

OAuth 2.0 스팩에서 권장하는 방식이다.
HTTP 요청의 Authorization Header 에 엑세스 토큰을 담아서 서비스 제공자에게 전달하는 것이다.
엑세스 토큰을 요청하기 위해 사용된, basic authorization 방법과 유사하다.
차이점은, Basic 이 아니라 Bearer 가 사용된다는 것이다. 

```http request
GET /resource HTTP/1.1
HOST: server.example.com
Authorization: Bearer mF_9.B5f-4.1JqM
```

### 인코딩된 Form 의 Parameter 로 전달

Authorization Header Field 를 사용하지 않고 HTTP Header 의 다른 Field 를 사용하는 것이다.
GET 아닌 POST 요청을 한다는 점이 중요하다.

```http request
POST /resource HTTP/1.1
HOST: server.example.com
Content-Type: application/x-www-form-urlencoded

access_token=mF_9.B5f-4.1JqM
```

### URI Query Parameter 로 전달

URI Query Parameter 로 엑세스 토큰을 전달하는 방식이다.
애플리케이션을 테스트하고 디버깅하는데 수월한 방법이다.
하지만, 보안 결함이 발생할 수 있다.

참고로, 클라이언트가 이 방법을 사용할 때는
'no-store' 옵션을 포함하는 Cache-Control Header 도 함께 보내서
엑세스 토큰이 캐쉬 되지 않도록 방지해야 한다.

```http request
GET /resource?access_token=mF_9.B5f-4.1JqM HTTP/1.1
HOST: server.example.com
```
---

OAuth 2.0 마스터 <찰스 비히스>