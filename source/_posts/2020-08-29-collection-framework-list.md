---
layout: post
title:  "Collection Framework - List"
date:   2020-08-29
categories: Java
---

Java Collection Framework 의 List 를 정리한다.

## 1. Collection Framework

우선, Collection Framework 가 무엇일까.
공식 문서 (https://docs.oracle.com/javase/8/docs/technotes/guides/collections/overview.html) 
에 따르면 다음과 같이 정의되어 있다.

> 컬렉션 프레임 워크는 컬렉션을 표현하고 조작하기위한 통합 아키텍처로, 구현 세부 사항과 독립적으로 컬렉션을 조작 할 수 있습니다.

컬렉션 프레임워크를 사용하면, 내부 구현을 몰라도 컬렉션을 일관된 방식으로 조작할 수 있다는 것을 알 수 있다.
그렇다면, 컬렉션이란 무엇일까 ? 공식 문서에 따르면,

> 컬렉션은 객체 그룹 (예 : 클래식 Vector 클래스)을 나타내는 객체입니다.

조금 더 구체적으로 정리하면, 컬렉션은 자료구조 (데이터 저장) 와 알고리즘 (데이터 연산) 을 클래스로 구현해 놓은 것이다.

## 2. Architecture

컬랙션 프레임워크의 인터페이스 구조는 아래와 같다.
List, Set, Queue 인터페이스가 Collection 인터페이스를 상속하고 있다.

![](/image/collectoin-01.png)

각각의 인터페이스를 앞으로 정리할 예정이다.

## 2. List

![](/image/collectoin-02.png)

위와 같이, List 인터페이스를 구현하는 구체 클래스로는 ArrayList 와 LinkedList 가 이 있다. 이 둘의 공통점은,

1. 동일한 인스턴스의 중복 저장 허용
2. 인스턴스 저장 순서 유지

그렇다면 이 둘의 차이점은 무엇일까 ?
가장 큰 차이는, 내부적으로 인스턴스를 저장하는 방식에 차이가 있다.
각각의 특징을 파악하며 이해해보자.

## 3. ArrayList

ArrayList 는 내부적으로 배열을 이용해서 인스턴스의 참조 값을 저장한다.
ArrayList 의 add 연산을 보면, 아래와 같이 size+1 인덱스에 저장하는 것을 알 수 있다.

```java
public boolean add(E e) {
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    elementData[size++] = e;
    return true;
}
```

이러한 특징을 기반으로 장단점을 다음과 같이 정리할 수 있다.

1. 장점
   데이터의 참조가 용이해서 빠른 참조 가능 : 배열 기반이기 때문에 `인덱스 값을 알면 바로 접근이 가능`하기 때문.

2. 단점
   저장소 용량을 늘리는 과정이 복잡 : 배열은 한 번 생성되면 길이를 변경시킬 수 없어서, 용량을 늘리면 배열 인스턴스를 생성해야하고 `기존 데이터를 복사`해야하기 때문.
   데이터 삭제 연산 과정이 복잡 : 특정 위치의 데이터를 지울 때, 그 뒤에 저장된 데이터들을 `한 칸씩 앞으로 이동`해야하기 때문.

## 4. LinkedList

![](/image/collectoin-03.png)

LinkedList 는 위와 같이 노드 간에 서로 연결하는 방식으로 데이터를 저장한다.
LinkedList 의 add 연산을 보면, 마지막 노드에 새로운 노드를 연결하는 것을 알 수 있다.

```java
public boolean add(E e) {
    linkLast(e);
    return true;
}
```

```java
void linkLast(E e) {
    final Node<E> l = last;
    final Node<E> newNode = new Node<>(l, e, null);
    last = newNode;
    if (l == null)
        first = newNode;
    else
        l.next = newNode;
    size++;
    modCount++;
}
```

이러한 특징을 기반으로 장단점을 다음과 같이 정리할 수 있다.

1. 장점
   저장소 용량을 늘리는 과정이 간단
   데이터 삭제 연산 과정이 간단

2. 단점
   데이터 참조가 불편 : 연결되어 있는 노드들을 순회해야하기 때문

---

난 정말 JAVA 를 공부한적이 없다구요 <윤성우>