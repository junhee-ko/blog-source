---
layout: post
title:  "Sync/Async, Blocking/Non-Blocking"
date:   2020-06-10
categories: Java
---

Sync 와 Async 의 차이, Blocking 과 Non-Blocking 의 차이를 정리한다.

## Sync 와 Async

메서드의 결과를 제공하는 Server 를 기준으로 생각해야한다.
Sync 는 다음 그림과 같이, 메서드의 결과과 결정되고 나서야 반환을 한다. 

![](/image/sync-async01.png)

Async 는 다음 그림과 같이, 메서드의 결과가 결정되기 전에 반환을 한다.

![](/image/sync-async02.png)

## Blocking 과 Non-Blocking

메서드를 호출하는 Client 를 기준으로 생각해야한다.
Blocking 은 다음 그림과 같이, 메서드를 호출하고 메서드의 결과를 return 받을 때 까지 다른 작업을 못한다.

![](/image/sync-async03.png)

Non-Blocking 은 메서드를 호출하고 다른 작업을 할 수 있다. 이는 call-back 을 통해 가능하다. 즉, 메서드의 결과를 제공하는 서버에서 결과가 완성되면 클라이언트에게 call-back 을 하는 것이다. 다음 그림과 같다.

![](/image/sync-async04.png)



