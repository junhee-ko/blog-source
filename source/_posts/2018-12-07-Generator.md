---
layout: post
title:  "Generator"
date:   2018-12-07
categories: Node.js
---

##### geneator가 뭐죠?

함수 처리 중 임의의 장소에서 처리를 중단 및 재개할 수 있는 방법입니다.

![](/image/generator01.png) 

함수 선언시 function* 표시는이 함수가 제네레이터 함수로 선언된 것을 의미합니다. 실행하는 도중 중단 지점을 설정하는 방법이 yield 키워드입니다. 함수가 실행되는 도중 yield 함수를 만나면 이 제너레이터 함수를 호출한 caller 에서 다시 이벤트를 주기 전까지 대기합니다.

![](/image/generator02.png)

이제 이 quipsObj 객체는 제너레이터 객체가 되었습니다.

![](/image/generator03.png)

모두 실행되면 done 플래그값이 true로 바뀌는 것을 확인할 수 있습니다. 작업을 반복하는 사이에 어떤 액션이 취해져도, 상관이 없습니다. 이 제너레이터 객체는, 이번에 실행한 yield 에 멈춰서, 이벤트를 계속 기다리기 떄문입니다.

##### reference

https://joshua1988.github.io/web-development/javascript/promise-for-beginners/