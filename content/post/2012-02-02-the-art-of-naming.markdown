---
author: Hernan Wilkinson
categories:
- naming
- object-design
comments: true
date: 2012-02-02T00:00:00Z
title: The Art of Naming
url: /2012/02/02/the-art-of-naming/
---

Today we all know now that variables should not be called `x` or `y`, not even `i` nor `n` when we use the famous “for”... but do you know why? have you had
trouble naming a variable? and what about a method (message) or a class?

The truth is that naming in software development is really important, so important that I usually say that programming is *“the art of naming”*.

<!--more-->

But why naming is so important in software development? Because a name **“synthesizes” the “meaning”** of what it is being named. Names help us understand the “meaning” of the program, what the program is representing without thinking on the how.

Good names make our lives easier, on the other hand, bad names make us feel in hell. Bad names suck energy from us because we have to **“re-think”** every time we see that bad name what it really means, what it is the real purpose of that thing being named. At the end, bad names obligate us to do the work that other should have done, that is why we *“hate”* programs written by others and we *“hate”* them more if they are badly written, if the names are bad.

Bad names force us to “synthesize” again the “meaning” of what is being named and that is a big effort for our mind. Sometimes we find a good name and we can “rename” the bad name, but sometimes we do not have enough information, enough experience to find a good name and also sometimes (but only sometimes :-) ) our mind prefers to put energy in something else.

A good name has to provide enough information for us to understand the purpose of what it is being named. In the case of an object, its name has to make us understand its role in the context it is being named (it is not the object’s type what it is important but its name! I can remove the object's type from a program and keep understanding it, but I can not remove the object's name. If you don't believe me, see the dynamically typed languages :-) ). The name of a class (that it is also an object of course), has to help us understand the concept that that class is modeling, it has to help us grasp the behaviour of the objects that the class defines without having to see all the message the class defines. A class name “synthesizes” the meaning of all the messages its instances can understand.

Names are for humans, not for computers. A computer does not care about the name of a variable or a class, for a computer they are all symbols, but it is not the same for us. We as humans, interpret those symbols, we attach them semantic to be able to understand what we are reading. That is why a refactoring is not important for a computer but it is very important for a programmer, it is because we are providing semantic, meaning, to what we are refactoring. The extract method is one of the most paradigmatic examples of this process of providing meaning, that is why every time you have to maintain code you don’t understand, the first thing you have to do for every piece of code you end up understanding is to extract it to a method, but not to make it “nicer” but **to name that piece of code!** to provide meaning to that set of collaborations between objects that you are extracting, to help you understand the program the next time you read it.

And yes, naming is not easy, naming is hard. A name is a **“summary”** of what it is being named and no good summary can be written without understanding first what has to be summarized. Therefore, if you have problems naming something it is basically because you have not fully understood what you are trying to name.

For example, why classes are so difficult to name? because you have not completely understood what concept the class is representing. But why smart people, people that have degrees from universities, that are well versed and manage a vast vocabulary can not find a good name for a class? Basically because they don’t have “enough experience” with that “element” they are trying to mane.

Classes are the more interesting example when dealing with **“class based”** languages, like Java, C#, Smalltalk, Ruby, etc. In those languages we are “forced” to name a class when we did not have the chance to see, to play, to understand how its instances are going to behave. Those languages go against the natural way of learning, they enforce us to name things we don’t know and that is why we end up with class names like **“ObjectManager”, “ServiceHelper”, “ObjectDirector”** and so on, names that can mean anything, and that makes sense because at the time they were named they could represent anything.

So, if you feel bad with yourself because it is difficult for you to name a class, don’t be, most of the time it is the programming language fault, it is because the language is forcing you to learn in an anti-natural way. It is forcing you to name something you don’t understand, but yet, you have to name it, you have to “summarize” its meaning with a name... crazy.

That is why **“prototype based”** languages like Self, IO, JavaScript, that is class-less languages, are better to tackle new problems and domains that you don’t know. They are better because they don’t force you to name something you don’t understand, you just create it and use it, and then, when you have enough information, you name it. Working with Self or LivelyKernel (Dan Ingalls current project) makes you see and feel the difference.

So, are we stuck? we are doomed to badly name classes? Of course not! and now that we understand what the problem is, we can think on a solution. But before talking about the one I propose, it is important to understand that programming is not a “point in time” task, it is not something that happens in a moment but in a “lapse of time”. Why it is important to understand this? Because it will help us with the solution, it give us the opportunity to do things wrong for certain time until we can do it right.... until we can do the last step in TDD, the refactoring.

So, when you don’t know how to name a class do not use a **“bad name”** for it, do not name it with what gets to your mind at first glace but use a **“meaningless name”**, a name every time you see it, it tells you **“Hey! change me!”**. That is the difference between a “bad name” and a “meaningless name”. A bad name becomes a good name with the time... after reading ObjectManager many times we don’t see it anymore as a bad name, we get use to it and our mind provides it a meaning that it is not explicit but implicit in our head and we live happily ever after with that bad name.

On the other hand if a class is named “XYZ” or “ZZZ” or something like that, a symbol you can not provide any meaning, your mind will force you to change it some day, and that day will be the day you will understand what you are naming. Will that name be the best? Maybe not, but for sure it will be better that one you thought without enough experience.

Conclusion, if you don’t know how to name a class, don’t waste your time thinking three hours on that name because the only thing you will get is a headache and a bad name. If you don’t know how to name a class, give it a meaningless name until the time you can correctly name it. That moment will be after you have played sometime with its instances... Software development is a learning process, therefore incremental and iterative so don’t try to make it right at the first time, you won’t anyway... and that is why TDD is so important in software development, but that is a topic for another post :-)