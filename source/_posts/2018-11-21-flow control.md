---
layout: post
title:  "flow control"
date:   2018-11-21
categories: Network
---

##### flow control 이란 ?

flow control은 송신측과 수신측의 데이터 처리 속도 차이를 해결하기 위한 방법입니다. stop and wait 와 sliding window 방식이 있습니다.

##### stop and wait 란?

매번 전송한 패킷에 대해 확인응답을 받아야만 그 다음 패킷을 전송하는 방법입니다.

![](/image/flowcontrol01.png)

##### sliding window 란?

수신측에서 설정한 윈도우 크기만큼 송신측에서 확인 응답 없이 세그먼트를 전송하여 데이터 흐름을 동적으로 조절하는 제어 기법입니다.

1. 초기 구조를 다음과 같다고 가정합시다. 

   ![](/image/slidingWindow01.png)

2. 위 그림에서 데이터 0과 1을 전송했다고 가정하면,  슬라이딩 윈도우의 구조는 다음과 같이 변합니다. 

   ![](/image/slidingWindow02.png)

3. 만약 수신측으로부터 ACK라는 프레임을 받게 된다면 전송측은 0, 1이 데이터를 정상적으로 받았음을 알게 되고, 전송측은 ACK 프레임에 따른 프레임의 수만큼 오른쪽으로 확장됩니다.

   ![](/image/slidingWindow03.png)

##### sliding window 구현은 어떻게 ?

Go Back n ARQ 와 Selective Reject ARQ 방식이 있습니다.

1. Go Back n ARQ(Automatic Repeat Request)

   손상되거나 분실된 프레임 이후의 프레임을 모두 재전송하는 기법입니다. 데이터 폐기 방식을 사용하여 추가	적 버퍼가 필요 없습니다.

2. Selective Reject ARQ

   손상되거나 분실된 프레임만을 재전송하는 기법입니다. 데이터 폐기 방식을 사용하지 않으므로 순차적이지 않	은 프레임을 재배열하기 위해 필요합니다.

## Reference

<http://jwprogramming.tistory.com/36>

<https://goodgid.github.io/Error-Flow-Control/>