---
author: Nicolás Perez Santoro
categories: null
comments: true
date: 2014-07-31T11:54:45Z
title: 'Two ways we understand code: Unitary vs Integral understanding'
url: /2014/07/31/two-ways-we-understand-code-unitary-vs-integral-understanding/
---

There is an often unresolved question in software about comments and documentation, about in what measure they should be used, and what should be commented and documented. About comments in particular it is often said that code should strive be "self-documenting", or that the tests should be (and are) the documentation, thus comments can be a sign of a lack of clarity in the code, a way to compensate for a weak or bad design, or an inadecuate test suite.

<!--more-->

I'm not sure what the True Answer might look like (maybe it all depends on each project and there is no silver bullet), but in general I find documentations and comments in most systems sorely lacking. To show why, first I'd like to introduce a pair of concepts that need to be taken into account in this analysis.

Unitary understanding vs Integral understanding
-------------
Look at this small code example:

``` ruby
def mysterious_method
  i = 0
  while i < 10
     puts i + 1
     i = i + 1
  end
end
```

We have here a small piece of code, a method called mysterious_method that for any programmer is not mysterious at all, 
and certainly requires no comments. I want you to look at the last significant line, "i = i + 1", and ask yourself "what
does this line does?"

Most programmers would answer "it increments i by 1". While this is not wrong, I want you to look to another way you might
answer this: "The line increments i plus 1 and allows the while condition to get from 0 to 9 while the "puts i + 1" sentence
spits the numbers from 1 to 10. In other words the sentences 'moves the loop forward'".

What did I just do? Many would question that I've just explained the whole method, not just the line. But does this line
 exist cut off from the method, or does it exists in the context of the method?

By "unitary understanding" I refer to the understanding of a piece of a system (in this case, a method is a system of 
interrelated expressions) by looking it only by itself, observing what it does from the point of view of the pure 
computation. And by "integral understanding" I mean to the understanding of the piece of the system not by itself but 
by looking at how it relates to rest of the system.

So, by the point of view of unitary undersanding, "i = i + 1" does what it reads: it increments i by 1. But from the 
point of view of integral understanding, we've still need to describe how does this line relates to the rest of the 
lines in the system. That's why I talked earlier about "moving the loop forward", I was relating the sentence to the 
while loop above.

Note that I'm not talking about anything technical, I'm merely elaborating on the understanding of the system parts 
in the minds of the programmers. These two levels of understanding exists, and every programmmer handles intuitively
 that the result of the method arises from the interaction of all the elements in the method, what I did is just give
  two different names to the different "levels" of understanding, which mirror unitary and integral testing.

Comments and integral understanding
-----------------------------------
Well, when we talk about adding a comment in a line to explain what the lines does, while the line might be 
self-documenting in the sense of an unitary understanding, it might very well not be self-documenting at all 
in the sense of an integral understanding. In the example given above, the code is self-documenting because 
it's short and simple: Any programmer can read it and have an integral understanding. But try to integrally 
understand what i = i + 1 does in this example:

``` ruby
class QuickSort
 
  def self.sort!(keys)
    quick(keys,0,keys.size-1)
  end
 
  private
 
  def self.quick(keys, left, right)
    if left < right
      pivot = partition(keys, left, right)
      quick(keys, left, pivot-1)
      quick(keys, pivot+1, right)
    end
    keys
  end
 
  def self.partition(keys, left, right)
    x = keys[right]
    i = left-1
    for j in left..right-1
      if keys[j] <= x
        i = i + 1
        keys[i], keys[j] = keys[j], keys[i]
      end
    end
    keys[i+1], keys[right] = keys[right], keys[i+1]
    i+1
  end
 
end
```

In the same way, if we see a piece of code that does an SQL INSERT, to understand it integrally, we need to 
understand how and when the table is updated, queried, and inserted elsewhere. In general, stateful code is 
more complex to understand than stateless code precisely because integral understanding is harder to achieve 
there, you need to have a clear view of the evolution of the system state through time. In those cases, comments 
are very useful to help you achieve integral understanding of what something does, something like "this insert here 
will trigger a sending of an sms because there is a supervisor process polling the table".

A quick top level documentation of the system can also help inmensely to have an integral understanding, it does not 
need to be complete or explain every little detail of the system. Your code might be very nicely structured, and have 
great namings and interfaces, but still, when faced with a pile of code that no one knows, I think no programmer would 
dislike from a little top level exposition of how the system works. Because the code might be self documenting... 
but not integrally. And in general I see that a lot of software projects lack documentation and/or comments, the code 
is expected to be self-explanatory, but code rarely (if ever) is self-explanatory in an integral way.

Automated tests and integral understanding
------------------------------------------
Tests can help you work through the integral understanding because they verify the integral assumptions of how the code 
will work. In general what automated tests do is to code the integral behaviour of your code so that you don't need to 
have it all in your head: good tests allow you to try a change, and if you broke the supposed integral behaviour, they 
will let you know. In some sense, that's why people will argue that in self documenting code, "the tests are the 
documentation".

For example, suppose you figured out an SQL INSERT inside a class was superfluous, you rewrite the unitary test, then 
rewrite the class, and you're done....only when you run the full suite and you break another test you figure out that 
this insert leaved a trail that was meant to be queried elsewhere. So even if the sql insert did not have a comment 
telling you why this insert was there, and you did not have the integral understanding to know what the insert did, 
the test suite had a way to tell you.

From this, one can easily conclude that in general, code should be tested as integrally as possible: It's dangerous to 
test separately two integrated components, at the very least if you test them separately you should test the integration 
itself in some place.

An example: If you have a component, let's call it Producer, that produces a change in the state of the system, and you 
have another component, let's call it Consumer, that reads such a change in the state of the system, many programmers 
will hardcode the change in the state of the system as would have made by the Producer component in the setup of the 
Consumer unit test, instead of calling the Producer component. If you do that then one could change the Producer component 
and the test suite will pass, without realizing the the relationships between the two components are broken.

Grades of understanding
-----------------------
Take into account that what we get to call "integral understanding" eventually depends on what we consider the "rest of 
the system", and this might not always be obvious. A good example is a public API: Are the callers of a public api part 
of the system, thus part of what we need to take into account as integral understanding? As soon as an API becomes public, 
we lose control of who uses, and eventually, we lose understanding of how the API interacts with the rest of the system, 
when inside the limits of the system we include the callers of the API. A painful side effect of this can be seen in 
Operating Systems and Browsers who preserve "quirks" and even "bugs" that turn into "features" because a set of sites 
or drivers depend on a behaviour that now cannot be changed.

In a sense, what you consider is the rest of the system can be as big as you want it to be. Another example, to understand a 
line within a method integrally, you need to understand how that line relates to the rest of the lines in the method. 
Is that enough? Well, no, because to understand the method, you need to understand how and when it is called and with 
which kind of parameters. Is that enough? Well, have you considered what happens if the system breaks down because it 
has no memory in the middle of the method? In the end, the rest of the system includes the compiler/interpreter, 
the machine that it is running on, basically any abstraction that can leak.

Human beings in the end are also an often ignored part of the systems, even if in the end, software is meaningless 
unless it impacts a human being in some way. And true integral understanding eventually relies on human beings, 
and while explaning why and in what sense is beyond the scope of this article, I think we can easily see that even 
if the code is self-documenting, even if the tests are beautifully written, the system gives away the *what* 
is doing but often lacks a *why*, as in *why does the system behaves like this instead of that*. In enterprise systems, 
where business rules can often get convoluted and seem arbitrary, it applies even more. In the end the code plus the 
test suite can have that *why* lacking and sometimes there are good reasons to document the why, if the why is non obvious.

As we can see, integral understanding thus in the end is *hard* if not *impossible*, and no one has complete integral 
understanding, yet another reason why tests are helpful, they cover up for this deficiency.

Can code be self documenting?
-----------------------------
Fans of the "code should be self documenting" idea will point out that your code needs to have an easy, self documenting 
structure. A simple, well written system will be easy to understand integrally compared to spaghetti code. So you still 
need to try to write code that needs the least amount of comments, code where the integral structure is as self evident 
as possible, where you can figure out the impact of a line of the code by tracking (with a little help from the IDE)
the scope of the full integral impact of the line. And they are correct.

And while it's true, we need to take into account those two levels of understanding, and try to be as helpful as possible 
to explain to a fellow programmer, or maybe yourself in the future, what may not be so obvious to understand from a glance.