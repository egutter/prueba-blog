---
author: Hernan Wilkinson
categories:
- java
comments: true
date: 2014-06-05T00:00:00Z
title: Is Java Dead?
url: /2014/06/05/is-java-dead/
---

The answer is: of course not!!, but now that I got your attention surely due to
the title that mimics the lately hot [_"Is TDD Dead?"_][is-tdd-dead] discussion
started by [@dhh][dhh], I would like to concentrate on the real objective of
this post that I think it is really more important: is Java taking the right
road? or in other words, are the last changes in Java 8 _"good"_ ones or not?

<!--more-->

So, let me start saying why I think Java is not dead, to stop any language fight
from the beginning even though the reason is a no brainer. It is not dead
because despite how old Java is becoming and how it is starting to show its
wrinkles through the lack of type inference, numbers not being objects, etc.,
it is still widely used, it has a big and productive community and I think that
almost for sure, its community is the one that more open source projects has
produced. A language it is not only used by its technicals features but also for
the community it has, the projects it supports, etc.

Having said that and made that clear, let's now concentrate only on the
technical part. So, are the changes of Java 8 _"good"_? Are they useful? How can
we measure if they are _"good or bad"_? The plain answer is, it depends, it
always depends :-). But, it depends on what?. A few months ago after a talk I
gave at the [Scrum Gathering Bolivia][rgsbol-talk], an attendee came to talk to
me because I made some comments about Python's issues. He told me: "I'm really
happy with Python! I think I'm more productive that I used to be with the
language I used before". I asked him which language was that, and he said:
"Java". So yes, it depends. If you are used to program with Java, suffering due
to its archaic type system, the lack of closures and its verbosity, Python looks
like the holy grail, but is it?

Programming languages reflect the knowledge, beliefs and ways to solve problems
the authors of those languages have. But not only that, they carry on their
shoulders the heavy weight of its history and that is, as I understand, what it
is killing Java, or at least what it is not making Java get any better when you
compare it with other languages.

Of course that for Java programmers, version 8 has many interesting features,
but from the point of view of other programming languages, how _"good"_ are
they? Are these changes making Java a "better" language or just keeping it from
its (in)evitable dead end? I propose you to analyze some of the Java 8 changes
from both perspectives and see where we get.

## New Date and Time model (so called API)

It is about time to throw away the ugly, full of design errors, best example of
all the things you don't have to do, **`Calendar`** class. That class contains
almost all the design flaws you can look for, like a misleading name because its
instances do not represent calendars but dates, or dates with time, or... wait a
moment, another problem, it does not represent just one thing but many! it does
not follow the _"Single Responsibility Principle"_ and not only that, it allows
you to represent invalid dates as February 31st of 2014, or change dates when
dates should be immutables like the numbers and many other objects. You have no
idea the amount of people I interviewed with the misconception of what a
**`Date`** is and what a **`Calendar`** is due to this really bad abstraction
(all Java programmers).

So, the new date/time model is _"good"_, it is an advance towards less
misunderstanding, it provides new nouns that makes the language grow and allows
us to talk and think about days of month, like `December 25th` (with the
abstraction `MonthDay`) or months of year as `April 2014` (with `YearMonth`).
Finally, with the new time model not everything is a `Calendar`!. 

We can also express "measures of time" with a `Duration` and even `Years`! Yes!
they broke the fear of "optimization" that makes most programmers represent
years with integers! We all know that a number is not a year, we all know that a
number does not respond to `isLeap` but it makes a lot of sense for a `Year` to
answer that, so I toast for this new time model that prioritizes good
abstractions over performance/space.

Yes, this model is "good", but is it? Let's see how it compares to a date/time
model of other languages. In this case the one implemented in [Pharo][pharo] (an
open source implementation of Smalltalk) called [Chalten][chalten]. (You can
read about it in this [paper][paper] published in 2005). With Chalten you can
not only represent those elements that you can with Java 8 but also many more,
starting with real measures of time because it uses an algebraic model that
allows you to represent any type of measure ([Aconcagua][aconcagua]). With
Chalten you can represent _1 day_, _3 months_, _5 year_, _7 centuries_ and any
other kind of time measure like _2 semesters_ (if you create the unit
**semester**). There is not only a `Duration` that it is limited to the time
units it provides through messages like `toHours`, `toMinutes`, `toNanos`, etc.
but the possibility to create any time unit you need or want.  

Chalten is based on an analogy that sees _Time_ as a line of different
granularities where you can zoom in and zoom out, and therefore you can
represent intervals in those lines, vectors and segments of any granularity and
apply the common well known operations like intersection, union, iteration, etc.
on any of them.

With Chalten you can represent relative points in time like _"20 days from now"_
or _"3 months since April 2014"_. Those relative points can be filtered with the
filtering rules of a "calendar", as a labor calendar that defines Saturdays and
Sundays as not working days, and therefore "20 days from now" will not be the
same applied that calendar that to another one where Saturdays are workable
days. These are, to name a few, features Chalten has since 2005 that the new
Java time model lacks.

So, is the new java date/time model "good"? It is better than the previous one.
Is it the best one? Sadly it is not if you compare it to other long ago known
solutions. But it definitely does not hurts you like other "new features" of
Java 8.

## The so waited Closures… wait Lambdas!

Let's start making clear that **Closures** are not the same as **Lambdas**, not
even **Full Closures**. The definition of these concepts will vary depending on
the literature, programming language, etc., but mainly the difference is how
they bind. With Java Lambdas you can not change variables of the current
lexicography context, where with Closures like the ones you have in C# you can,
but you can even do more with Full Closures where the return also binds to the
current context as in Ruby and Smalltalk. 

So, are Java Lambdas "good"? Well, definitely they are better that the freakish
anonymous classes because all the boilerplate to use them is gone, but in the
end are almost the same. The Java lambdas are mainly syntax sugar to avoid using
certain kind of anonymous classes and a minor (good) change on how they bind. 

Java lambdas will allow you to code better, to create better abstractions, to
reduce "lines of repeated code" but still, Java lambdas are not the "best
solution". Full closures would be better as proved by Ruby, Smalltalk, Scheme
and the famous ["Lambda: The ultimate x"][lambda-ultimate] series of papers
written of the last half of the 70s... yes, a long time ago.

Why Java 8 does not have Full Closures or at least Closures? Definitely not
because they are not good, definitely not because they are difficult or very
hard to implement, Ruby has them since its creation circa the same year as Java,
Scheme has them since the mid 70s and Smalltalk too.

## Null considered harmful

If I took @dhh's "Is TDD Dead..." phrase, why not used the so famous "x
considered harmful" derived from Dijkstra's well know "Goto considered harmful"?

I see with great joy that we are starting to realize that `null` is not a good
idea. We are starting to see more and more languages that are trying to provide
a solution to all the problems `null` provokes. Even the newly Apple's Swift
provides some aid allowing variables to reference `nil` or not, and send
messages with a "?" at the end that helps avoiding to check for `null`/`nil` on
each message send. 

What is Java 8 doing about it? It provides an abstraction called `Optional<T>`
that responds to messages like `ifPresent` (encapsulating the hated
`if x==null`), `orElse(T anObject)` that returns the wrapped object or the
parameter `anObject` if the wrapped one is `null`, and some others.

At first sight looks good, it is an abstraction that encapsulates all the
`if x==null` we are forced to write if we want to produce robust software. Looks
like a step forward, but is it? Let see what other languages have, for example
Ruby.

Let's start pointing a big difference. In Ruby `nil` is an object, not a
**"reserved word"** that the compiler recognizes and treats specially. So
because `nil` is an object it can answer messages like `nil?` that encapsulated
the `if x==nil` and many other you would like. The same in Smalltalk, `nil`
responds messages like `ifNil:` that encapsulates the
`if anObject isNil then ...` in just one `anObject ifNil ...`, and `ifNotNil:`,
and `ifNil:ifNotNil:`, etc. All very handy messages and all the new messages you
may need because `nil` is an object and therefore you can extend its protocol.

So what is the difference? The difference is that in Ruby and Smalltalk you
don't need a special abstraction to handle `null`, it is just it!. There is no
need for an `Optional<T>` abstraction to wrap an object and respond messages
you would like `null` to answer.

With time, we will start to realize that the `Optional<T>` class has the same
issues that the `Integer` class, the `Long` class and all the wrappers of the
data types used to represent numbers in Java because numbers are not objects. It
looks like we have learn nothing from the mistake of not having objects all the
way down. Let's see some examples starting to interact with the wrapped object.

``` java
...
Optional<String> aString = Optional.of("something");
aString.charAt(1); <-- It does not compile because aString's type is Optional
...
```

As you can guess, the example does not compile in the line 3 because `aString`
is not really a `String` but an `Optional`. So we have to send the message `get`
to `aString` to get the real String (does it sound as a tongue-twister? I
promise it is not my purpose :-) ):

``` java
...
Optional<String> aString = Optional.of("something");
aString.get().charAt(1); <-- Now it works because we are getting the wrapped object
...
```

Therefore, everytime we want to access the "real" object we need to send `get`
to the optional one. So if we want to send many messages we have to:

``` java
...
Optional<String> aString = Optional.of("something");
aString.get().charAt(1);  aString.get().charAt(1);
aString.get().charAt(2);
aString.get().charAt(3);
aString.get().charAt(4);... etc
... etc
```

To avoid sending that many `get`, we could just do:

``` java
...
Optional<String> aString = Optional.of("something");
...
String realString = aString.get();
realString.charAt(1);
realString.charAt(2);
realString.charAt(3);
realString.charAt(4);
... etc
```

But if we do this, why are we using `Optional` for? We can make a mistake and
assign `null` to `realString` and get back to what we wanted to avoid in the
first place. So `Optional` does not really stops programmers to send messages to
`null`.

Now let's go a little bit deeper. A common use case is to stop execution and
return from a method if a variable references `null`, like:

``` java
...
if (aVariable==null) return;
...
```

How does the new abstraction help us in this case? Let's see, it would be great
to write something like this:

``` java
...
Optional<String> aString = Optional.ofNullable(null);
aString.ifNotPressent({return;});
...
```

But it is not possible because there is no message `ifNotPresent` in `Optional`!
just `ifPresent`. Let's see how we can do the opposite with `ifPresent`. Let's
say I want to return if the variable is not referencing `null`:

``` java
...
Optional<String> aString = Optional.ofNullable("something");
aString.ifPresent(aNotNullString ->{return;});
System.out.println("got here");
...
```

What do you say? does it get to _"got here"_? :-) or does it return from the
method where this code is? Sadly it gets to _"got here"_, it does not return as
one would expect because lambdas are not full closures! That return exits the
lambda but not the method where the return is written.

So, how do we get out of this mess? How can return from a method if an
`Optional` is wrapping `null`? Writing:

``` java
...
if (!aString.isPresent()) return;
...
```

Please, you tell me the difference with:

``` java
...
if (aString==null) return;
...
```

What have we gain? Only more complexity. Are there any alternatives? How do
other languages treat this problem? In languages with full closures where null
is an object like Ruby or Smalltalk, you could just write:

``` smalltalk
aString ifNil: [ ^something ] (the ^ means return)
```

And that is all folks, simple, direct and does what you expect, if `aString`
references `nil` returns "something".

So, not only the `Optional` abstraction is a halfway solution (a fullway
solution would be to make `null` a first class object) but the lack of full
closures do not really help much here, on the contrary.

To end this section, I would like you to think a little bit more about this
problem. Is there another solution besides making `null` a first class object
and have full closure to solve this problem? I think there is. I think the best
solution is to get rid of `null`/`nil` completely. Not even the "?" of Swift or
C#. The truth is that we do not need `null`/`nil`, it has so many meanings, we
use it so badly, it is so error prone that the best solution would be to remove
that idea completely. So instead of `null`/`nil` we would have an abstraction
called `UninitializedVariable` to indicate that case, another called
`NoSuperclass` to point out the a class has no super class, or `EndOfList` to
indicate the we got to the end of a linked list or `UndefinedAddress` to
represent the fact that the user did not specify his/her address and so on.
Think about it for a moment, you may like it :-) but if not, watch Tony Hoare's
opinion about ["Null References: The billon dollar mistake"][null-references]

## Conclusion

Are the Java 8 changes "good"? Some of them yes, the date/time model is
definitely better of what we had, the way to represent code with lambdas is
definitely better that anonymous classes but sadly not good enough, and the
`Optional` abstraction does not solve the real problem but adds complexity. Some
of these changes are good if you are a Java programmer but some are not, even if
you are a Java programmer, because they add unnecessary complexity, and if you
are not a Java programmer, most of them are still ancient code, like for Ruby or
Smalltalk programmers, even for people that work with C#.

You may wonder what it is the goal of this post. Is it to talk/promote Smalltalk
or Ruby? Is it to show that I don't like Java? Not really. I wrote it because
I'm afraid that for people that only work with Java these changes may look
great, a step forward a better life. My fear is that those people may think that
the only way to treat `null` is with `Optional` because all the know is _"the
Java way"_, they only live in the _"Java world"_. My fear is that I will now
start interviewing people that instead of confusing Dates with Calendars will
confuse Closure with Lambda, people that will not see that instead of hiding the
problem under the carpet as with Optional, we should get rid of the real dust. 

So, I wrote this post in an attempt to open programmers mind, to show them that
there are other places too look at and learn, even if you make a living from
Java. If you really love Java, the only way to get a better Java is forcing
real, profound changes on it, not just changes that keeps back-compatibility. 

Will Java ever make profound changes on it? Will Java take better paths? I think
it will not. The back compatibility principle they apply is stopping Java to
evolve better. We will never see Java without `null` or at least a first class
object null, we will never see Java with a better type system, we will never see
numbers as objects in Java; its back compatibility does not simple allow it.

I would like to finish with something [Alan Kay][alan] said about Smalltalk, its
creation, when discussing the future of the language with his group composed by
great minds like [Adele Goldberg][adele] and [Dan Ingalls][dan] among others.
Basically he was not happy making Smalltalk a commercial language because he
knew that after releasing it, Smalltalk would stop evolving because the priority
would be to satisfy users/customers needs (backward compatibility among them)
instead of creating new features and experimenting new ideas.

In other words, when a programming language hits the market its future is
defined. 

[is-tdd-dead]: http://david.heinemeierhansson.com/2014/tdd-is-dead-long-live-testing.html
[dhh]: https://twitter.com/dhh
[rgsbol-talk]: http://scrumbolivia.com/?page_id=825
[pharo]: http://pharo.org/
[chalten]: http://smalltalkhub.com/#!/~HernanWilkinson/Chalten
[paper]: http://www.sciencedirect.com/science/article/pii/S1477842405000424
[aconcagua]: http://smalltalkhub.com/#!/~HernanWilkinson/Aconcagua
[lambda-ultimate]: http://library.readscheme.org/page1.html
[null-references]: http://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare
[alan]: http://en.wikipedia.org/wiki/Alan_Kay
[adele]: http://en.wikipedia.org/wiki/Adele_Goldberg_(computer_scientist)
[dan]: http://en.wikipedia.org/wiki/Dan_Ingalls
