---
author: Roman Rizzi
categories:
- TDD
- Tests
- Design
comments: true
date: 2015-09-18T01:28:42Z
title: 'Tests: Paving our way'
url: /2015/09/18/tests-paving-our-way/
---

>Three Months have passed since I started working in 10 Pines as a participant of the apprenticeship program. Although we covered different topics and technologies, one of the things that has touched me most during my short journey as a programmer is that you can actually use your tests as the documentation of your system (you read it right, I'm not insane). This was a breakthrough for me so I decided to write this post to share some thoughts about it.

## Why do we write tests?

I'd like to start with a short reflection about tests and the meaning we give to them. It's true, testing gives us security and confidence. It lights our way, allowing us to perform changes without blowing the entire system into pieces. However, there is another (and equally important) reason to write tests: It's the place where we write down the things that we've just learned about the domain we're modelling.
Bearing this in mind, tests have become a compilation of my discoveries and that’s what they must reflect.

<!--more-->

## What is all this fuss about?

Imagine for a moment that you've just entered a new project; it's a great project with hundreds of thousands of lines of code. The developers of the project are super proud of their work because everything is covered with tests, and the system seems pretty solid. In order to work on the system, you have to understand the domain that is being represented, and you also need to understand what solution is provided through it. After someone explains you the basics of the system and you made some pairing with coworkers, you are ready to take the scene. But once you go right after the tests, you find out that they don't tell you much about the system. You are there, making your first steps, trying to solve this puzzle. How much time do you think you would have to spend to learn something about the system on your own?

Earlier in this post, we talked about the tests reflecting the knowledge we have about the domain we were representing, and one of the most important properties that our tests gave to us is **transmitting this knowledge and help others to learning about it.**

## Let’s strike while the iron is hot

When we are modeling a domain, we always try to represent things that exists in the real world, we tend to use analogies for this, giving our classes the same responsibilities that they have in the real world. We name methods with equally representative names, a name is a very powerful idea, it transmits an idea that is linked to the domain, an idea that we, people of the real world, can connect with ideas that lies in our minds. 
Same things happens when we write tests and the abstractions that we create inside them, they should have a representative name that help us understand which role they play and pay off our learning curve.

Although, it's very interesting how a simple space between this steps can make your tests more readable, and if that's not enough, it also mentally prepare the reader to understand what are you trying to do. In other words, it's a simple but powerful detail.
Having this in mind, let's take a look at this test that I made for the tennis scoreboard kata:

``` java

@Test
public void when_Score_Is_FortyToADV_And_Player1_Makes_A_Point_The_Score_Is_Back_To_FortyToForty() {
     scoreMultiplePoints( threePoints, playerOne());
     scoreMultiplePoints( threePoints, playerTwo());
     advPoint(playerOne());
     backToFortyFortyPoint(playerTwo());
     scoreForCurrentGameIs(40, 40);
     scoreForGames(0, 0);
}

```


If we apply the things that we mentioned early, we get something like this:


``` java

@Test
public void when_Score_Is_FortyToADV_And_Player1_Makes_A_Point_The_Score_Is_Back_To_FortyToForty() {
    scoreMultiplePoints( threePoints, playerOne());
    scoreMultiplePoints( threePoints, playerTwo());
    advPoint(playerOne());

    backToFortyFortyPoint(playerTwo());

    scoreForCurrentGameIs(40, 40);
    scoreForGames(0, 0);
}

```


With these examples, we can appreciate the benefits of using a space between steps and the readability we gain for it.


On the other hand, and in a more technical way, the same standards that we use when we code should be applied to tests, we should spend some effort keeping our tests clean following simple principles like DRY. Practices like isolating the creation of complex objects through Factory or Builder objects can reduce the noise that we are introducing to a simple test where we want to test a certain aspect of it. 

Personally, I think that this practices comes very cheap when we use techniques like TDD. I like to think that when we follow these principles, we're paving a road. 
A paved road is easier to pass through, not only for ourselves, also for the people who will have to walk it down after us. I know sometimes we are tied to a schedule or time-boxed iterations but the effort pays in the long run. To sum up, pave the road or tell the people who will come after to adjust their seat belts, **It's going to be a bumpy ride!**
