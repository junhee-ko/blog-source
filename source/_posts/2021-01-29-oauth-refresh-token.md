---
layout: post
title: "[OAuth 2.0 마스터] 8장_엑세스 토큰 갱신하기"
date: 2021-01-29
categories: OAuth
---

액세스 토큰이 만료되면 어떻게 해야할까 ?

## Refresh Token Workflow

서비스 제공자가 Refresh Token Workflow 를 지원한다면, 
엑세스 토큰 요청이 대한 응답에 access_token 뿐만 아니라 refresh_token 값도 포함되어있다.
응답에 refresh_token 이 포함되어 있지 않으면 Refresh Token Workflow 를 지원하지 않는 것이다.

### 리프레시 요청

리프레시 토큰을 이용해서 엑세스 토큰을 요청하려면, 
서비스 제공자의 토큰 엔드 포인트로 POST 요청해야한다.

```http request
POST /token HTTP/1.1
HOST: server.example.com
Authorization: BASIC [ENCODED_CLIENT_CREDENTIALS]
Content-Type: application/x-www-form-urlencoded

grant_type=refresh_token&refresh_token=[REFRESH_TOKEN]
```

1. grant_type
   리프레시 토큰을 이용해서 새로운 액세스 토큰을 요청한다는 것을 나태내기 위해
   refresh_token 이어야한다.

2. scope
   원래의 엑세스 토큰보다 더 큰 접근 범위를 지정해서 요청할 수 없다.
   즉, 갱신된 엑세스 토큰은 이전과 동일하거나 작은 접근 권한 범위를 가져야한다.

### 엑세스 토큰 응답

엑세스 토큰 요청이 성공하면 다음과 같은 파라미터가 전달된다.

1. access_token
   얻고자 했던 엑세스 토큰이다.

2. token_type
   전달되는 토큰의 유형이다.
   대부분 bearer 토큰이다.

3. expired_in
   토큰의 유효기간이다.

4. refresh_token
   엑세스 토큰이 만료되면 엑세스 토큰을 갱신하기 위해 사용되는 토큰이다.

5. scope
   인가된 접근 범위이다.

## Refresh Token 이 없다면 ?

서비스 제공자가 리프레시 워크 플로우를 지원하지 않거나, 
리프레시 토큰이 만료되었으면 어떻게 해야할까 ?
유일한 방법은, 전체적인 인가 프로세스를 다시 시작하는 것이다.

---

OAuth 2.0 마스터 <찰스 비히스>