---
layout: post
title:  "[빅데이터] 3장_빅데이터를 위한 데이터 모델: 사례"
date:   2020-03-08
categories: Big Data
---

Apache Thrift 란 직렬화 프레임워크를 사용해 SuperWebAnalytics.com 데이터 모델을 구현한다.

#### 3.1 어째서 직렬화 프레임워크인가

많은 개발자들이 원시 데이터를 기록하는 방법으로 JSON 등의 스키마 없는 형식을 고른다. 작업을 쉽게 착수할 수 있는 장점이 있지만, 데이터 오염이 언제든지 터질 수 있는 단점이 있다.

데이터 오염 문제는 그 문제가 어떻게 발생했는지에 대한 전후사정을 거의 손에 넣을 수 없기 때문에 디버깅이 어렵다. 예를 들어, 필수 항목이 누락되어 NPE 가 발생하면 문제의 원인이 누락된 항목이라는 것은 바로 알 수 있지만 애초에 그 데이터가 어떻게 들어왔는지에 대한 정보는 없다.

강제 가능 스키마를 만들었으면, 데이터를 기록하는 시점에 오류가 나므로 데이터가 무효화된 사정과 원인에 대한 전후 사정을 알 수 있다. 그리고, 이때 발생한 오류 덕에 프로그램은 무효 데이터를 기록하지 못하므로 마스터 데이터 집합도 오염되지 않는다.

직렬화 프레임워크를 사용하면 강제가능 스키마를 쉽게 적용 할 수 있다.

#### 3.2 Apache Thrift

Apache Thrift 는 정적 타입의 강제가능 스키마를 정의하는데 쓰이는 도구이다. 이와 유사한 도구로, Protocol Buffers 나 Avro 등이다. 

Apache Thrift 의 주요 요소는 구조체 (struct) 와 공용체 (union) 타입 정의이다. 이들은 다음 필드의 조합으로 구성된다.

1. 기본 데이터 타입 : 문자열, 정수, long 정수, double 실수
2. 다른 타입의 집합체 (리스트, 맵, 세트)
3. 다른 구조체와 공용체

##### 3.2.1 노드

SuperWebAnalytics.com 의 사용자 노드에서 개인은 사용자ID 나 브라우저 키기로 식별된다. 이 둘이 동시에 함께 식별에 쓰이지는 않는다. 이런 패턴은 노드를 나타날 때 흔히 볼 수 있는데, 공용체 데이터 타입과 일치한다. 하나의 값으로 여러 가지를 나타내는 타입이다. 

```
union PersonID {
	1: string cookie;
	2: i64 user_id;
}

union PageID {
	1: String url;
}
```

##### 3.2.2 간선

간선은 두 개의 노드를 포함하는 구조체로 표현될 수 있다. 간선 구조체의 이름은 그것이 표현하는 관계를 가리킨다. 간선 구조체 내의 필드는 그 관계로 엮인 개체들이다.

```
struct EquivEdge {
	1: required PersonID id1;
	2: required PersonID id2;
}

struct PageViewEdge {
	1: required PersonID person;
	2: required PageID page;
	3: required i64 nonce;
}
```

스리프트 구조체의 필드는 required 나 optional 로 표시한다. 필드가 required 정의되었는데 값이 없으면 직렬화나 역직렬화할 때 스리프트 자체에서 오류가 난다. 그래프 스키마에서 각 간선은 두개의 노드가 필수이므로 required 필드로 정의된다.

##### 3.2.3 속성

속성은 노드와 그 속성 대한 값을 가진다. 값은 여러 타입이 될 수 있으므로 공용체를 사용한다.

```
union PagePropertyValue {
	1: i32 page_views;
}

struct PageProperty {
	1: required PageID id;
	2: required PagePropertyValue property;
}

struct Location {
	1: optional string city;
	2: optional string state;
	3: optional string country;
}

enum GenderType {
	MALE = 1,
	FEMALE = 2
}

union PersonPropertyValue {
	1: string full_name;
	2: GenderType gender;
	3: Location location;
}

struct PersonProperty {
	1: required PersonID id;
	2: required PersonPropertyValue property;
}
```

##### 3.2.4 노드, 간선, 속성을 모두 엮어 데이터 객체로 만들기

정보에 접근하는 단일한 인터페이스를 제공하기 좋게 모든 데이터를 함께 저장하고자 한다. 그리고, 단일 데이터 집합에 저장되면 데이터 관리도 쉽다. 

```
union DataUnit {
	1: PersonProperty person_property;
	2: PageProperty page_property;
	3: EquivEdge equiv;
	4: PageViewEdge page_view;
}

struct Pedigree {
	1: required i32 true_as_of_secs;
}

struct Data {
	1: required Pedigree pedigree;
	2: required DataUnit dataunit;
}
```

Pedigree 구조체는 정보에 붙을 타임스탬프를 가지고 있다. 필요하면 디버깅 정보나 데이터 출처를 가질 수 있다. Data 구조체는 팩트 기반 모델의 팩트에 해당한다.

##### 3.2.5 스키마 발전시키기

스리프트는 시시때때로 스키마를 발전시키도록 설계되었다. 스키마를 변경할 때 기존 데이터와의 하위 호솬성을 유지하고자 하면 다음 규칙을 지켜야한다.

1. 필드의 이름은 변경해도 무방하다.

   직렬화된 객체 형식은 필드 식별을 위해 ID 를 사용한다.

2. 필드를 삭제할 수 있지만, 필드 ID 를 재사용하면 안된다.

   기존 데이터를 역직렬화할 때 스리프트는 스키마에 포함되지 않은 ID 를 갖는 필드는 모두 무시한다. 삭제된 필드의 ID 를 재사용하면 스리프트는 오래된 데이터를 새로운 필드로 역직렬화할 것이다. 그래서 유효하지 않거나 잘못된 데이터가 만들어질 수 있다.

3. 기존 구조체에는 optional 필드만 추가할 수 있다.

   기존 데이터는 그 필드가 없을 것이고 결국 역직렬화가 되지 않을 것이다. (공용체의 경우 required 나 optional 개념이 없다.)

#### 3.3 직렬화 프레임워크의 한계

직렬화 프레임워크는 required 필드가 모두 존재하고, 타입이 일치하는지만 점검한다. 즉, "나이는 음수가 아니어야한다.", "참인 시점의 타임스탬프는 미래가 아니어야한다." 와 같은 다채로운 속성은 점검할 수 없다.

> 스키마는 데이터를 받아 유효 여부를 반환하는 하나의 함수로 보는게 좋다. 

그런 이상적인 도구는 현실적으로 세상에 없지만, 한계를 우회할 수 있는 두 가지 방법이 있다.

1. 생성된 코드를 추가 코드로 감싸고, 추가 코드에서 나이는 음수가 아닌 조건을 체크하는 등의 추가적인 속성을 확인하도록 한다.

   여러 언어를 사용한다면 동일한 로직을 여러 언어로 작성하는 중복 작업이 요구된다.

2. 일괄 처리 작업 흐름의 시작점에서 추가 속성을 확인한다.

   이 확인 단계에서 데이터 집합을 유효한 데이터와 유효하지 않은 데이터로 나누고 유효한 데이터가 발견되면 알림을 보낸다. 그러면, 작업 흐름의 나머지 부분 구현이 쉬워진다. 유효성 확인을 통과한 데이터이면 엄격한 속성을 일단 가지고 있다고 가정할 수 있다. 그런데, 무효 데이터가 마스터 데이터 집합에 기록되는 것을 막지는 못하며, 데이터 오염이 발생한 전후 사정을 알아낼 수 없다.

---

빅데이터, 람다 아키텍처로 알아보는 실시간 빅데이터 구축의 핵심 원리와 기법 <네이선 마츠, 제임스 워렌>
