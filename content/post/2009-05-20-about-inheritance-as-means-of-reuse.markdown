---
author: Hernan Wilkinson
categories:
- object-design
- code-reuse
- smalltalk
- inheritance
comments: true
date: 2009-05-20T00:00:00Z
title: About inheritance as means of reuse
url: /2009/05/20/about-inheritance-as-means-of-reuse/
---

When talking about Smalltalk, there is definitively an over use on the possibility to add messages to `Object` class. It is so easy to do it, that people usually do it just to get something working fast, even if the coding is poor. There are a lot of messages (mainly `#isXXX` messages) that do not belong to `Object` and represent a bad design decision. Most of them are implemented there because they are "handy" and easily "reused". For example `#->` or `#assert:` implemented in Squeak. Definitively not all objects should respond to them.

<!--more-->

Many of these messages are only used from "inside" the object, like the `#assert:` message. I would never write something like this: `1 assert: xxxx`. Instead I would write: `self assert: xxx` witch clearly shows that `#assert:` is not a message that should be respond by every object, but only for those that represent assertions.

From my point of view, this issue is not an inheritance's problem per se, it is a misuse of inheritance. If I try to use a hammer as a screwdriver, it is not the hammer's fault, but mine.


## How is inheritance related with reuse?

Well, that is an important question. Inheritance from the "pure theory" point of view, should not be used as a means of reuse. Reuse comes from good models, not from inheritance. Inheritance should be used as a tool to organize the knowledge that, as a programmer, you are acquiring from the business domain.

Classes should be used to represent the concepts and ideas you see in the business domain and inheritance should be used to organize how this concepts are related in an ontological way. So for example, an abstract class should represent an abstract concept wich defines the essential behavior that all the objects instances of its concrete subclasses should respond. Due to this relationship reuse comes aside, but reuse is not the means of inheritance. Subclassing just to reuse the implementation of a superclass is not a good design decision; it will bring problems sooner or later.

There has been an attempt to minimize the methods implemented in the `Object` class. For example on the Squeak distribution the class `ProtoObject` has been created. `ProtoObject` has only 35 methods whereas `Object` has 436! (on the basic image of Squeak version 3.10.x). Although `ProtoObject` has fewer methods, I do not agree with some of them. For example `#ifNil:` (and the like) and `#tryNamedPrimitive:` (and the like). Clearly these two messages (and their mutations) are implemented in `ProtoObject` as a means of reuse and not because every object should understand them.

For example, why an account should respond `#tryNamedPrimitive:`? What does it mean to an account? A better design should have an object that represents the VM (for example) to which I can send the message `#tryNamedPrimitive:`. Of course the problem with this is how to access to this object and that is why that message is implemented in `ProtoObject`, because every object will inherit that method! And it will be so easy to use it as to write `self tryNamedPrimitive: xxx`. But thinking a little bit we can see that it is very easy to solve this problem. For example declaring this VM object in a global scope, so any object could send the message `#tryNamedPrimitive:` (and therefore reuse it), but not understand it.

## Composition vs Inheritance

This brings me to the "rule" that says composition is a better tool to reuse. The problem of using inheritance to reuse is that inheritance generates a strong coupling between the classes, its subclasses and its instances, where composition does not.

Inheritance generates an implementation and structural coupling between the classes and its subclasses (affecting directly their instances) where composition only couples an object with the composed one through the messages the former sends to the later. No implementation coupling, no structural coupling, just coupled by the message names one object send to the other, therefore a better design (the lower the coupling the better). This is the reason why good frameworks, black-box frameworks use composition over inheritance, where white-box frameworks being more immature, use inheritance to configure them.

This brings me to the idea of how hard should we stick to this, should we never "break" this rule? Well, I would not called it a rule but an heuristic, therefore it should be follow as much as possible, but we should also be pragmatic too. Sometimes subclassing to reuse while we are still learning about the problem is ok, is like making a white-box framework at the beginning. But we should never forget that our goal is a "black-box framework".

Other languages like Java do not suffer this problem and good designs can be implemented with it (although not so easily :-)). Java does not have this problem because `Object` class cannot be modified. This clearly shows that having too many methods in `Object` is not a problem of inheritance, but on how use it.

## To summarize

Inheritance should not be used to reuse, therefore having a lot of methods in `Object` class just because it is "handy" is a clear example of inheritance being used incorrectly. It generates unnecessary coupling which will make the system harder to change or refactor later.

Inheritance is just a tool; you can use it right or wrong.

There are other tools like composition and design techniques which solve the same problems and generate less coupling.

Reuse comes from good models, not from inheritance.