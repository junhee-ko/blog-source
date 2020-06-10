---
layout: post
title:  "[빅데이터] 5장_일괄처리 계층의 데이터 저장소 : 사례"
date:   2020-03-31
categories: Big Data
---

HDFS  를 사용하는 방법과, 상위 수준 API 를 사용하여 필요한 작업을 자동화하는 방법을 정리한다. 항상 그랬듯이, 도구를 비교하는 것이 아니라 상위 수준의 개념을 보강하는 것이 목적이다.

#### 5.1 하둡 분산 파일 시스템 사용하기

HDFS 의 동작 방식을 정리하면,

1. 파일은 블록으로 쪼개져서 클러스터에  있는 여러 노드로 퍼뜨려진다.
2. 블록은 여러 노드로 복제되어서 장비가 다운되어도 데이터는 여전히 사용 가능하다.
3. 네임노드는 각 파일이 어떤 블록으로 구성되는지와, 그 블록이 어디에 저장되어 있는지를 추적한다.

파일과 폴더를 조작하는 HDFS API 사용법을 보자. 서버에 로그인한 정보를 모두 저장한다고 하자.

```
$ cat logins-2020-03-31.txt
jko	192.168.12.125 Thu Mar 31 22:33 - 22:46
jh	192.168.12.125 Thu Mar 31 21:15 - 22:42
ko	192.168.12.125 Thu Mar 31 23:31 - 22:13
jko	192.168.12.125 Thu Mar 31 22:33 - 22:43
```

이 데이터를 HDFS 에 저장하려면 데이터 집합 저장용 디렉토리를 우선 만들고 올리면 된다. 파일을 올리면 자동으로 블록으로 쪼개져서 여러 데이터 노드 사이에 분산된다.

```
$ haddop fs -mkdir /logins
$ haddop fs -put logins-2020-03-31.txt /logins
```

##### 5.1.1 작은 파일 문제

하둡은 데이터가 HDFS 상의 여러 작은 파일에 저장되어있을 때는, 계산 성능이 떨어지는 특성이 있다. 

그 원인은, 맵리듀스 작업이 입력 데이터 집합의 각 블록마다 테스크를 하나씩 실행하기 때문이다. 각 테스크는 실행을 계획하고 조정하는 오버헤드를 소모하는데, 각각의 작은 파일은 독립적인 테스크에서 실행되어야하므로, 그 비용은 반복적으로 발생한다.

##### 5.1.2 상위 수준 추상화를 향하여

솔루션이란, 

1. 확장성 있고
2. 내결함성을 지니며
3. 성능이 좋아야하며
4. 우아해야한다. ( = 관심 있는 계산을 간결하게 표현할 수 있는 부분이 있어야한다. )

지난 장에서, 마스터 데이터 집합을 조작할 때, 다음 두 중요한 연산을 살펴봤다.

1. 새로운 데이터를 데이터 집합에 추가하기
2. 데이터 집합에 수직 분할을 적용하고 기존의 분할이 깨지는 것을 방지하기

이제 여기에, HDFS 의 요구사항을 추가하자. **작은 파일을 효율적으로 큰 파일로 통합할 수 있어야 한다**는 것이다.

앞 장에서 봤듯이, 파일과 폴더를 직접 다루어서 작업을 하기에는 불편하고 오류가 발생하기 쉽다. 우아한 방식의 Pail 이란 라이브러리 살펴보자. Pail 을 사용하면, 코드 한 줄로 폴더를 추가하거나 작은 파일들을 통합 할 수 있다.

> [잠깐 뒤돌아 보기] 큰 관점으로 살펴보자. 
>
> 마스터 데이터 집합은 람다 아키텍쳐에서 모든 정보의 원천이다. 
>
> 일괄 처리 계층은 거대하고 꾸준히 증가하는 데이터 집합을 문제없이 처리해야한다.
>
> 실제 질의에 응답하기 위해서, 데이터를 일괄 처리 뷰로 변환하는 쉽고 효율적인 수단이 있어야한다.

#### 5.2 페일을 사용하여 일괄처리 계층에 데이터를 저장하기

페일은 dfs-datasource 라이브러리에 포함된 것으로, 파일과 폴더를 얇게 추상화한것이다. 이 추상화를 통해 일괄처리에 사용할 레코드 집합을 관리하기가 쉬워진다. 페일의 목적은, 우리가 신경써야 할 연산 ( 데이터 집합에 새로운 데이터 추가하기, 수직 분할, 파일 통합 ) 을 안전하고 쉽고 성능 기준에 맞게 수행할 수 있도록 해준다.

내부적으로 페일은, 표준 하둡 API 를 사용하는 자바 라이브러리일 뿐이다. 하위 수준 파일 시스템 상호 작용을 처리하며 하둡 내부 구조의 복잡성을 감추는 API 를 제공한다.

페일을 살펴볼 때는, 어떻게 HDFS 의 장점을 보존하면서 데이터에 대한 연산을 간소화하는지를 기억해두어야한다.

---

빅데이터, 람다 아키텍처로 알아보는 실시간 빅데이터 구축의 핵심 원리와 기법 <네이선 마츠, 제임스 워렌>
