---
author: Federico Zuppa
categories:
- agile
- scrum
comments: true
date: 2010-06-22T00:00:00Z
title: Agile is clean code
url: /2010/06/22/agile-is-clean-code/
---

But ... Isn't this a requirement for all approaches? Why do I have the impression that in Agile, the technical aspect is more important? When I was at University, I remember vividly having heard (and unfortunately later repeated) that the programmer had one of the less important tasks in the chain of producing software. The most important, more senior and smarter people are the architects/designers that analyse and design the application, giving the programmers some diagrams that they 'just' have to translate into code. That seemed like an easy task!!

<!--more-->

## What means Clean Code?

Ron Jeffries says that we should write "clean code that works". Uncle Bob then made an entire book of Clean Code. What is clean code? Following uncle Bob's explanation, at the high level, it is code that is well structured and which follows SOLID design principles. At a lower level, it's code that has meaningful names, good and useful comments, elegant functions and formatted code. It's code that expresses the intent of the developer in a clear way. It's code that contains no duplication. And is code that is easy to understand.

And how do you know it works? You don't know because you code it and you are a great programmer :-). You know it because there are automated tests that certify it. Agile development relies on test automation to know that features work and to know thay they do all the time. Beck says "Software features that can't be demonstrated by automated tests simply don't exist". Test automation is one of the pillars of clean code because it allows to continually refactor, it allows to redesign and ultimately it allows to deliver a continuous flow of value.

## Why is so important in Agile?

These are a few reasons to justify the 'pain' of writing better code:

1. ** Cost of Change is lowered:** Agile welcomes change, even late in the development process. However, to be able to change, it should be cheap to change. Code that is well designed, easy to read and thoroughfully tested is easy to change and can be changed without fear of introducing new bugs. On the contrary, code that is hard to understand, is not well modularized and has no automated tests is difficult to change and the risks of introducing bugs while modifying it are many.

2. **Iterative & Incremental Development means more time coding:** In sequential development, there are big phases of analysis and design before the coding phase starts. When the code phase starts, most of the design decisions should have been taken, so development proceeds fast. In Agile, development starts earlier so this phase takes more time. As the code base needs to be manipulated for a longer time, it pays to have it clean. Reading Clean Code, one of the a-ha moments that I had was in the chapter where Martin explains that in the life of a system, the majority of the time is spent reading code, not writing it. I started thinking about my experience, and that was entirely true. I have worked in many systems that already have a lot of features in it. In those systems, each time that I needed to add a new feature, I spent most of my time finding where to put the feature and how to accomodate it to the existing features than actually coding it.

3. **[Iterative & Incremental Development](http://agilebooknote.blogspot.com/2010/02/agile-is-iterative-and-incremental.html) means the code is changed frequently:** Design is performed incrementally in Agile. Every iteration, new features are included in the code base. The design of the system should be one that cares about all features included until that moment. Therefore, this means that a lot of times the existing design needs to be modified. If the code is not maintained clean, understanding and modifying it may become difficult in a short period of time.

4. **Working software is more important than Comprehensive Documentation:** This is what the [Agile Manifesto](http://www.agilemanifesto.org/) says. But having the code base in good state, with good comments, good names, well modularized and with a comprehensive set of automated tests is the best documentation that a developer may have. Been able to understand a piece of code easily because it's simple and it expresses its purpose with clarity is much better than going through UML design documents. Being able to run and debug a set of unit tests for certain functionality provides the best tool to understand that functionality. After all, what really happens is what is in the code, not what is in the documentation. Code is always up to date while documentation is hard to maintain.

## Why don't we do it then?

Lack of skill? Sometimes... This is a craft. A difficult one and we need to spend a lot of time getting better on it. Commitments.. Yes. Our own commitments and the commitments of our managers. Being under pressure and with the deadlines on sight, make it 'work' takes another meaning (I mean, you just want to throw some code that at least compiles and shows something on the screen). I won't follow. There are always plenty of reasons to do things worse!!

## Humm.... but Scrum doesn't include a topic on technical practices

I've seen great debates last year over the lack of technical practices in Scrum. Some people believe it is a shortcoming of Scrum while others say Scrum is a product development framework and therefore it is out of scope to include a technical section. Without entering into this debate, something that I've seen a number of times is software development teams starting 'just' with Scrum, leaving the introduction of some XP practices like TDD for the future (if you take a look the popularity of XP and Scrum, I guess many people observed the same!). This may be justified by an approach where changes are introduced gradually (after all, Scrum per se represents a major change at the organizational and process level).

But what happens if you start with 'just' Scrum?. Basically, I believe Scrum alone is not sustainable. At least, in software development it is not. As iterations go by and the code base grows, there is more and more work to finish each feature. By consequence, teams cannot keep their commitments, the business people get mad and you hit the [Scrum Wall](http://allankelly.blogspot.com/2009/07/scrum-wall-another-agile-failure-mode.html) (ouch, hit it many times)

This is still a topic I think about often. Scrum, per se, is not sustainable in software development. However, it is the most succesful process in the Agile world. It seems that leaving technical aspects out of it is both something that has help Scrum become more popular and the reason of many of its failures in the development world.

## Conclusion

Clean Code is one of the pillars of Agile. Being able to deliver clean code that works greatly enhances the chances of deliver value sustainably and it makes our life as programmers better. Clean code provides the best ROI and the best documentation. Of course it is difficult to do it and it certainly takes a lot more time than writing crappy code. However, by not doing it, we lose a lot of the advantages that Agile claims.