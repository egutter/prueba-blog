---
author: Federico Zuppa
categories:
- agile
- product management
- backlog
comments: true
date: 2015-04-09T09:24:15Z
title: The Backlog Guy
url: /2015/04/09/the-backlog-guy/
---

## Introduction

Yeah, this is how everyone called me in my latest gig. You want to hear the story? (I heard yes). It was a green field, ambitious project that was going to be performed differently than other projects in the same organization, using design thinking and Agile. People (mostly business people) was thrilled. Everyone working together, designing this new product. After a couple of weeks of design thinking, I said, innocently, that it was time to start building a backlog. Ohh nooo. Business people panic! This obviously was going back to the old, slow and painful way of building software that was used before. Interestingly enough (I thought) the backlog is that artifact that even myself alone use for any personal project. It’s light, useful and very powerful. In this post, I won’t focus on the psychological reasons of the freaking out. I will just focus on the backlog.

<!--more-->

## What is the Backlog?

In a few words, the backlog is the prioritized list of things you need to get done to build your product. I deliberately left out the word ‘requirement’. Are they requirements? Who knows. What we do know is we have a problem or an opportunity and we are going to build something to solve it. The backlog is that tool that allows us to define and manage what we need to build.

Let’s think how Scrum works:

1) First you have an idea

<img src="/images/ID-100311609.jpg" />

2) Then you start thinking, brainstorming and describing your idea

<img src="/images/ID-100248850.jpg" />

3) Then you need to break it in pieces that can fit an iteration and prioritize it. You can also try to estimate how big are the other items to start measuring your velocity and understanding the dimension of what you need to build.

<img src="/images/backlog.png" />

Voila! The result is a backlog. At least a backlog to start with. The backlog is a living artifact that will mutate throughout the project, as more is understood, described and defined in the product. It is important to notice that the backlog it’s a document that serves as an interface between the business and technical sides of the team. Business folks will create it and development folks will consult it (and provide feedback) all the time. It is also important to understand that the backlog will contain items with different levels of refinement and granularity. The items on the top, which will be worked on in the near future need to be better understood and defined. The items after them less. And the items that are at the far bottom are just ideas. Why spending time and money if we don’t even know if we are going to do them, right?

## A Note about User Stories

Everyone that knows a bit about Scrum knows that these ‘items’ (as I called them before) that make up the backlog are no other thing that User Stories. A User Story expresses, using business language, a functionality of a product. User Stories come from the XP world and in the essence of them was a shift from writing requirements to talking about them. Basically the idea was: here’s a deck of cards (the ones you can write one). Start writing in each of them what the system should do.

Everyone knows as well that there is a template for writing good user stories. The template goes something like this:

As a *type of user* I want *goal* so that *some reason*

There is a template for writing an acceptance criteria as well that goes something like:

Given *some context*   
When *an action is carried out*   
Then *a particular set of observables consequences*

Of course in my early days, I followed these templates strictly. Not anymore. I found many examples where it just didn’t make sense to express what is need to be done this way. I’ll just give one, which is a mobile application that I was working on last year where the development that was needed was to translate a mobile screen that was already designed into the application. There was no functionality, just showing data brought from a backend. I saw there were user stories written, but they were just an incomplete duplication from what was already described in the design. We all know the evils of duplication in software development, right?

So what is not negotiable for me:

1. A story should show something new to the final user. In other words, the user should see a difference from what is been built in this US. To users used to other methodologies, this is a big imposition (and there’s a lot of resistance). But think about the benefits. We’ll show actual stuff to the final users really fast and we will be able to use their feedback!
2. A story should fit an iteration: Scrum prescribes iterations (and iterations are good!). To make iterations we need to have small enough items to fit iterations.

And there isn’t much more. In the end, what is really important is that team knows how to write these items so that everyone understands the scope as precisely as possible and in the simplest, non duplicated fashion. Be smart to find this way in your particular project. Don’t follow recipes just because is recommended. Always think about the rationale and optimize according to your needs.

## Who writes User Stories?

The Scrum Guide says there is a Product Owner entitled with the responsibility of building and managing the backlog. It even says he is responsible for the ROI of the project. Do you think this makes sense? Unless the project is really small, do you think one person has the necessary expertise to be able to do this? The business could be complicated enough to need more than one person and added to this, there could be other areas (UX for example) working actively in the definition of the backlog. Again, think of what is important. The important thing when this role is shared is to agree on what needs to be built and to transmit this information coherently to the development team.

## A Note about the Tool

That manages the backlog of course. What does it need? Not much I believe. It needs to be very comfortable for the people that creates and manages the stories to write them and prioritize them. Prioritization needs to be performed dragging and dropping. It could have a taskboard which would be used if the team is not colocated (i.e. for co-located teams, the physical taskboard is much more efficient as it’s a constant information radiator). Of course, for Scrum it should support iterations (Trello is an amazing tool, but it doesn’t support them). For the graphs, I believe the Release Burndown is important and the Cumulative Workflow Diagram could be useful to measure the lead time and sense the Work in Progress.

## Conclusion

Being the backlog guy, of course I’ll advocate for it. The backlog is a great tool to manage the items necessary to build a product. The backlog should be light, there should be no duplicated information and it should be simple to create and maintain. The tool used to create the backlog should support this (in fact, what is important is that USs are written easily and prioritization can be performed in a snap, dragging and dropping).

Images courtesy of FreeDigitalPhotos.net