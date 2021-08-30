---
layout: post 
title:  "GraphQL 소개"
date:   2021-08-25
categories: GraphQL
---

GraphQL 이 무엇이고, REST 와의 차이는 무엇인지 정리한다.

## GraphQL 이란 

GraphQL 은,
1. API 를 만들 때 사용할 수 있는 쿼리 언어이자, 
2. 쿼리에 대한 데이터를 받을 수 있는 server-side 런타임이다.

GraphQL 은 선언형 데이터 패칭 언어라고 불린다.
무엇이 필요한지에 대한 요구사항만 작성하면 되고, 어떻게 가져올지는 신경쓰지 않아도 된다.

이에 대한 이해를 하기 위해, GraphQL 쿼리문과 응답 테스트를 여기서 해보자. 
쿼리문을 보내고 데이터를 받을 수 있다.
https://graphql.github.io/swapi-graphql/

다음과 같이 입력하고 실행해보자.

```graphql
query{
  person(personID: 5){
    name
    birthYear
    created
  }
}
```

아래에서, 
왼쪽이 쿼리문이고 오른쪽이 쿼리문에 대한 응답 데이터이다.

![](/image/gql-test-01.png)

응답 데이터의 형태가 쿼리문과 일치하며, 필요한 데이터만 들어있다.
쿼리문을 아래와 같이, 변경해보자.

```graphql
query{
  person(personID: 5){
    name
    birthYear
    created
    filmConnection{
      films {
        title
      }
    }
  }
}
```

결과는 아래와 같다.

![](/image/gql-test-02.png)



클라이언트 입장에서 GraphQL 을 사용하면,

1. 쿼리문을 중첨함으로써, 연관된 객체를 응답 데이터로 같이 받을 수 있다.
2. 복수의 객체 데이터를 받기 위해 요청을 여러번 반복할 필요가 없다.
3. 원치 않은 데이터가 포함되어 있으면 제외할 수 있다.

서버에서는,

1. 쿼리가 실행될 때마다, 타입 시스템을 기반으로 쿼리가 유효한지 검사한다.
2. 또한, GraphQL 스키마에 사용할 타입을 정의해야한다. 

위에서의 person 쿼리는 다음 Person 객체를 바탕으로 작성된 것이다.

```graphql
type Person {
    id: ID!
    name: String
    birthYear: String
    gender: String
    hairColor: String
    height: Int
    ...
}
```

## REST 와의 비교

### 오버패칭

REST 버전으로 테스트하기 위해, 여기에서 응답 JSON 을 확인해보자.

https://swapi.dev/api/people/1

실제 클라이언트가 필요한 데이터는 이름, 신장, 몸무게가 전부인데
불필요한 데이터까지 함께 응답받는다.

이것이, over-fetching 의 사례이다.
필요하지 않은 데이터를 많이 받는 것이다.

GraphQL 에서는,
필요한 데이터만 응답 받을 수 있다.

```graphql
query{
  person(personID: 1){
    name
    height
    mass
  }
}
```

결과는, 아래와 같이 필요한 데이터만 담겨 있다.

![](/image/gql-test-03.png)

### 언더패칭

아래의 응답에서 모든 영화 제목을 얻으려면 films 배열에 담긴 각각의 라우트에서 데이터를 가져와야한다.

https://swapi.dev/api/people/1

```json
    "films": [
        "https://swapi.dev/api/films/1/", 
        "https://swapi.dev/api/films/2/", 
        "https://swapi.dev/api/films/3/", 
        "https://swapi.dev/api/films/6/"
    ]
```

즉, 추가 데이터를 또 요청해야하는 상황이다.
이것을 under-fetching 되었다고 한다.

GraphQL 에서는,
쿼리를 중첩함으로써 페치 한번에 필요한 모든 데이터를 응답받을 수 있다.

```graphql
query{
  person(personID: 1){
    name
    height
    mass
    filmConnection {
      films {
        title
      }
    }
  }
}
```

아래와 같이 필요한 데이터를 모두 받아왔다.

![](/image/gql-test-04.png)

### 앤드포인트 관리

클라이언트의 변경 사항이 생기면, 보통 앤드포인트를 새로 만들어야한다.
이렇게 되면, 앤드포인트의 개수가 갈수록 늘어나게 된다.

GraphQL 을 사용하면, 앤드포인트가 보통 하나로 끝난다.
단일 앤드포인트가 게이트웨이 역할을 하게 되는 것이다.

---
웹 앱 API 개발을 위한 GraphQL <이브 포셀로, 알렉스 뱅크스>
