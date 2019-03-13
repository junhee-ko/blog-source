---
layout: post
title:  "Singleton Pattern"
date:   2018-04-13
categories: Design Pattern
---

- Intent

  - Ensure that only one instance of a class is created.
  - Provide a global point of access to the object.

- Implementation

  ![](/image/si.png)


- EX

#### 01 : Early instantiation using implementation with static field

![](/image/SingleObject.png)

#### 02 : Lazy instantiation using double locking mechanism

![](/image/sgg.png)

#### Client

![](/image/SingletonPatternDemo.png)

- Reference
  - <http://www.oodesign.com/singleton-pattern.html>
  - https://www.tutorialspoint.com/design_pattern/singleton_pattern.html



