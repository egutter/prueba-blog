---
author: Lucas Giudice
categories:
- trait
- mixin
- java
- inheritance
- behavior
- interfaces
comments: true
date: 2015-05-19T17:15:00Z
title: Java Interface Default Methods. What?
url: /2015/05/19/java-interface-default-methods/
---

In my last post I've presented both traits and mixins, with the promise of identifying the Java 8 Interface Default Methods. So, let's get on it.

First of all, if you don’t know what I’m talking about, [read my previous post](/2014/10/14/mixins-or-traits/) -and _please_ ignore the fact that it took me almost half a year to write this second part :)

A quick summary:

 - **Mixins:** Abstract subclasses. You can call `super` from inside the mixin, and can define both methods and variables. Mixins automatically solves conflict by linearization (inserting the mixin in the method lookup chain from left to right). 

 - **Traits:** Allow us to define behaviour outside the class, but don't define state. The methods defined in the trait «have the same semantics» as the method defined in the class (Flattening property). It’s like they were copied into the class. You have to handle conflicts by yourself.

Now we can fully dive into Java Interface default methods.

<!--more-->

# How do we define a default method (whatever that is)?
Easy peasy:

``` java

public interface InterfaceA {
	public default String m1() { return "This is InterfaceA.m1"; }
}

```


The only difference with a Class method is the little keyword `default`.

So, assuming we have a class like the following

``` java

public class SomeClass implements InterfaceA { }

```

When we call `new SomeClass().m1()` the evaluation sysout is `This is InterfaceA.m1`. As I said, easy peasy.

# Aaaaaand... what about multiple inheritance?

Both traits and mixins allow us some kind of multiple inheritance. I'm trying to define default methods as some of the two, so is logical to expect this particular behavior in java 8 too.

Well, you are right. The way to do it is:

``` java

public interface InterfaceB {
	public default String m1() { return " This is InterfaceB.m1"; }
}

public class SomeClass implements InterfaceA, InterfaceB { }

```

And we have our multiple inheritance... or not?

## Conflict Solving

There are three rules to consider in the method lookup while dealing with default methods:

 1. **Classes win over interfaces.**  If a class in the superclass chain has a declaration for the method (concrete or abstract), you're done, and defaults are irrelevant.

 2. **More specific interfaces win over less specific ones*** (where specificity means "subtyping").  A default from List wins over a default from Collection, regardless of where or how or how many times List and Collection enter the inheritance graph.

 3. **There is no Rule #3.** If there is not a unique winner according to the above rules, concrete classes must disambiguate manually.

And, because of Rule #3:

{% img /images/compile-error.png %}

So, we need to solve the conflicts ourselves. Let's clean up our own mess:

``` java

public class SomeClass implements InterfaceA, InterfaceB {
	@Override
	public String m1() {
		return InterfaceA.super.m1() + InterfaceB.super.m1();
	}
}

```

Ok, it's not the prettiest solution, but it's definitely easy to understand what's happening there: I'm calling first to `InterfaceA.m1()` and then `InterfaceB.m1()`[^1]. 

With this override, if we reevaluate `new SomeClass().m1()` we get `This is InterfaceA.m1 This is InterfaceB.m1`.

# So? What it is?

There are two big things to take into consideration for defining (or deciding) between one or the other:

 - Default methods are defined in _interfaces_. As we know, we cannot declare variables in interfaces - there are just method declarations (and now, definitions); and
 - We don't have an automatic method for conflict solving (i.e. linearization), we have to decide _manually_ how we are going to call our methods.
 
With this things in mind, I'm inclined to say that **Interface default methods _are_ traits**.

Mixins can have state and solve multiple inheritance problems with linearization, while traits can’t' define variables and they let you deal with conflict solving.

Yes, Pharo's traits had more ways to solve conflicts, but I don't think that is enough to discard default methods as traits.

They are no perfect traits, but as a mostly-java developer, I'm already used to that :)

## Wait, you are forgetting the flattening property

Well... I don't _really_ know about this.

I know the object understand `m1()`. And I can distinguish between default and non-default methods[^2] (if that is useful for something ._.). 

What I cannot know is if the method is actually "flattenizing" (it's that even a word?) into the class. Nor I know a way to demonstrate it. So I decided it's easier to ignore that issue :) 

I think that flattening it's not the core of traits, it's just an implementation issue and a nice feature for analyze in academical environments. IMHO the real value comes from conflict solving and variables - or in this case, the absence of them.


# Some thoughts I came up with during this process
It is not particulary necessary to classify default methods. I see them closer to traits than to mixins. Some can disagree. Some others can have a different understanding about traits/mixins. The thing is, _there's no real reason for making this distinction_.

I did it just because I find things like these entertaining, not practical. In the end (i.e. day-to-day work) I call'em default methods, just like the rest of my teammates.

Another fun (and maybe practical) thing to do is find the best practices with these things. Default Methods are another tool we have in our hands, so let's try to get the best of'em while we build _good software._

But I've just getting into this path, so let's talk about that some other time :)

[^1]: It does looks [suspiciously similar to Scala](http://blog.10pines.com/2014/10/14/mixins-or-traits/#fn:2), doesn't it?
[^2]: Check this gist for the code: [https://gist.github.com/lgiudice/cd5981b38f04c979f5ee](https://gist.github.com/lgiudice/cd5981b38f04c979f5ee)
