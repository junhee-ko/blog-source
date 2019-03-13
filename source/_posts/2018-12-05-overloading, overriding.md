---
layout: post
title:  "overloading, overriding"
date:   2018-12-05
categories: Java
---

![](/image/overloading.png)

1. 오버 로딩

   같은 이름의 함수를 여러 개 정의하고, 파라미터의 타입과 개수를 다르게 하여 다양한 유형의 호출에 응답하게 합니다.

   ```java
   public class Overloading{
       void test(){}
       void test(int a, int b){}
       void test(double d){}
   }
   ```

2. 상속 관계에 있는 클래스 간에 같은 이름의 매서드를 정의하는 기술 입니다.

   ```java
   public class Employee{
       public void print(){
           System.out.println("aa");
       }
   }
   
   public class Manager extends Employee{
       public void print(){
           System.out.println("bb");
       }
   }
   ```