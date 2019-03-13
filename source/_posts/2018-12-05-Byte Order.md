---
layout: post
title:  "Byte Order"
date:   2018-12-05
categories: Network
---

##### Endian이 무엇이죠 ?

메모리와 같은 1차원 공간에 여러 개의 연속된 대상을 배열하는 Byte order 입니다. 빅 엔디안과 리틀 엔디안이 있습니다.

##### 빅 엔디안과 리틀 엔디안의 차이는 무엇이죠 ?

빅 엔디안은 최상위 바위트(most significant byte) 부터 차례대로 저장하는 방식이고, 리틀 엔디안은 최하위 바이트(least significant byte) 부터 차례대로 저장하는 방식입니다.

##### Network Byte Order은 뭔가요 ?

네트워크 통신시 표준 Byte Order은 빅 엔디안입니다. 리틀 엔디안을 사용하는 머신에서는 빅 엔디안으로 변화하여 전송해야 합니다.