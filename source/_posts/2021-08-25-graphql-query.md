---
layout: post 
title:  "GraphQL Query"
date:   2021-08-25
categories: GraphQL
---

GraphQL Query 를 어떻게 작성하는지 정리한다.

## Query

Query 로 API 에 데이터를 요청할 수 있다.
Query 안에는 서버로부터 받고 싶은 데이터를 필드로 넣는다.

`https://snowtooth.moonhighway.com` 에 접속해서, 아래 Query 를 실행해보자.

```graphql
query {
  allLifts{
    name
    status
  }
}
```

응답되는 JSON 에는 아래와 같이,
allLifts 배열과 각 Lift 에 대한 name 과 status 가 있다.

![](/image/gql-simeple-example.png)

## query root type

`query` 는 GraphQL 의 타입 중 루트 타입이다. 
타입 하나는 하나의 작업을 수행하는데, query 는 쿼리 문서의 루트를 의미하기 때문이다.
query 에 사용할 수 있는 필드는 API 스키마에 정의한다. 
사용할 수 있는 필드는 아래와 같이 liftCount, allLifts, allTrains 등이 있다.

![](/image/gql-query-field.png)

## Selection Set

쿼리를 작성할 때는 필요한 필드를 중괄호로 감싸야한다.
이 중괄호로 묶인 블록을 Selection Set 라고 부른다.

liftCount, allLifts, allTrains 필드는 모두 Query 안에 정의된 필드이다.
selection set 는 서로 중첩시킬 수 있다.
allLifts 필드는 Lift 타입 리스트를 반환하므로, Lift 타입에 대한 새로운 selection set 를 만들기 위해 중괄호를 사용해야한다.

## Alias

응답 객체의 필드명을 다르게 받고 싶으면, 쿼리 안의 필드명에 별칭을 부여하면 된다.

```graphql
query {
  myAll: allLifts{
    name
    status
  }
}
```

결과는 아래와 같다.

![](/image/gql-query-alias.png)

## Filtering

쿼리 결과에 대한 필터링을 하고 싶으면, query arguments 를 전달하면 된다.

```graphql
query {
  allLifts(status:CLOSED){
    name
    status
  }
}
```

데이터를 특정하고 싶으면,

```graphql
query {
  Lift(id : "jazz-cat"){
    name
    status
    night
  }
}
```

## Filter Type

필드는 다음 두 타입 중에 하나이다.

1. 스칼라 타입
2. 객체 타입

### Scalar Type

스칼라 타입에는 다음 다섯 가지가 있다.

1. 정수(Int)
2. 실수(Float)
3. 문자열(String)
4. 불(Boolean)
5. 고유 식별자 (ID)

정수와 실수 타입은, JSON 숫자 타입 데이터를 응답하고
문자열과 ID 타입은, JSON 문자열 타입 데이터를 응답한다.
특히, ID 타입은 항상 유일한 문자열을 응답하도록 되어 있다.

### Object Type

객체 타입은 스키마에 정의한 필드를 그룹으로 묶어둔 것이다.

### Example

특정 리프트에서 접근할 수 있는 코스 목록을 조회해보자.

```graphql
query {
  Lift(id : "jazz-cat"){
    capacity
    trailAccess {
      name
      difficulty
    }
  }
}
```

selection set 의 capacity 는, 스칼라 타입 데이터이다.
selection set 의 trailAccess 는, 객체 타입 (Trail 타입) 이다.

## Fragment

Fragment 는 selection set 의 일종이다.
여러 번 재사용할 수 있다.

다음 쿼리를 보자.

```graphql
query {
  Lift(id : "jazz-cat"){
    name
    status
    night
    capacity
    trailAccess {
      name
      difficulty
    }
  }
  
  Trail(id: "river-run"){
    name
    difficulty
    accessedByLifts {
      name
      status
      capacity
      night
      elevationGain
    }
  }
}
```

Lift selection set 와 accessedByLifts 필드의 selection set 에는, 다음 필드들이 있다.

- name
- status
- night
- capacity


이런 경우에 아래와 같이,
프래그먼트로 쿼리에서 중복된 부분을 줄일 수 있다.

```graphql
fragment liftInfo on Lift{
    name
    status
    night
    capacity
    elevationGain
}

query {
  Lift(id : "jazz-cat"){
    ...liftInfo
    trailAccess {
      name
      difficulty
    }
  }
  
  Trail(id: "river-run"){
    name
    difficulty
    accessedByLifts {
      ...liftInfo
    }
    
  }
}
```

liftInfo 라는 이름을 붙인 프래그먼트는, Lift 타입에 대한 selection set 이다.


## Union Type

타입 여러개를 한 번에 리스트에 담아 반환하고 싶으면, union type 을 사용해라.

AgendaItem 이라는 유니언 타입이 있다. 
AgendaItem 은 Workout or StudyGroup 타입을 반환한다.
이 때, 프래그먼트를 사용하면 AgendaItem 이 Workout 일 때와 StudyGroup 일 때 특정 필드만 선택되도록 할수 있다.

```graphql
query schedule{
    agenda {
        ...on Workout{
            name
            reps
        }
        ...on StudyGroup{
            name
            subject
            students
        }
    }
}
```

여기서의 프래그먼트를 인라인 프래그먼트라고 한다.
인라인 프래그먼트는 프래그먼트와 다르게 이름이 없다.

물론 아래와 같이 이름이 붙은 프래그먼트로도 쿼리를 작성할 수 있다.

```graphql
query schedule{
    agenda {
      ...workout
      ...study
    }
}

fragment workout on Workout{
 	name
 	reps
}

fragment study on StudyGroup{
	name
	subject
	students
}
```

## Interface

인터페이스는 필드 하나로 객체 타입을 여러 개 반환할 때 사용한다.
추상적인 타입이며, 유사한 객체 타입을 만들 때 구현해야하는 필드 리스트를 모아둔 것이다.

agenda 필드가 ScheduleItem 인터페이스를 반환한다고 하자.
ScheduleItem 인터페이스에는 name, start, end 필드가 정의되어있다.
Workout 과 StudyGroup 이 이 인터페이스를 구현했다면, 두 타입 모두 name, start, end 필드를 가져야한다.

아래 query 를 보자.

```graphql
query schedule{
    agenda {
      name
      start
      end
      ...on Workout{
        reps
      }
    }
}
```

프래그먼트를 사용하면 특정 객체 타입이 반환될 때, 필드가 더 들어갈 수 있게 작성할 수 있다.

---
웹 앱 API 개발을 위한 GraphQL <이브 포셀로, 알렉스 뱅크스>
