---
layout: post
title:  "Prototype Pattern01"
date:   2018-05-28
categories: Design Pattern
---

## Intent

- specifying the kind of objects to create using a prototypical instance
- creating new objects by copying this prototype

## Class Diagram

![](/image/proto11.png)

- Client
  - creates a new object by asking a prototype to clone itself.
- Prototype 
  - declares an interface for cloning itself.
- ConcretePrototype
  -  implements the operation for cloning itself.

## Example

![](/image/proto011.png)

## Code

Create an abstract class implementing *Clonable* interface.

![](/image/proto02.png)

Create concrete classes extending the above class. 

![](/image/proto03.png)

![](/image/proto04.png)

![](/image/proto05.png)

Create a class to get concrete classes from database and store them in a *Hashtable*.

![](/image/proto06.png)

PrototypePatternDemo uses ShapeCache class to get clones of shapes stored in a *Hashtable*.

![](/image/proto07.png)

## Reference

<https://www.tutorialspoint.com/design_pattern/prototype_pattern.htm>

<https://www.oodesign.com/prototype-pattern.html>
