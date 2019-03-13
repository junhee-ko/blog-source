---
layout: post
title:  "Push Server"
date:   2018-12-07
categories: Network
---

##### Push Server 란?

클라이언트의 요청이 오면 응답해주는 방식이 아닌 서버가 클라이언트에게 공지사항과 같은 무엇인가 통지해주기 위한 방법입니다. 다시 말해 클라이언트의 요청이 없이도 서버는 클라이언트에게 응답하는 방식입니다.

HTTP 프로토콜의 특성때문에 실제 push server 구현은 되지 않습니다. 하지만 push server 의 효과와 비슷한 모델들이 등장합니다. 이것이 Polling, Long Polling, Stream 입니다.

##### Polling이란?

클라이언트가 끊임없이 웹서버에게 새로운 내용이 있는지 물어보는 방식입니다.

![](/image/pushserver01.png)

##### Long Polling이란?

클라이언트가 웹서버에게 새로운 내용이 있는지 물어보았을 때 웹서버에 새로운 내용이 없다면 대답해주지 않다가 새로운 내용이 생기면 이때 대답해주는 방식입니다.

![](/image/pushserver02.png)

##### Stream이란?

웹서버의 응답 헤더의  content-length 를 이용한 방법입니다. 응답헤더의 content-length 가 존재하지 않는다면 클라이언트는 연결이 끊길때 까지 응답을 받아들입니다. 즉 한번의 요청에 대해 끊임없는 응답을 보내는 방식입니다. 이 방식은 한번의 요청만 있으면 되므로 효율은 좋지만 error 처리에 매우 취약합니다.

![](/image/pushserver03.png)

##### reference

http://onecellboy.tistory.com/category/%ED%95%99%EC%8A%B5%EC%9E%90%EB%A3%8C%28~2017%29/%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC