---
author: Ariel Umansky
categories:
- development
- organizations
- romantic
- opinion
- technical debt
comments: true
date: 2015-08-18T12:49:27Z
title: Organizational Debt and The Romantic Dev
url: /2015/08/18/organizational-debt-and-the-romantic-dev/
---

## Introduction

While thinking what I could write about, I remembered a distant sunny afternoon, back at the beginning of my professional experience. I left writings about what is now a possible future post and began to write these words...

<!--more-->

In my early days on one of the first companies that I used to work, a bug in production was detected and I was assigned to fix it. After long minutes diving into the code, I discovered that the problem was in a small method of 472 lines which contained 11 nested ifs. When we evaluate this with the architect, the answer I got was "easy .. Add an if here and there, and this should be fixed".

I remember going home extremely absorbed in my thoughts that day, while my body, with a strange autonomy, led my way home on its own.

On one hand, I’ve come from the university with an innocent idealism, thinking that one should always take the time to design the best abstractions to model the domain, trying to maximize as many design qualities as possible, favoring those that mattered most depending on the context. On the other hand, I felt totally incapable of providing a comprehensive solution to the problem, because I was overwhelmed by its magnitude, I hadn’t (and I think I still haven’t yet) the experience to improve it, and even suspected that the large amount of resources that would be necessary to refactor this will far exceed the benefits.

Eventually, I realized that it wasn’t convenient to the company to stop providing the service to redesign the product completely, but rather preferred to hire a lot of developers to maintain its product, which certainly was widely accepted in a portion of the market as it fulfilled certain needs like no other.

And then… then I saw it crystal clear. The architect was teaching me how to be efficient in that context: I just had to find the missing “if” as fast as possible because that business model worked fine that way.. And who was I to criticize that?

Still, if they had made the right design from the beginning, or made the refactor in time, perhaps they would have even higher margins, since they would had fewer bugs in production, need fewer programmers to maintain the system, and had a better product, or a product easier to improve. This is closely related to the concept of [**technical debt**](https://www.youtube.com/watch?v=pqeJFYwnkjE), coined by Ward Cunningham, and explained by Martin Fowler [here](http://martinfowler.com/bliki/TechnicalDebt.html).

## A team of seconds

Generalizing a little, this not only occurs in companies that offer a product. This also happens in systems builded by consultants, freelancers teams, academic groups, etc :.

It’s quite common to find systems that are so technically in debt that the only possible flavors of refactors are "small" or "impossible". 

But .. why does this happen?

According to my **short** experience, there are, at least, two kind of professionals:

* There are developers who are not even worried about that, either due to *ignorance* or  lack of *interest*

* There are others, who sometimes are tempted to make a provisional implementation (for any reason whatsoever, which may be **perfectly valid**) and then rely on their future self that will make the corresponding refactor. The problem is, that in many cases, either the refactor does not appear, is incomplete, or happens too late.

However, there is a big difference between the first and the second developer: regardless the reason, the *second* **_cares_**.

It seems to be a trivial difference, but in my personal opinion, it’s a key aspect that influences the growth prospects of the organization and the person itself:

* The *second* probably doesn’t always know the right way to do something, but he **does** have interest in doing it that way
* Ergo, the presence of doubt is much more common in the *second*, therefore, he will probably ask for guidance more often
* A good *second* is open to criticism. Even asks for it
* A *second* feels a *characteristic discomfort* when he is implementing something far away from the state of the art (whose precondition is the knowledge of such state of the art, or at least a notion of it)
* A *second* feels guilty when he adds the "daily if", or he makes sure to have a reason that explains that addition
* It’s more likely that a *second* takes the time to refactor
* It’s more probable that a *second* anticipates impossible deadlines, inconsistent, incomplete or contradictory requirements
* A team of good old *seconds* probably have right designs from the beginning 
* A team of good old *seconds* have higher chances to implement a software with better design qualities

Great! Then it seems you just need to have a **team of seconds**. 

But that’s not so common that .. why?

In my opinion, the concept of "second" as I am using in these paragraphs is not to classify only developers, but people. Organizations are nothing but interrelated people trying to achieve common objectives.

If within the pillars of the organization aren’t:

* the valorization of knowledge as the principal asset of the organization, and, therefore, of those who own it
* doing what was evaluated as correct as a must.. doing something else **is not** **consequenceless**, **it’s wrong**
* the naturalization of the dialog, the discussing and the argueing about what are the right ways to do something, whose mandatory roots are in the previous item
* and the emphasis and commitment to training: teaching instead of punishing

... then it’s very difficult to have that team of seconds, and it’s pretty likely that the few  existing seconds lose interest and **become firsts**, or **leave the organization**

## Organizational Debt

Isn’t there a parallel between software and organizations regarding quality? Could we say that there is something like an **organizational debt**, parallel to the technical debt, which would establish an inverse relationship between the mere passage of time and the ability of organizations to produce quality services or products, if such organizations do not pay the interest that would involve the development of certain internal improvement activities?

In other words, organizational debt will increase while certain values ​​are not met and no continuous-improvement-costs are assumed, like feedback processes, the development of a self-criticism conscience , the materialization of the mentioned feedback and auto-criticism, furthering communication and cooperation between the teams, among other activities, favoring the seizing of short-term opportunities instead, and the constantly compliance with deadlines, being unable to get free of the daily maelstrom circle vicious.

## The Romantic Dev 

Focussing on the individual again and in the developer as a professional, I propose the idea that in my opinion there is something beyond to being a "second".

Perhaps it’s the beginning, but what we really could aspire to become is that **romantic programmer** that even if the organizational context does not help, even if the tools are not the best, even in the absence of any expectation of improvement and even if the efforts are not recognized, he has the courage of going against the current and maintain a certain level of quality passing all those obstacles. 

*In other words, one who fights lost causes with the belief that they are the only ones that deserve to be fought.*

"A Man is what he does with what others did to him" said Sartre.

A certain degree of vocation, satisfaction with study, and perfectionism, are an essential part of an aspiring  romantic dev.

**However**, like all ideal concepts, it’s (and in this case, **must be**) inexistent in practice. The description of such professional is naive, and even inefficient. In this contexts, it’s much more difficult to do the right thing, and growth opportunities become **exotic**. *Note that to fight lost causes is noble and an artistically powerful idea, but it’s not an interesting feature for work environments.*

However, at least in my opinion, **these ideal concepts are useful to be identified and described, since they set the direction and sense of our evolution**. They can even help us to receive at least a hint of satisfaction in certain situations that were once all frustration.

And this is the way..

This is the way that someone realizes that is in the path of the romantic dev.

It’s spending long hours until he is in peace with his implementation. It’s evaluating all possible ways to comply with the requirements along with the tradeoffs, and choosing the one he thinks is better. It’s finding himself at home, late at nite, researching on the subject.

And this is the way.. This is the way that the romantic dev will go to his next meeting: with peace on his mind since he knows that he had suffered enough to be satisfied with his work. This is how he will sit, like anyone else, with predisposition to analyze and explain every single detail.

But his fate must be tragic. His turn in this meeting will be short: the implementation will icily please his client, since a working system is something to be expected, but difficult to appreciate.

And that is how that meeting will end, another sprint will start, and the cycle will repeat forever.

That will be his fate. His contribution will be forgotten and will occupy a little space in the universal superfluous hours. It’s his fate and the fate of all of us.

But his fate..

..his fate will be unfair.




*(1) related to Hash Oriented Programming, name by which, not without some humor, we refer to one of the many smells that are living in actual systems*