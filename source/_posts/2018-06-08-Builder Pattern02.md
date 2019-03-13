---
layout: post
title:  "Builder Pattern02"
date:   2018-06-08
categories: Design Pattern
---

## Implementation

We have considered a business case of fast-food restaurant where a typical meal could be a burger and a cold drink. 

Burger could be either a Veg Burger or Chicken Burger and will be packed by a wrapper. 

Cold drink could be either a coke or pepsi and will be packed in a bottle.

We are going to create an *Item* interface representing food items such as burgers and cold drinks and concrete classes implementing the *Item* interface and a *Packing* interface representing packaging of food items and concrete classes implementing the *Packing* interface as burger would be packed in wrapper and cold drink would be packed as bottle.

We then create a *Meal* class having *ArrayList* of *Item* and a *MealBuilder* to build different types of *Meal* objects by combining *Item*. *BuilderPatternDemo*, our demo class will use *MealBuilder* to build a *Meal*.

## Class Diagram

![](/image/builderPattern02.png)

## Code

![](/image/bp01.png)

![](/image/bp02.png)

![](/image/bp03.png)

![](/image/bp04.png)

![](/image/bp05.png)

![](/image/bp06.png)

![](/image/bp07.png)

![](/image/bp08.png)

![](/image/bp09.png)

![](/image/bp10.png)

![](/image/bp11.png)

![](/image/bp12.png)

![](/image/bp13.png)

![](/image/bp14.png)

## Reference

<https://www.tutorialspoint.com/design_pattern/builder_pattern.htm>