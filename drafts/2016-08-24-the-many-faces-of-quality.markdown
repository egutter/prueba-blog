---
layout: post
title: "The Many Facets Of Quality"
author: Nicolas Perez Santoro
date: 2016-08-24 17:39:59 -0300
comments: true
categories: [development]
---

Some time ago, in a talk, someone asked "What is software quality, what gives quality to software?". My gut, lightning fast, half-joking answer was "Software shouldn't get in my way, that's quality!".
It was a half-truth, or a quarter-truth, because really, what is software quality is something with far too many sides.

We all want quality, but how can we tell what quality is?

<!-- more -->

Quality is something of a magic word, it means everything yet precisely because of that it means almost nothing.
We could define quality as "what is good" or maybe say that a particular piece of software has quality if it "tends towards what is good and avoids what is bad". But then we've shifted the mystery towards the already fuzzy concepts of good and bad.

[In my previous article](https://blog.10pines.com/2016/02/17/how-what-why-the-three-essences-of-software-development/), I outlined that software must be understood in terms of its "HOWs", "WHATs" and "WHYs", which loosely map to:

- HOW the computer system works below the hood: The inner details.
- WHAT is the interaction between the computer system and the human: The dialogue.
- WHY is the human driven towards the system: Human will, which shapes our systems and our relationships with them

And quality being what is _good_, it is what's _good_ for every human in relation in the system. Quality permeates every layer but ultimately springs from the _WHY_.

A few examples:

- We developers want the code to be harmonic, beautiful, well tested and properly structured. We call that quality. But why? Because this quality in the code means it's easier to reason about the system, it will be less likely to have bugs, which annoy the end user and cost time and money, it will be easier to add new features, it will be easier to add new developers to the project, etc. (And of course there is a certain _drive towards beauty_ that can possess developers, when the code quality becomes not a means but an end in itself, like the enjoyment of a fine wine or a symphony... even in that case, it rests on a _WHY_, a self-sufficient one)
- We want the user interface to look pretty, to be clean, to not be cluttered with superfluous information and to to have important information hidden from sight, etc. But why? It might be for purely aesthetic purposes, or you might want to improve the flow of the user and increase productivity or to avoid mistakes, or to improve adoption in your app, to free time for your user, etc.

So, to understand what quality is made of, how to create it, and appraise it, we're back to understanding the _WHATs_, _WHYs_ and _HOWs_... that is, the computer system, the human system, and their intersection.

# The Right Thing vs Worse is Better

Richard Gabriel in a famous (at least in some circles) essay titled [The Rise of the Worse Is Better](https://www.jwz.org/doc/worse-is-better.html) posed two contrary 'styles of design' characterized by the phrases 'The Right Thing' and 'Worse is Better'. Letting aside the actual contents of the essay, I think the phrases themselves capture a powerful idea, and I want to convince you that worse is, indeed, **better**.

Why? Because quality:

- is complex and multifaceted
- does not exist in a vacuum, it has its 'enemies': constraints.

That quality is complex and multifaceted shouldn't be a surprise, but it is easy to forget with the mindset of 'The Right Thing'. Quality is clear, expressive code, but it's also being secure, having good traces of execution, being scalable, having low latency, solving the needs of business, etc.

Quality is also to discover what is really needed: A process of "good quality" should be (theoretically) better at figuring out business requirements that needed to be tackled that in a lower quality process might be forgotten or neglected, while correctly figuring out which features and corner cases can be safely de-emphasized.

Quality also has constraints. 'The Right Thing' mindset can lead some people into thinking every feature is worth implementing, every refactor is worth making, every bug is worth fixing. But work requires time, money, and focus, and competes against other possible actions for resources.

When, instead of building software, we _select_ software, we risk falling prey to similar partial views of quality. For example, a language might have fancier features than another language, yet be beaten in terms of performance, or available libraries, or IDE integrations, etc.

We can now understand what 'Worse is Better' really means (if you read Richard Gabriel original article, that's not his interpretation, but it is _mine_ ;). It means that 'what seems worse from one perspective might be better from another, while The Right Thing might be... wrong'.

Now, certainly, one _can_ do **bad** tradeoffs "in the name of tradeoffs". Maybe worse was really worse after all. The key lesson is to not get caught in seducing 'obviously better' partial views with blind spots.

# Conclusion

So, I haven't really told you what quality **is**, right? I couldn't if I wanted to... at least not in a short blog post. Quality is a fractal, many facets that when looking closer reveal more aspects to be taken care of.

So, for example, we want code to be **maintainable**. When we look into what makes code 'maintainable', we find more 'facets of quality': has good expressive names, logic is encapsulated and isn't duplicated, state mutations are easy to track, objects and modules have uncluttered interfaces, code is easy to navigate with the aide of an IDE, entities are reified, we have good overview documentation of the system, non obvious segments of the code have in-place comments, logging allows you to trace activity and find the root cause of bugs, etc. And each one of these aspects, in turn, matter more or less to different people, accustomed to work with different flows, different tools, different mindsets. And in code bases where the code isn't perfect and people are often running against the clock and the budget, tradeoffs in quality are often made (good tradeoffs? bad tradeoffs?). For some people, "quality" will be finishing the project in budget.

In summary, we can achieve quality when we admit is not an easy thing to achieve and we probably don't know what it is or how to achieve it. To fully know what quality is and what makes it, we would have to be omniscient and know every little detail of every interaction (and what drives those interactions) each human (user, developer, ops, customer, etc) has or can have with the software. Quality must remain 'open' for us in order to truly get it. Only then we can have an honest conversation about what is good and bad for each one of the stakeholders, what are the constraints and the tradeoffs we're willing to make, and make plans for achieving it.
