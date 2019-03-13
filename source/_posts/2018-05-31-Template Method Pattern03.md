---
layout: post
title:  "Template Method Pattern03"
date:   2018-05-31
categories: Design Pattern
---

## Definition 

The Template Method Pattern defines the skeleton of an algorithm in a method, deferring some steps to subclasses. 

Template Method lets subclasses redefine certain steps of an algorithm without changing the algorithm’s structure. 

## Class Diagram

![](/image/templateh01.png)

## Code

![](/image/tmppattern01.png)

![](/image/tmppattern02.png)

![](/image/tmppattern03.png)

## Using the hook 

To use the hook, we override it in our subclass. 

Here, the hook controls whether the CaffeineBeverage evaluates a certain part of the algorithm; 

that is, whether it adds a condiment to the beverage. 

How do we know whether the customer wants the condiment? Just ask ! 

![](/image/tmppattern04.png)

![](/image/tmppattern05.png)

![](/image/tmppattern06.png)

## Reference

Head First Design Pattern