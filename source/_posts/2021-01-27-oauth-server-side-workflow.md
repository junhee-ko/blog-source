---
layout: post
title: "[OAuth 2.0 마스터] 6장_Authorization Code Grant"
date: 2021-01-27
categories: OAuth
---

## Sample

인가 코드 그랜프 플로우를 샘플 앱인 WMIIG 샘플 앱으로 확인해보자.

1. 사용자 : WMIIG 로 접속해서, 인포그래픽을 보려고 한다.
2. WMIIG : 당신의 프로파일과 당신이 작성한 글에 접속해야 하므로, 여기서 인가해달라고 요청한다. (페이스북으로 연결)
3. 페이스북 : WMIIG 이 사용자의 프로파일과 작성한 글에 접근하는 것을 허용할지 사용자에게 물어본다.
4. 페이스북 : 사용자가 허락했으면, WMIIG 에게 사용자의 프로파일과 글에 접근하는 데 사용할 수 있는 `엑세스 토큰과 교환 가능한 인가 코드`를 전달한다.
5. WMIIG : 페이스북으로부터 받은 인가 코드를 이용해서 사용자의 페이스북 프로파일과 글에 접근할 수 있는 엑세스 토큰을 요청한다.
6. 페이스북 : 전달된 인가 코드를 확인하여, WMIIG 에게 엑세스 토큰을 전달한다.
7. WMIIG : 페이스북으로부터 받은 엑세스 토큰을 이용해서 페이스북에게 사용자의 프로파일과 글을 요청한다.   
8. 페이스북 : 전달된 엑세스 토큰을 확인한 후, 사용자의 프로파일과 글을 전달한다.

![](/image/server-side-workflow.png)

## 인가 코드 그랜트 플로우

1. 사용자 동의 화면에서 사용자가 동의를 하면 
2. 리다이렉션 엔드포인트로 인가 코드가 전달되고
3. 인가 코드를 엑세스 토큰과 교환한다.

### 인가 요청

사용자를 서비스 제공자의 인가 엔드 포인트로 연결한다.
OAuth 2.0 스팩은 다음과 같다.
response_type 이 code 이 경우만 제외하면, 암시적 그랜트 플로우와 동일하다.

```http request
GET /authorize?
  response_type=code&
  client_id=[CLIENT_ID]&
  redirect_uri=[REDIRECT_URI]&
  scope=[SCOPE]&
  state=[STATE] HTTP/1.1
HOST: server.example.com
```

1. response_type
   인가 코드 그랜트 플로우를 사용하고 있음을 나타내기 위해, code 로 세팅한다.

2. client_id
   클라이언트 등록 과정에서 제공된 클라이언트 고유 아이디이다.
   
3. redirect_uri
   서비스 제공자는 요청이 성공하면 인가 코드가 이 uri 로 전달한다.

4. scope
   클라이언트가 요청하는 접근 권한의 범위이다.
   각 범위는 공백 문자로 구분된다.

5. state
   클라이언트의 요청과 그에 따른 콜백 간의 상태를 유지하기 위해 사용된다. 
   클라이언트가 서비스 제공자에게 전달하면 서비스 제공자는 이 값을 다시 응답에 포함해서 전달한다. 

### 인가 응답

인가 요청 URL 에 질의를 보내면, 사용자의 동의 화면을 보게 되고, 거기서 인가 요청에 동의 하거나 거절할 수 있다.
인가 요청에 대한 유효한 응답은 다음과 같은 형태로 전달된다.

```http request
HTTP/1.1 302 Found
Locatoin: [REDIRECT_URI]#
  code=[AUTHORIZATION_CODE]&
  state=[STATE]
```

1. code
   엑세스 토큰과 교환하는 데 사용되는 인가 코드이다.

2. state
   인가 요청에 state 파라미터가 포함되었다면, 응답에도 이 파라미터가 있어야한다.

### 엑세스 토큰 요청

지금까지는, 인가 코드만 있고 아직 엑세스 토큰이 없다.
그래서 인가 코드를 엑세스 토큰과 교환하기 위해 요청을 한 번 더 해야한다.
서비스 제공자의 토큰 엔드 포인트로 POST 요청을 해야한다.

```http request
POST /token HTTP/1.1
HOST: server.example.com
Authorization: BASIC [ENCODED_CLIENT_CREDENTIALS]
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&
  code=[AUTHORIZATION_CODE]&
  redirect_uri=[REDIRECT_URI]&
  client_id=[CLIENT_ID]
```

1. grant_type
   엑세스 토큰으로 교환하고자 한다는 것을 나타내기 위해, 
   authorization_code 로 세팅되어 있어야한다.

2. code
   인가 요청에 의해 전달 받은 인가 코드 값

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
   
---

OAuth 2.0 마스터 <찰스 비히스>