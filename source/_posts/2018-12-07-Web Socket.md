---
layout: post
title:  "Web Socket"
date:   2018-12-07
categories: Network
---

##### Web Socket이란?

WebSocket은 Web server 와 Web browser 간의 쌍방향 통신을 위한 통신 규약입니다.

##### Web Socket 전에는?

polling, long polling, stream이 있습니다.

##### Web Socket이 기존 통신 방법과 차이는?

WebSocket 프로토콜은 접속 확립에 HTTP를 사용하지만, 그 후의 통신은 WebSocket 독자의 프로토콜로 이루어집니다. 또한, header가 상당히 작아 overhead가 적은 특징이 있습니다. 장시간 접속을 전제로 하기 때문에, 접속한 상태라면 클라이언트나 서버로부터 데이터 송신이 가능합니다. 더불어 데이터의 송신과 수신에 각각 커넥션을 맺을 필요가 없어, 하나의 커넥션으로 데이터를 송수신 할 수 있습니다. 통신시에 지정되는 URL은 http://www.sample.com/과 같은 형식이 아니라 **ws://**www.sample.com/과 같은 형식이 됩니다.

##### reference

http://duckdevelope.tistory.com/19