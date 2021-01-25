---
layout: post
title: "[OAuth 2.0 마스터] 5장_엑세스 토큰 얻기 : Implicit Grant"
date: 2021-01-26
categories: OAuth
---

## Sample

암시적 그랜프 플로우를 샘플 앱인 WMIIG 샘플 앱으로 확인해보자.

1. 사용자 : WMIIG 로 접속해서, 인포그래픽을 보려고 한다.
2. WMIIG : 당신의 프로파일과 당신이 작성한 글에 접속해야 하므로, 여기서 인가해달라고 요청한다. (페이스북으로 연결)
3. 페이스북 : WMIIG 이 사용자의 프로파일과 작성한 글에 접근하는 것을 허용할지 사용자에게 물어본다.
4. 페이스북 : 사용자가 허락했으면, WMIIG 에게 사용자의 프로파일과 글에 접근하는 데 사용할 수 있는 엑세스 토큰을 전달한다.
5. WMIIG : 페이스북으로부터 받은 엑세스 토큰을 이용해서 페이스북에게 사용자의 프로파일과 글을 요청한다.
6. 페이스북 : 전달된 토큰을 확인하여, WMIIG 에게 사용자의 프로파일과 글을 전달한다.

## 암시적 그랜트 플로우

WMIIG 은 사용자를 서비스 제공자의 인가 엔드 포인트로 연결함과 동시에,
리다이렉션 엔드 포인트와 scope 같은 정보를 서비스 제공자에게 전달한다.
사용자가 동의하면, 응답에 엑세스 토큰이 포함되어 전달된다.

### 인가 요청

사용자를 서비스 제공자의 인가 엔드 포인트로 연결한다.
OAuth 2.0 스팩은 다음과 같다.

```http request
GET /authorize?
  response_type=token&
  client_id=[CLIENT_ID]&
  redirect_uri=[REDIRECT_URI]&
  scope=[SCOPE]&
  state=[STATE] HTTP/1.1
HOST: server.example.com
```

1. response_type
   암시적 그랜트 플로우를 사용하고 있음을 나타내기 위해, token 으로 세팅한다.

2. client_id
   클라이언트 등록 과정에서 제공된 클라이언트 고유 아이디이다.
   
3. redirect_uri
   서비스 제공자는 요청이 성공하면 엑세스 토큰을 이 uri 로 전달한다.

4. scope
   클라이언트가 요청하는 접근 권한의 범위이다.
   각 범위는 공백 문자로 구분된다.

5. state
   클라이언트의 요청과 그에 따른 콜백 간의 상태를 유지하기 위해 사용된다. 
   클라이언트가 서비스 제공자에게 전달하면 서비스 제공자는 이 값을 다시 응답에 포함해서 전달한다. 

### 엑세스 토큰 응답

인가 요청 URL 에 질의를 보내면, 사용자의 동의 화면을 보게 되고, 거기서 인가 요청에 동의 하거나 거절할 수 있다.
인가 요청에 대한 유효한 응답은 다음과 같은 형태로 전달된다.

```http request
HTTP/1.1 302 Found
Locatoin: [REDIRECT_URI]#
  access_token=[ACCESS_TOKEN]&
  token_type=[TOKEN_TYPE]&
  expires_in=[EXPIRES_IN]&
  scope=[SCOPE]&
  state=[STATE]
```

1. token_type
   전달된 토큰의 유형이다. 
   대부분 bearer 토큰 유형이다.

2. expires_in
   엑세스 토큰의 유효기간이다.

3. state
   인가 요청에 state 파라미터가 포함되었다면, 응답에도 이 파라미터가 있어야한다.

---

OAuth 2.0 마스터 <찰스 비히스>