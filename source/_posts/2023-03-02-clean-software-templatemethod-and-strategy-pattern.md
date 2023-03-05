---
layout: post
title: Template Method && Strategy Pattern
date: 2023-03-02
categories: Agile
---

템플릿 메서드 패턴과 스트레티지 패턴을 정리한다.

## Template Method Pattern

버블 정렬을 예로 들어보쟈.

```java
public abstract class BubbleSorter {

    protected int doSort() {
        // ...
        if(outOfOrder(index)) swap(index);
        // ...
    }

    protected abstract void swap(int index);
    protected abstract boolean outOfOrder(int index);
}
```

BubbleSorter 로 다른 어떤 종류의 객체든 정렬할 수 있는 간단한 파생 클래스를 만들 수 있다.
ex) IntBubbleSorter, DoubleBubbleSorter ...

템플릿 메서드 패턴은 고전적인 재사용 형태의 하나이다.
일반적인 알고리즘은 기반 클래스에 있고, 다른 구체적인 내용에서 상속된다.
하지만, 상속은 아주 강한 관계여서 파생 클래스에서 필연적으로 기반 클래스에 묶이게 된다.

## Strategy Pattern

버블정렬에 적용해보자.

```java
public abstract class BubbleSorter {

    private SortHandle sortHandle = null;

    public BubbleSorter(SortHandle handle){
        sortHandle = handle;
    }

    public int sort(Object array){
        // ...
        if (sortHandle.outOfOrder(index)) sortHandle.swap(index)
        // ...
    }
}

public interface SortHandle {
    public void swap(int index);
    public boolean outOfOrder(int index);
    public int length();
    public void setArray(Object array);
}

public class IntSortHandle implements SortHandle {
    private int[] array = null;

    // ...
}
```

IntSortHandle 의 swap 과 outOfOrder 메서드는 버블 정렬 알고리즘에 직접 의존하고 있지 않다.
그래서, BubbleSorter 가 아니라 다른 Sorter 의 구현과도 사용할 수 있다.

---

클린 소프트웨어 <로버트 C.마틴>
