---
layout: post
title:  "Collection Framework - TreeSet"
date:   2020-09-01
categories: Java
---

Collection Framework 의 인터페이스 구조는 다음과 같다.

![](/image/collectoin-01.png)

여기서, Set 인터페이스를 구현하는 클래스로는 HashSet, TreeSet 이 있다.
TreeSet 클래스를 정리해보자.

## 1. Set 인터페이스

Set 인터페이스를 구현하는 클래스는 다음 특징을 가진다.

1. 데이터 저장 순서를 유지하지 않음
2. 데이터의 중복 저장을 허용하지 않음

## 2. TreeSet - 예시 01

TreeSet 은 Set 의 위 두 가지 특성을 모두 만족한다. 그리고, 다음 특성도 만족한다.
3. 데이터를 정렬된 상태로 유지
다음 코드로, 확인해보자.

```java
@Test
void tree_set() {
    TreeSet<Integer> treeSet = new TreeSet<>();
    treeSet.add(30);
    treeSet.add(20);
    treeSet.add(30);
    treeSet.add(10);
    treeSet.add(40);

    Iterator<Integer> iterator = treeSet.iterator();
    while (iterator.hasNext()){
        Integer currentInteger = iterator.next();
        System.out.println(currentInteger);
    }
}
```

출력 결과는 아래와 같이, 30 을 중복으로 저장하지 않고 정렬 상태를 유지한다.

```java
10
20
30
40
```

## 3. TreeSet - 예시 02

그런데, 다음과 같이 정의된 클래스의 인스턴스가 TreeSet 에 저장하는 대상이라면
정렬 기준은 뭘까 ?

```java
public class IamPerson {
    private String name;
    private Integer age;

    public IamPerson(String name, Integer age) {
        this.name = name;
        this.age = age;
    }
}
```

정렬 기준을 알 수 없다.
왜냐하면 name 으로 할 것인지 age 로 할 것인지, 정렬 기준을 정하지 않았기 때문이다. 

## 4. Comparable

그래서, 위의 IamPerson 클래스는 Comparable 인터페이스를 구현해서 정렬 기준을 명시해야한다.

```java
public class IamPerson implements Comparable<IamPerson> {
    private String name;
    private Integer age;

    public IamPerson(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public int compareTo(IamPerson person) {
        if (this.age > person.age) {
            return 1;
        } else if (this.age < person.age) {
            return -1;
        } else {
            return 0;
        }
    }
}
```

그리고, 아래와 같이 테스트 코드를 실행하면,

```java
@Test
void tree_set_comparable() {
    TreeSet<IamPerson> treeSet = new TreeSet<>();
    treeSet.add(new IamPerson("junee", 29));
    treeSet.add(new IamPerson("jko", 13));
    treeSet.add(new IamPerson("ko", 35));

    Iterator<IamPerson> iterator = treeSet.iterator();
    while (iterator.hasNext()) {
        IamPerson next = iterator.next();
        next.show();
    }
}
```

다음과 같이 나이순으로 오름차순 출력이 되는 것을 알 수 있다.

```java
jko 13
junee 29
ko 35
```

## 5. Comparator

정렬 기준을 제시하는 두 번째 방법으로,
Comparator 인터페이스를 구현한 구체 클래스를 TreeSet 의 생성자에 전달할 수 있다.
TreeSet 클래스에, Comparator 를 인자로 받을 수 있는 생성자가 다음과 같이 정의되어 있다.

![](/image/collectoin-tree-set.png)

이제, 아래 코드와 같이 Comparator 인터페이스를 구현해보자.

```java
public class IamComparator implements Comparator<String> {

    @Override
    public int compare(String o1, String o2) {
        if (o1.length() > o2.length()) {
            return 1;
        } else if (o1.length() < o2.length()) {
            return -1;
        } else {
            return 0;
        }
    }
}
```

그리고, 아래와 같이 테스트 코드를 실행해보면,

```java
@Test
void test_comparator() {
    IamComparator iamComparator = new IamComparator();
    TreeSet<String> treeSet = new TreeSet<>(iamComparator);
    treeSet.add("A");
    treeSet.add("AAAAAA");
    treeSet.add("AA");
    treeSet.add("AAAA");
    treeSet.add("AAAAAAAA");

    Iterator<String> iterator = treeSet.iterator();
    while (iterator.hasNext()){
        String currentString = iterator.next();
        System.out.println(currentString);
    }
}
```

문자열 순서대로 출력이 되는 것을 알 수 있다.

```java
A
AA
AAAA
AAAAAA
AAAAAAAA
```

---

난 정말 JAVA 를 공부한적이 없다구요 <윤성우>