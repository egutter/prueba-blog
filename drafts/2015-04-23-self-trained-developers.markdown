---
layout: post
title: "Self-Trained Developers"
author: Nicolas Papagna
date: 2015-04-23 01:10:25 -0300
comments: true
categories: []
---
You can find the Ruby Gem [here](https://rubygems.org/gems/error_handling_protocol).

> **Captain:** Don't you ever talk that way to me. *(hits Luke violently)* NEVER! NEVER!
> 
> *(Luke rolls down hill; to other prisoners)*
> 
> **Captain:** What we've got here is failure to communicate. Some men you just can't reach. So you get what we had here last week, which is the way he wants it. Well, he gets it. I don't like it any more than you men.

First time I heard this quote from the '67 film Cool Luke Hand[^1] was in the opening of Guns N' Roses song "Civil War"[^2].

You can imagine what both are about, but I think there's a twist to the immediate analysis we do when reading the line "failure to communicate".

In the film Luke didn't fail to communicate with the Captain, he intentionally chose to ignore his "advices" and suffered the consequences to do so.

Happens in real life, happens in software development.

In "Kent Beck's Guide to Better Smalltalk: A Sorted Collection"[^3], Kent and Ward  comment how the failure to communicate caused users not to fully take advantages of the capabilities of the abstractions they developed.

As developers sometimes we fail to listen feedback we get from our system or other developers. The more we ignore it, the more it becomes part of the team's culture to the point no one recognizes problems as problems at all.
 
Let's take a look at a situation to illustrate that. Suppose we're working in Ruby and were asked to add a new class that needs to be polymorphic to others already defined in the system.

What options do we have?

No class hierarchy at all
-------------------------

If these classes don't participate in a class hierarchy, we'll have a really hard time *manually* finding them and determining which messages should be implemented in the new class to make them polymorphic.

    class A
    	def m
	    	#...
    	end
    end
    
    class B
    	def m
	    	#...
    	end
    end

With such error-prone task at hand, chances are we'll fallback to other team members to accomplish it. Even if this "works", this raises the bar unnecessarily for developers joining the team.

Whenever we forget to implement one of these methods a `NoMethodError` will be raised, but that is not helpful at all: we can't tell whether someone sent the wrong message or it was the right one but we forgot to implement it.

Class hierarchy with an abstract class
--------------------------------------
By creating a class hierarchy we're enabling knowledge organization and discovery, making it easier for developers to find classes related the one being added.

Bad news it that Ruby lacks abstract class, so we'll just leave it empty:

    class A
	    # yeah, this is "abstract"
    end
    
    class B < A
    	def m
	    	#...
    	end
    end

This solves the problem of finding related classes, but we still have to manually go though all subclasses in order to  determinine which messages should be implemented in the new class to make it polymorphic with the ones in the hierarchy.

Taking a step back
------------------
Whenever we send a method that an instance of  new class was supposed to implement, a `NoMethodError` will be raised.

The problem with that is ambiguity: it is not clear wheter someone sent the wrong message or if someone forgot to implement it.
That means is the developer's job to read the related classes definition to sort that out, on a case-by-case basis.

By now we can imagine is a task we won't be pleased to face. Yeah, let's grab a cup o' coffe and work on that one later...

> **Team Member:** Don't you ever ask me about these classes again. *(keeps drinkin' coffee and browsin' Reddit)* NEVER! NEVER!
> 
> *(You go back to your desk and keep reading piles of class definitions; with other monkeys...err... developers!)*
> *(Making a stand)*
> 
>  **You:** What we've got here is **failure to listen**. Some **developers** you just can't
> reach. So you get what we have seen **in the previous examples**, which is the way he
> wants it. Well, he gets it. I don't like it any more than you men.

We failed to communicate whether the message should be part of the hierarchy's procotol or not. A `NoMethodError` isn't good enough, we have to do better.

In an ideal world the computer should teach us about our models, there is no need to bother know-it-all team members.

Making it easier for everyone
-----------------------------
The solution is as simple as simlulating abstract method by raising an error:

    class A
    	def m
	    	fail 'My subclass should have implemented me'
    	end
    end
    
    class B < A
    	def m
	    	#...
    	end
    end

Now we're talking! If someone forgets to implement a method in the new class, we'll nicely blow up an error describing what went wrong.

Well, kind of. We're not reporting the name of the method (we'll have to manually look it up in the stack trace), nor the class that should implement it, or whether is is an instance or class one.

Also, if we blindly follow this alternative we'll end up with lots of abstract method reporting code duplicated all over the place.

Wrapping up
-----------

Because we care about how knowledge is organized using class hierarchies and we realize that computers are betters than humans in detecting and reporting this kind of situations, we developed a tiny gem that will allow you to define a method as abstract and, if called, will raise an error describing all you need to know to implement it.

Free team members. Let computers do what they're best for, so we can do what we're good for: creating abstractions.

And please remember: this is not only about code, is about taking advantage of the resources we have at hand to be better developers. Responsible ones.

References
----------
 - As described in the [gem](https://github.com/10Pines/error_handling_protocol), this article was inspired in Smalltalk's Error Handling Protocol. See [Smalltalk-80: The Language And Its Implementation](http://stephane.ducasse.free.fr/FreeBooks/BlueBook/Bluebook.pdf), Chapter 6, page 102.
 - [^1] ["What we've got here is failure to communicate"](https://www.youtube.com/watch?v=yBBWUZfgRiw) quote from the 67' film [Cool Hand Luke](http://www.imdb.com/title/tt0061512).
 - [^2] ["Civil War"](http://grooveshark.com/s/Civil+War/3YCp2K?src=5), song from Guns N' Roses.
 - [^3] "Constructing Abstractions for Object-Oriented Applications", in [Kent Beck's Guide to Better Smalltalk: A Sorted Collection](http://www.amazon.es/Kent-Becks-Guide-Better-Smalltalk/dp/0521644372).
