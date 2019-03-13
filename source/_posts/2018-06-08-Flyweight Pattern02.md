---
layout: post
title:  "Flyweight Pattern02"
date:   2018-06-08
categories: Design Pattern
---

## Implementation

We are going to create a *Shape* interface and concrete class *Circle* implementing the *Shape* interface.

 A factory class *ShapeFactory* is defined as a next step.

*ShapeFactory* has a *HashMap* of *Circle* having key as color of the *Circle* object. 

Whenever a request comes to create a circle of particular color to *ShapeFactory*, 

it checks the circle object in its *HashMap*, if object of *Circle*found, that object is returned otherwise a new object is created, stored in hashmap for future use, and returned to client.

*FlyWeightPatternDemo*, our demo class, will use *ShapeFactory* to get a *Shape*object. 

It will pass information (*red / green / blue/ black / white*) to *ShapeFactory* to get the circle of desired color it needs.

## Code

![](/image/flyweight011.png)

![](/image/flyweight022.png)

![](/image/flyweight033.png)

![](/image/flyweight044.png)

![](/image/flyweight055.png)

## Reference

<https://www.tutorialspoint.com/design_pattern/flyweight_pattern.htm>