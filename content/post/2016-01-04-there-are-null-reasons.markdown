---
author: Dario Garcia
categories:
- null
- object-design
comments: true
date: 2016-01-04T10:32:10Z
title: There are null reasons
url: /2016/01/04/there-are-null-reasons/
featured: true
---

This post will try and maybe fail to convince you that using `null` in your code is an error.
For those of you willing to listen, here are my reasons to stop using it.  

<!--more-->

## Super brief history
The programming notion of null was introduced by Tony Hoare in 1965 for ALGOL W. In his own words, his intention was:    
"  My goal was to *ensure that all use of references should be absolutely safe*, with checking performed automatically by
the compiler. But I couldn't resist the temptation to put in a null reference, simply because *it was so easy to
implement*. This has led to innumerable errors, vulnerabilities, and system crashes, which have probably caused a billion
 dollars of pain and damage in the last forty years."   

The name is probably inspired on Math, as the notion of empty set, or 'no value'. And from Legal systems, as the notion
 of nullity, an entity that possess no legal significance. In programming, null is used as a value for any variable that
  represents 'no value'.

You know by now that it didn't prevent unsafe references (as originally intended). On the contrary, unsafe references is
 probably the number one error on programming languages that use it.  
The funny thing is that you would think that the problem is a human error, not a language feature unintentionally
designed to fail when used.  

# The two wrong use cases
I didn't believe it either. But our 10Pines version of the white mage warned us against using null/nil.
Some 4 years ago [uncle Wilki](http://blog.10pines.com/authors/hernan-wilkinson/) tried in vain to convince us that using null was a proven mistake. I thought he was only
ranting and discredited the idea: 'So many people using it can't be wrong'.   
Nevertheless I payed attention, and over the years I collected information, trying to prove him wrong.  
Luckily, I was the one proven wrong and now I understand why.  

There are two use cases were null is used and they are both wrong. But they are so common, and feel so natural that you
don't question them. That's the trap of null. It's so easy to fulfill those use cases with null that you don't think
about the right way of doing it.

## As initial value
The first use case is introduced by the language itself. An uninitialized reference variable by default gets a null/nil
value.  
Making you think that if you don't know the value of a variable, it's ok to set its value to null.  
Harmless as it seems, an unsafe null reference cannot be detected on all cases by humans, static compilers, or even
flow analysis, and thus an uninitialized variable can be accessed on runtime producing an null reference error.  

More importantly, by giving you an 'easy to implement' *undefined* value, it generates a hole in your design.  
Instead of defining your own behavior for uninitialized values, you take the language default which falls short and
many times it's completely undescriptive.  

Look at null reference errors for 3 common languages:  
java:
```
Exception in thread "main" java.lang.NullPointerException
at ....
```

.Net:
```
System.NullReferenceException: Object reference not set to an instance of an object.
at
```

or javascript:
```
TypeError: foo is undefined
TypeError: foo is null
```
Without looking at the code there's no way to find the error, and sometimes you can only discover it by debugging.

Instead of using null as a default value, take the chance to think on the expected behavior of your system when that
happens. Complete your design by using a NullObject if appropriate default behavior can be defined, or add an
error object that describes the error with context that is helpful to spot the original cause. Add domain semantics
to the problem by creating a custom *undefined* value instead of null.    

Look how mockito informs null references, when configured to do so:

```
org.mockito.exceptions.verification.SmartNullPointerException:
You have a NullPointerException here:
-> at test.SNPETest.doSomething(SNPETest.java:23)
Because this method was *not* stubbed correctly:
-> at test.SNPETest.doSomething(SNPETest.java:22)

        at test.SNPETest.doSomething(SNPETest.java:23)
```
It's a little more helpful, right? What about an error message specific to your domain? It only takes a little more work.
 But it pays off big time, and it's not that much if you use a generic tool to generate them.

Just by doing this, you will reduce a percentage of your bugs because you won't have automatic omissions. By removing
null as a valid initial value, every variable needs to be initialized explicitly, and thus you will need to think about
it, or have a better error description when you don't.  
Some errors will still persist, but hopefully with a better detection mechanism in place.

## As optional value
The second use case, and the most difficult to eradicate, is the use of null to represent the absence of a value.  
I know that using null for its purpose sounds like the logical thing to do but the problem is that null falls short. It
doesn't have the behavior that you need from it.  
It's no wonder that language designers need to complicate the language to deal with null values by adding operators that
 don't follow the object-message principle, like Ruby's `&.`, or Groovy's `?.`. You have no polymorphism to use with
 null. Null is a value for all types, that behaves like none. Once you have null, you have no object.

That's why these operators come handy when dealing with 'optional values'. But again, the solution falls short.
By using these operators you deprive your design from behavior that is needed to represent the absence of a value
(a case that is part of your domain!). To complicate things worse, the error description will be the same that is used
for uninitialied values.  
Accessing an optional value as it was present, is an error that you can better describe with custom errors, or prevent
completely by using better tools than null.  

## Alternatives to null

In a nutshell there are 4 alternatives you can choose (in no particular order):  

a. a null object for default behavior  
b. a block/closure to handle the absence  
c. a maybe object to reify the absence  
d. don't make it optional  

To put it in an example, let's say you need to implement a method to find a user. You can do:

a) NullObject
```
findUser(credentials)
  return unknownUser

```
If you know the expected interface of the returned object, and you can guarantee that in every context that is going to
be used a default behavior is acceptable then you can return a default object.  
That object can then be interacted with as a normal user.  
For the sake of the example, let's say that the unkown user can redirect you to a registration form as its main page.

b) Explicit block
```
findUser(credentials, blockIfNone)
```
If you can't anticipate the expected behavior, you can ask for it.  
Add a second parameter to your code that accepts a block of code to be executed when the user is not found. That forces
your client to define what to do in the case of absence, and makes explicit that scenario. The client code will then
have to decide its course of action as part of finding a user which removes the possibility of ommitting that detail.


c) Reify the empty set/result or Maybe
```
Maybe<User> findUser(credential)
```
If your client code can't define what the correct action for absence is, use a `maybe` object to reify the optionality,
and return that object, instead of the user.  
[Taken from functional programming](https://en.wikipedia.org/wiki/Monad_%28functional_programming%29#The_Maybe_monad) a
'maybe' object responsibility is to represent an optional value (not the value itself, but the uncertainty of it).   
Instead of sharing the same interface as the user, the interface of a maybe object has semantics specific to the
presence or absence of values, and how to deal with it.    
This option defers the decision of what to do for absence to the moment the user is actually needed. And because this maybe object responsibility is to represent a set, it can share semantics with collections.   
If you need the name of the user, instead of doing `findUser(credential)&.name` and getting a possible string, or nil (which inevitable needs an if statement), you can do `findUser(credential).map(&:name)` which gets you another `maybe` for the name, delaying the problem until the name is really needed.  

d) Avoid optionality if not needed
```
findUser(credentials) throws NotfoundError
```
In some domains, the absence is not a valid state for the system. In those cases, don't consider the absence as a valid use case. Treat it as an error, and prevent it.  
For instance, when you create a new object, you shouldn't have optional instance variables. It's a code smell that may imply two classes in one (as pointed out by our Object Evangelist [npapagna](http://blog.10pines.com/authors/nicolas-papagna/) ). Don't allow nil parameters in the creation of an object.

Finally, don't use exceptions to represent absence as a normal case. They are not polymorphic, require additional code branches (code complexity) and more importantly should be used for exceptional cases. If the value is optional, its absence it's not an exception. It's part of the domain.

## Conclusion
I tried to be as brief as possible to keep this post short. Because of that I may be ommiting some details or edge cases.  
But my intention is to give you few simple rules to follow and avoid null at all cost.

If you follow these rules, or at least question the necessity for null and create your own, you will make me, uncle Wilki, npapagna and your future self a lot happier.
