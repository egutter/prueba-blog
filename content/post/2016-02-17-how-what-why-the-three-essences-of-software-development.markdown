---
author: Nicolas Perez Santoro
categories:
- development
comments: true
date: 2016-02-17T17:39:59Z
title: 'How, What, Why: The three essences of software development'
url: /2016/02/17/how-what-why-the-three-essences-of-software-development/
---

> "Facebook is as much sociology and psychology as it is technology" - _Mark Zuckerberg_

In my previous article, [Unitary vs Integral Understanding](http://blog.10pines.com/2014/07/31/two-ways-we-understand-code-unitary-vs-integral-understanding/),
I delved into how we understand a part of a system in terms of how it relates to the rest of the system. To recap, Unitary Understanding is understanding the
part by itself, abstracting the rest of the system, e.g. "i = i + 1" understood as a unit means nothing more than "increment i by one".
Integral Understanding would require us to see how that sentences interacts witht the rest of the code, so for example that sentence could "move a loop forward".

"How", "What", "Why" are three questions that are also helpful to see different facets of a system. A lot of what follows in this article might seem obvious to seasoned developers, but in my experience, these dimensions, because they're intuitive, are often not adecuately explained to junior developers.

<!--more-->

(warning: Abuse of the English language for rhetorical purposes might follow)

# How

To ask "how" the software works, it means to basically treat it as many subsystems
that collaborate between each other to produce a result. We don't care what the result is or why do we want that result.

Suppose we have a Cow Counting System. You have this web app:

{% img /images/2016-02-05-cow-counting-system.png %}

The buttons plus and minus alter the current amount of cows we have right now. To ask "how" this works would mean answering

- "Where it is deployed?"
- "Does it use POST roundtrips or AJAX requests?"
- "how does it keep state? mongo? sql? cookies?"

So "How" (and also "what" and "why") is a guiding question we use to both understand existing system and to build new systems.
And the starting point of the "how" is the result we want to achieve, so "how" is actually the last question we ask, since it's preceded by "what".


# What

"What" as in "What does this system do"? Going back to the Cow Counting System example, one might be tempted to answer "what" with:
"Well, the system has two buttons, and a cow count display, when the buttons are clicked,
a POST goes to the server, increases the record in the database...", but that's what I called the "how".

So let's remove the "how" and just focus on the "what" and try again:
"Well, the system has two buttons, and a cow count display, when the buttons are clicked, the cow count is increased or decreased."

But this description is lacking: _where is the human?_
What is not only "what the computer system does?" but "what the human does with the system and outside the system?"

## What does the user _do_?

In this case, suppose there is someone who wants to keep track of the cows in a corral. Cows go out sometimes to eat grass, so someone is there at the door
of the corral hitting the minus button everytime a cow goes out to eat grass, and the plus button everytime a cow goes in.
After the cows have finished eating grass, the user looks at the count, if it's lesser than the known amount of cows, the user will go out to look for the
missing cow.

This is a _story_ (this is why Agile focuses on User Stories as the basis of requirements!). Stories basically have actors interpreting and acting
through time with the state of things changing as a result of it.

Let's look deeper into what happens in this very simple story. The user ACTS on the computer and ACTS outside the computer, clicks on the buttons, and
goes looking for the unruly cow. And the user also INTERPRETS the computer and INTERPRETS the world outside the computer. All four things are linked.

1. (Interpreting the world) User looks at cow going in and out
2. (Acting on the computer) User clicks on plus and minus
3. (Interpreting the world) User notices enough time has passed, must now ensure all cows are in the corral.
4. (Interpreting the computer) Looking at the cow count display
5. (Acting on the world) Go look for any missing cows.

The external world is then "linked" to the computer system, allowing the user to interpret the world according to what they read on the screen.
The user by reading the count on the screen takes that as a symbol of the actual amount of cows in the corral.

This is not a feature of computer systems. Something fairly similar would happen if instead of a computer there were two people, someone counting
the cows in their head, and another asking this person how many cows are in the corral and hearing back the answer.
We humans interpret the world symbolically: we need to interpret the world and we need to go beyond our five senses, so words become facts about the world,
a smile becomes happines, a foul smell becomes a gas leak, etc. Symbols work by hinting into something that goes beyond the symbol, in this case,
the cow display is just a number on the screen, yet the user will go beyond the number on the screen and say "this means the internal state of the computer
equals the number I'm reading, and the internal state of the system equals the amount of cows in the corral".

So the interaction with the computer takes the form of a dialogue, human and computer talking
with each other and interpreting what each other says, both interpreting symbollicaly each other, the human reads a plus sign and goes "I guess that increments the counter"
and the computer when it receives a click it goes "the user wanted to increment the counter, store updated data and refresh display".

## The interpretation of the computer system is dependent on the contextual story

Each user will interpret what they read from the world and the computer, each symbol, according to their previous knowledge (i..e representation) of the world.
So like children, who have little previous knowledge about the world, won't see in certain foul smell a hint of a gas leak, or might not see the subtleties
in different kind of smiles, or might take words at face value without accounting for error and lies... users might sit in front of your lovely user interface
and not understand a damn thing of what you meant to convey.

Because every user has lived different lives, thought different things, and will interpret differently the symbols you're presenting to them.

The Cow Counting System I presented earlier is a good example of this: After I told you the story of how the system was used, the symbols took on new meaning,
e.g. plus means not only "adding one" but also "cow went back into the corral".

So, for example, doing a [user test](http://blog.10pines.com/2014/11/07/mvps-the-real-deal/) is immensely helpful to understand _what_ the system actually does when the user
is sitting in front of the screen. My (admittedly short) experience with it is that it quickly reveals that:

1. The people developing the system have a highly invested story with the system and understand everything and are generally not thinking of what a "blank state" user might
interpret unless they cultivate this skill.
2. Different users will create their own story with the system that won't match what you originally thought, for example someone might use the Cow Counting System to count
how many cows do they have, regardless of whether they're inside the corral or not.

The user interface needs to tell a good story, that's it, it needs to provide symbols along the ride, taking the context of each moment of the story at every point, to make
sure the user builds a story that's coherent. I say that the user is the one that builds the story because like all symbolic communication, the story is hinted at by the
teller, and it's the user's tasks to actually build it in their heads. It's a bit like the relationship between a recipe of a cake and the cake itself, and it's also why two
different movie watchers might watch a different "story" when watching the same film (one might say it sucked, another might say it rocked).

And you need to think your target users: a sleek interface might be mysterious to less proeficient users, a more explicit interface might look clunky to more proeficient ones.

## Bugs are what happen when the story breaks down

All software developers know this story: A user comes and reports a bug. The developers answer "it's not a bug, it's a feature". Resentment might grow if the dev forgets
what a bug is: A rupture in the story that the user is building. For example, if I click on a button that says 'delete' and I see a message that says "this user was deleted",
then I figure out that the user is still there somehow (it was a soft deletion), I as a user might not care or not know if keeping the user "half-deleted" was a feature because
it conflicted with my expectation. It might even conflict with my will ("I want the user really deleted! I don't care if they cannot login anymore!").

From the user point of view, that was a bug. Which brings us to the next question...

# Why

"Why" are to "whats" like "whats" are to "hows". To think about how to build a thing, first I need to know what I want to build, and to think what I want to build,
I need to know why. If "how" was the realm of computers, "what" was the realm of human-computer interaction, "why" is the realm of the physical world, ideas, and will.
Those three things are what actually gives form to software.

Back to our example: what drives "what" the Cow Counting System does? and why we built it that way, with a display of the current count of cows and two buttons?

- Physical reality: Cows and their movement
- Ideas that mirror this reality: The mathematical concept of an indifferentiated set (an integer), where substraction and addition mirrors the physical reality
of cows getting in and out.
- Symbolism: The symbols of a "plus" and a "minus" sign, the shape of the button, numbers, the image of the cow, and the expression "cow counting system" all draw from an
 ideal-symbolic system that's shared and enables communication.
- Human will: The user wants to control the cows. Why? Because a cow might get lost. Why do we care a cow might get lost? Because the owner of the cow profits from them,
and a lost cow means a loss of profit.

All these work like "invisible forces" that we intuitively graps and shape computer systems as much as pointers and loops.

## Know thy forces

Knowing these forces is _partly_ what allows to be creative and precise when designing _what_ our software will do. Most of you, when I first introduced the
Cow Counting System, probably didn't see any use for it, but when I introduced the story associated with it, could quickly see possible improvements: The system
should know the total amount of cows besides knowing how many there are currently within the corral, compare the two, and possibly show a green sign whenever
we have all cows in, and red whenever some cows are out. This proposal comes from knowing the forces at play outside the system, the will to control in one hand,
and human distractedness and forgetfulness, by avoiding the need of the user of remembering the total count of cows and making a mental comparison, we avoid a source
of mistakes that might lead to a loss of profit (symbolic reactions to the green and red colour is also a force we know applies here).

Where do forces lie? Ultimately they lie in the human being, in forms of ideas, behaviors, desires, interpretations, institutions. The outside world plays a role as well,
of course, but only insofar as it relates to use (remember the dictum 'man is the measure of all things'). Again, in this example, cows are not just a number, each of them is
unique, but as so far as this system is concerned, they're an undifferentiated set.

## A common mistake: minimizing the forces

An usual mistake I see is to believe that what one needs to learn from the outside world is the domain, where the domain is basically entities, their attributes, their
relations, how state changes through time, and the processes the user has interacting with these entities. So for this case, the domain would be cows, how many there are, which state they can be in (outside/inside the corral), and the act of counting and controlling them.

But you need to consider _all_ possible forces that can have any potential impact on the user. For example, an app you're trying to sell to end users is very different than, say,
a cash register app that all cashiers on a supermarket are going to use. The driving force in both cases is profit, but in one case, the user needs to be interested in using the app,
then it needs to use it, not shy away from it, and then buy it... everything that works towards this goal or against it matters. Psychological responses matter: Curiosity,
desire, frustration, pleasure, etc. So the UI must look pretty, be clear to avoid frustration and confusion, engagement has to be built...

Meanwhile, in the cashier app, efficiency is king. The UI doesn't need to be intuitive at first sight, nor pretty, it might even be complex, but the cashier has time to master it,
and they cannot whine about it and stop using it... because they would lose their jobs. The person we need to convince to buy the app is not the cashier (the end user),
but the supermarket owner, and very different goals mean different forces are at play.

## Some more examples of typical forces that affect most systems


- Security: ownership/privacy of data
- History of what has happened: Being able to see what happened for debugging purposes, being able to blame a particular employee for missing cash, mining data, etc.
- The Law: Legal requirements, users being able to commit fraud by using your system
- Human forgetfulness
- Human mistakes, data being out of sync with the outside world: e.g. consider a feature to import thousands of users into a system via a csv, and what happens when the columns of
 the first name and last name are switched and you have no quick way of undoing the operation... people can and will make mistakes.

## Know all sides and their interests, know the forces in opposition

We need to be aware as well that there are many actors in a system that relate differently in each case.
Take a social network for example. Many people use it for different things. Someone might use it to know new people, someone might use it to communicate with their families,
someone might use it to promote themselves, their projects, their business, their points of views and experiences... while some people might create fake ids and learn when
someone is on vacation to break into their houses. Advertisers, administrators, data miners, owners of the app, developers, devops, all have their sides and interests.

And forces can be in opposition, and often are. People's interests with an app might collide. A banal example: Traceability might take developers time, disk space, and might
slow down the app. So we desire traceability but we also need to consider the costs that come with traceability. A less banal example: you might want
to make registration simple and unencumbering for the users, but at the same time avoid fraud and false identities. Nobody likes bureaucracy, right? But we might do
want it when it helps us avoid undesirable situations.

# Conclusion... or something like it: Why should I care? how should I take this into acount?

One might ask: "why should I care about all this in the presence of division of labor?" (or alternatively, "knowing all those things is damn near impossible,
that's why we do agile iterative processes and we seek constant feedback"). It's true that sometimes different things are handled by different people, and with
division of labor, some people focus more on the "whys" and "whats" and others more on the "how".

But even if you do work only in the "how", it's helpful to be mindful of these things. A few examples:

- You'll notice that what might initially seem like an implementation detail, purely "how" stuff, actually ends up impacting in the end user. For example,
in the Cow Counting System, doing a whole POST of the whole html page vs doing an ajax request is not equivalent, since POST makes the screen refresh,
and it might be annoying for the user, and if many cows are going out at the same time, end up being problematic.
- You'll see that "how" you build the system and the "whys", the external forces and entities, if they're in harmony or not, matters a lot when new requirements come in.
E.g. Conceptually a user, how the user logs in into a system, and the email account we send notifications too, are different things that can be highly related,
so in many systems, you get a single db record for User with an email field. This works fine and it's very simple, until it turns out a single user might have many ways of
logging into the system, and the email for logging in might be different from the email you wanted email notifications to be sent to.
- Sometimes someone might say you "what" to implement, but when thinking of "how" to
  do it, you'll see that little details of the "what" are missing. Suppose a requirement that says "Delete a user from the system". What does it mean deleting for
  traceability?
  What does it mean for entities created by that user? Should those entities disappear, or not? Is there any chance the user might want to undo the deletion? If the user
  profile is still visible, should it have a message saying "this user has been deleted"? To be able to
  understand these things and its ramifications the developers's theory of the system, the inner mental representation,comes into play, the theory of not only how the
  system works, but what it does, and how it relates to people and what do those people want from the system.

In general, even in presence of division of labor, we should be mindful of exactly what you're delegating to whom and what is expected of each one of us, while at the same
time it's good to strive to add the most value possible besides of what you're supposed to specialize in. And it's also good of being mindful of the forces at play, and the
symbolic links we're reading and giving away.

# Further Reading

- [Design as Communication](http://www.jnd.org/dn.mss/design_as_communicat.html): An article by Don Norman, author of the book The Design of Everyday Things, that touches
on how the interfaces we build communicate and help the user build a story.
- [Programming as Theory Building](http://pages.cs.wisc.edu/~remzi/Naur.pdf): An old article (1985) by Peter Naur, early influential Computer Scientist, about how the
activity of programming relies heavily on the programmer building useful internal theory of the system.
- [Conceptual Debt is Worse than Technical Debt](https://medium.com/@nicolaerusan/conceptual-debt-is-worse-than-technical-debt-5b65a910fd46#.asspxn589) an article on the
concepts the system exposes to your users and how that impacts on the user experience.
- [Reflections on Trusting Trust](https://www.ece.cmu.edu/~ganger/712.fall02/papers/p761-thompson.pdf) A _very_ short, 3 pages, but _very_ important and _very_ recommended
reading about computer security by Ken Thompson. It relates to the concepts outlined here
because it talks precisely about the symbolic thought and links that the _developer_ makes when reading code, and the security pitfalls that come associated with it.
It also reveals why computer systems security is so hard.