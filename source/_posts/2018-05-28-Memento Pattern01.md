---
layout: post
title:  "Memento Pattern01"
date:   2018-05-28
categories: Design Pattern
---

## Intent

The intent of this pattern is to capture the internal state of an object without violating encapsulation and thus providing a mean for restoring the object into initial state when needed.

## Class Diagram

![](/image/memento01.png)

- Memento
  - Stores internal state of the Originator object. The state can include any number of state variables.
  - The Memento must have two interfaces.
    - an interface to the caretaker. This interface must not allow any operations or any access to internal state stored by the memento and thus honors encapsulation. 
    - The other interface is to the originator and allows the originator to access any state variables necessary to for the originator to restore previous state.
- Originator
  - Creates a memento object capturing the originators internal state.
  - Use the memento object to restore its previous state.
- Caretaker
  - Responsible for keeping the memento.
  - The memento is opaque to the caretaker, and the caretaker must not operate on it.

## Example

![](/image/mementoPattern011.png)

## Code

![](/image/memento02.png)

![](/image/memento03.png)

![](/image/memento04.png)

![](/image/memento05.png)

![](/image/memento06.png)

## Reference

- <https://www.tutorialspoint.com/design_pattern/memento_pattern.htm>
- <https://www.oodesign.com/memento-pattern.html>