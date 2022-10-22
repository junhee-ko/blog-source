---
layout: post
title: Formatting
date: 2022-10-21
categories: Clean Code
---

코드 형식을 맞추는 방법을 정리하자.

## 적절한 행 길이를 유지하자

큰 파일보다 작은 파일이 이해하기 쉽다.

## 신문 기사처럼 작성하자

이름은 간단하고 설명이 가능하게 짓는다. 이름만 보도 올바른 모듈인지 판단가능하도록.

소스 파일 첫 부분은 고차원 개념과 알고리즘을 설명하고, 아래로 내려갈수록 의도를 자세하게 묘사하자.
마지막에는 가장 저차원 함수와 세부 내용이 나온다.

## 개념은 빈 행으로 분리하자

패키지 선언부, import 문, 각 함수 사이에 빈 행이 들어간다.
빈 행은 새로운 개념이 시작한다는 단서다.

## 서로 밀접한 코드 행은 세로로 가까이 놓자

줄바꿈이 개념을 분리한다면, 세로 밀집도는 연관성을 의미한다.
연관성이란, 한 개념을 이해하기 위해 다른 개념이 중요한 정도다.
연관성 깊은 두 개념이 서로 떨어져 있으면, 코드를 읽는 사람이 소스 파일과 클래스를 여기저기 뒤지게 된다.

## 변수는 사용하는 위치에 최대한 가까이 선언하자

```java
private static void readPreferences() {
    InputStream is = null;
    try{
        is = new FileInputStream(getPreferencesFile());
        ...
    }
}
```

## 인스턴스 변수는 클래스 맨 처음에 선언하자

잘 알려진 위치에 인스턴스 변수를 모은다는 사실이 중요하다.

## 한 함수가 다른 함수를 호출하면, 두 함수를 세로로 가까이 배치하자

규칙을 일관적으로 적용하면, 독자는 방금 호출한 함수가 잠시 후에 정의되리라는 사실을 쉽게 예측한다.

## 친화도가 높을 수록 코드를 가까이 배치하자

위 처럼, 한 함수가 다른 함수를 호출해서 생기는 직접적인 종속성이 있을 수 있다.
이 외에도, 비슷한 동작을 수행하는 일군의 함수도 있을 수 있다.

```java
public class Assert{
    static void assertTrue(String message, boolean condition){
        ....
    }

    static void assertTrue(boolean condition){
        ....
    }

    static void assertFalse(String message, boolean condition){
        ....
    }

    static void assertFalse(boolean condition){
        ....
    }
}
```

## 가로 공백을 사용해 밀접한 개념과 느슨한 개념을 표현하자

할당 연산자를 강조하기 위해, 앞 뒤에 공백을 준다.
할당문은 왼쪽 요소와 오른쪽 요소가 분명히 나뉘기 때문이다.

반면, 함수 이름과 이어지는 괄호 사이에는 공백을 넣지 않았다.
함수와 인수는 서로 밀접하기 때문이다.

```java
private void measureLine(String line) {
    lineCount++;
    int lineSize = line.length();
    totalChars += lineSize;
}
```

## 들여쓰기

들여쓰기한 파일은 구조가 한눈에 들어온다.
변수, 생성자 함수, 접근자 함수, 메서드가 긍방 보인다.

---

클린코드 <로버트 C.마틴>
