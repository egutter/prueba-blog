---
author: Hernan Wilkinson
categories:
- naming
- object-design
comments: true
date: 2012-01-12T00:00:00Z
title: About names when designing with objects
url: /2012/01/12/about-names-when-designing-with-objects/
---

One of the recurrent problems I see when teaching OO design is the “names” used by the programmers to name classes, methods, etc.

The problems are different depending if attendees are from the industry or the university. At the industry I see that names are wrongly selected, bias technology names, while at the university it is common to see the lack of ideas to name things (which makes sense because they have less experience). I will concentrate on the problem I see at the industry.

<!--more-->

Basically the problem can be summarized as using names towards the technology model instead of the business model. I'll show a few examples that will clarify the idea.

## The traffic light problem

I usually use a problem when teaching the introductory OO design class, where attendees have to model a traffic light ("semáforo" in Spanish). This problem has many interesting edges like what really is a "traffic light", is it just the one that control the traffic on a street or in a corner (or street intersection)?. But besides that and going to the problem of this post, it is interesting to see the names that are used to call the class that will model that concept. Usually those names are `LightContainer` or `TrafficController` instead of just `TrafficLight`. Clearly `TrafficLight` is a better name because it is the name used in the problem domain and therefore using it in our model will narrows the semantic gap, but why do attendees think on those names?

## The shopping store problem

Another example I use, in this case on the TDD course, is to model a on-line shopping store. That problem is really interesting because there are many many concepts to model that usually are ignored or not seen by most attendees, like shopping cart, the cashier or the just the sale. Shopping carts are generally discovered and modeled, but sometimes attendees use names like `ProductContainer` or similar.

The cashier is a interesting case. I guide the solution using a metaphor on real supermarkets so they can see that when it is time to pay, it is the cashier responsibility to do the sum, debit that amount on your credit card and so. But it is funny to see that even when I use the word “cashier” sometimes the class that models it as `SaleTransaction` (not realizing that the sale is the result of what the cashier does) or `CreditCardService` or `PaymentManager` and the like.

The sale is an object that usually nobody see and therefore it is not modeled. After the attendees realize that concept has to be modeled I go to the next issue, that is where to “store” the sales. Sadly, and as much as I repeat during the course to think about the problem domain, I usually get names for that object like `SalesRepository`, `SaleDAO` or the like instead of `SalesBook` (in Spanish `LibroDeVentas`, a well know accounting term).

## Understanding the problem

Why does this happen? Why the right names are not selected? I realized after given these courses that the pattern is the same: the selected names are thought from a “technology” point of view, as programmers; we do not use the words of the problem domain in our programs. I mean, I don't know any accountant that calls the sales book as “sale repository”, neither “sale dao”, and I never saw any supermarket employee call the shopping cart as “product container” and my wife never called a traffic light as “traffic controller”.... what is the problem?

The problem is that we continue programming (modeling) focusing on the technical side and not on the problem domain side. Of course, if we go to a supermarket we will not tell our companion “please, give me that product container” but “please, give me that shopping cart”. Why?, because we are not programming there, we “are” in the problem domain “for real”.

Ok, let's assume that that is the problem, why do I see this problem on the industry and not at the university? I think this is due to some kind of “infection” that people at the industry get from models, frameworks, finally examples that students at the university has not been exposed yet. As humans we learned (a big deal) from examples, and sadly most examples we have are really bad at giving names, sometimes because “hey! it is just an example” and sometimes because the ones that wrote the framework, model or example has became part of this vicious circle and therefore the names they use are like those we later choose, that is “nnnContainer”, “nnnRepository”, “nnnController”, etc.

How can we avoid this mistake? First of all realizing that we have to make a change in the way we think, we have to start thinking on the domain problem and not on “programming issues”. **Date** is called `Date` and not `NumberContainer`, and **String** is called `String` and not `CharacterCollection`, those are good examples of names thought from a problem domain point of view. After that, we can use a smell to see if we are choosing incorrect names. The smell is basically to see if the name has words like *container*, *repository*, *service*, *controller*, *manager* or any of the design pattern's name (again, **Directory** is called `Directory` and not `FileComposite` or the like). But please remember, this is a smell, not a rule!, sometimes we have to use names bias technology issues, and that is basically when modeling a technological problem domain.

So from now on, if a name you think for a class has those words, if it has that smell, try to think of a better one and if you can not come up with a good one (that for sure will happen) then choose a meaningless name but please, not a bad name. I will explain the difference between them in another post, this one has became larger that I expected :-)
