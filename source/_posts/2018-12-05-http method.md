---
layout: post
title:  "http method"
date:   2018-12-05
categories: Network
---

##### http method가 뭔가요?

http method 는 서버에게 무엇을 해야 하는지 말해줍니다.
GET, HEAD, POST, PUT, TRACE, OPTIONS, DELTE 가 있습니다.

1. GET

   서버에서 어떤 문서를 가져옵니다.

2. HEAD

   서버에서 어떤 문서에 대해 헤더만 가져옵니다.

3. POST

   서버가 처리해야할 데이터를 보냅니다.

4. PUT

   서버에 요청 메시지의 본문을 저장합니다.

5. TRACE

   메시지가 프락시를 거쳐 서버에 도달하는 과정을 추적합니다.

6. OPTIONS

   서버가 어떤 메서드를 수행할 수 있는지 확인합니다.

7. DELETE

   서버에서 문서를 제거합니다.

##### GET과 POST의 차이는 무엇인가요?

1. 데이터 전송 방식

   GET 방식은 요청하는 데이터가 HTTP Request Message의 Header 부분의 url 에 담겨서 전송됩니다. 

   즉, url 의 ? 뒤에 데이터가 붙어 request 를 보냅니다. 이러한 방식은 url 이라는 공간에 담겨가기 때문에 전송할 수 있는 데이터의 크기가 제한적입니다. 또 보안이 필요한 데이터에 대해서는 데이터가 그대로 url 에 노출되므로 GET방식은 적절하지 않습니다. (ex. password)

   POST 방식의 request 는 HTTP Message의 Body 부분에 데이터가 담겨서 전송됩니다. 때문에 바이너리 데이터를 요청하는 경우 POST 방식으로 보내야 하는 것처럼 데이터 크기가 GET 방식보다 크고 보안면에서 낫습니다. 

   (하지만 보안적인 측면에서는 암호화를 하지 않는 이상 고만고만합니다.)

2. 목적

   우선 GET 은 가져오는 것입니다. 서버에서 어떤 데이터를 가져와서 보여준다거나 하는 용도이지 서버의 값이나 상태 등을 변경하지 않습니다. SELECT 적인 성향을 갖고 있다고 볼 수 있는 것입니다. 

   반면에 POST 는 서버의 값이나 상태를 변경하기 위해서 또는 추가하기 위해서 사용됩니다.

3. Caching 유무

   부수적인 차이점을 좀 더 살펴보자면 GET 방식의 요청은 브라우저에서 Caching 할 수 있습니다. 때문에 POST 방식으로 요청해야 할 것을 보내는 데이터의 크기가 작고 보안적인 문제가 없다는 이유로 GET 방식으로 요청한다면 기존에 caching 되었던 데이터가 응답될 가능성이 존재한다. 때문에 목적에 맞는 기술을 사용해야 하는 것입니다.

##### reference

<https://github.com/JaeYeopHan/Interview_Question_for_Beginner/tree/master/Network>