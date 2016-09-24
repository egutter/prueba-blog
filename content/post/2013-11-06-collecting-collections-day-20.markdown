---
author: Dario Garcia
categories:
- collections
- research
comments: true
date: 2013-11-06T00:00:00Z
title: Collecting Collections - Day 20
url: /2013/11/06/collecting-collections-day-20/
---

Some time ago I started digging collection APIs in a few languages trying to answer a philosophical question. How were the whole/part relationships represented on programming languages (through collections) and what were the operations that we (as programmers) used on them.What started as a few minutes skimming through collections sources in Smalltalk, Java and (thanks to Ignacio) Ruby, have become an intensive and interesting work of exploration and discovery that branched more questions and a toy Java project also.

<!--more-->

After a month and a half, I don't have a clear answer to my question but as I dig deeper, some concepts that were unknown to me seem interesting enough to share. So I thought I could keep a log in this blog of what I'm doing and thinking, and at the same time share what I discover.Hopefully it might be helpful to someone.

## First steps

I started by looking at the different protocols for collections and different types of collections in each of three languages:

* Smalltalk: Because is THE reference for everything
* Java: Because is the language I know best
* Ruby: Because is very popular nowadays and I have who ask to in the company

After navigating source code, reading online documentation and writing down every common or interesting method I collected approximately 250 different methods. Some were shared among languages as `size`, some were exclusive as `identityIncludes: anObject` from Smalltalk. My objective was to discriminate methods by its semantics regardless its name or its class. The idea was to identify different operations. That's how `removeLast`, `pop → obj or nil` or `E removeLast();` were grouped as the same operation.

Online sources consulted for collection apis

- **Smalltalk**
  - [http://www.gnu.org/software/smalltalk/manual-base/](http://www.gnu.org/software/smalltalk/manual-base/)
  - [http://stephane.ducasse.free.fr/FreeBooks/BlueBook/](http://stephane.ducasse.free.fr/FreeBooks/BlueBook/)
  - [http://pdep.com.ar/material/apuntes/Guia_de_lenguajes-1-6.pdf?attredirects=0](http://pdep.com.ar/material/apuntes/Guia_de_lenguajes-1-6.pdf?attredirects=0)

- **Ruby**
  - [http://ruby-doc.org/core-2.0.0/Enumerable.html](http://ruby-doc.org/core-2.0.0/Enumerable.html)
  - [http://ruby-doc.org/core-2.0.0/Array.html](http://ruby-doc.org/core-2.0.0/Array.html)

- **Java**
  - [http://docs.oracle.com/javase/7/docs/api/java/util/Collection.html](http://docs.oracle.com/javase/7/docs/api/java/util/Collection.html)

## Too many operations and variants

250 something methods was not what I expected, 30 or 50 at maximum! I had to find a way to reduce that number. I didn't believe that some many concepts were needed when working with collections.

{% img /images/methods_table.jpg %}
(Screenshot of part of the method table)

I started analyzing each one of them and trying to find a way to describe one operation in terms of others. For example, `addAllFirst` could be described as doing for each element `addFirst`. If I could find the main operations, probably the rest were variations of them. I purposely ignored obvious performance considerations. I added attributes to categorize the methods (didn't work well but helped me understand them).

Interestingly, when trying to describe some methods in terms of others I found that I couldn't get a good description, but there were variations of the concept as `addFirst` and `addLast`. One could say that `addLast` is a version of `addFirst` in which you start reversed. But the same would be true simmetrically. Maybe, there was a missing concept there. Maybe I was trying to indicate the position but in a relative way. What is the difference between `addFirst` in Smalltalk and `add(0, ...)` in Java? Pragmatically, they do the same, but semantically they are using different coordinate systems. Why there isn't an `addSecond` method? Nobody add at the second position?

I couldn't answer those questions either, but surely some concepts started to emerge. I was sure that something like `add:obj at:ordinal` used as `add: 'text' at: firstPosition` was missing and some concepts were not reified, as "order", or "positon".If you don't have and order, which is the first one? Could you `addFirst` in a fixed size array?

## Common operations or shared operations?

As I checked different collection types, I realized that some operations were perfectly applicable on some types but there was no method for that (specially in Java). If I could find a common API between collections, was it possible for me to extend existing APIs to include those additional methods?

But wait, what if I needed to share a method definition between classes from different hierarchies? Single inheritance, Java and Java closed collections, started to sound as no-go. What about Smalltalk, did they have that problem? It turned out that as long as you followed standard operations on standard classes you didn't; and the guys that designed Smalltalk did a very good job at designing the collections hierarchy. However, you do have the problem when going out of the standard (that is rarer than in Java because of the extensive collection protocol). But if you need some standard collection methods on your custom classes and you don't inherit from collection, you might be in trouble.

There are approaches to attack this problem (sharing behavior in different hierarchies), that range from multiple inheritance, class algebra, traits, etc. But I wanted to think in a solution that were applicable on Java (the language I use most), so my options weren't many (any?). On the other hand I started to question if it had sense to define a method in seemingly different and unconnected classes, or maybe the operations on the collections were something of weight of their own, not something proper of the collection class. What if I wanted to create a new type of collection that could `addAllFirst`? Should I define that method again? Should I move it up in some strange shared hierarchy? What if `addAllFirst` were a concept applicable to files? If a file is an ordered collection of bytes should it have and `addAllFirst` somewhere in its hierachy? What if it didn't? Was a bad designed hierarchy? Could I design the perfect hierarchy for every possible collection? What was the possible cost of that?

Maybe I should abandon the idea of inheritance to gain the freedom of composition, but that carries the problem of identity. How can an external object reference to my state as it were his? Normally you would say that, you don't do that. You don't want another class messing with your state, but then you have to provide a good interface between both. The problem of the perfect hierarchy started to evolve as the problem of the perfect collaboration protocol for collections.

I don't have the answer to that either. But I'm on day 42 now, and messing with a toy project in java, trying some ideas that came after a little more research. I don't have a clear answer now but I feel I'm close to something (I hope it is not big wall of "can't do").

I will tell you more in another post so I don't bore you more and continue coding!
