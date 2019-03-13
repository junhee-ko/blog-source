---
layout: post
title:  "keep alive"
date:   2018-12-05
categories: Network
---

##### keep-alive 방식이란?

Keep Alive Time Out 이내에 Client 에서 Request를 재 요청하면 Socket을 새로 여는 것이아니라, 이미 열려 있는 Socket(port)에 전송하는 구조입니다.

##### http connectionless 방식과 keep-alive 방식의 차이는 ?

connection Less 방식은 매번 Socket(port)를 열어야 하기때문에 비용적인 측면에서 비효율적이며, keep-alive 방식은 port Conenction에 소요되는 시간을 절약할 수 있습니다.
