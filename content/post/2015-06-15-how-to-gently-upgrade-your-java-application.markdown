---
author: Gisela Decuzzi
categories:
- migration
- java8
- development
- experience
comments: true
date: 2015-06-15T14:48:00Z
title: How to gently upgrade your Java application
url: /2015/06/15/how-to-gently-upgrade-your-java-application/
---

When I think about getting an upgrade I usually picture myself like this:

{% img /images/Feel_like_a_sir_in_hd_by_lemmino-200.png %}

But… as the time passes… I feel like Homer applying for the university, something like:

{% img /images/homer205ei9.png %}

Since in 10Pines we reached a successful migration I want to share with you our experience on the road. Also, I want this post to be as short as it can be in order to be handy and easy to read, so, if you have doubts please leave a comment!

<!--more-->

>**Context**:
>In 10Pines we work with a set of tools to help us building robust software. The main tools are Jenkins (+sonar+findbugs) for continuous integration, Github and Artifactory for artifacts management, among others. Our goal is to **upgrade Java without losing our environment** (including our metrics) in our existing projects.

#Lessons learned

##1) Retrocompatibility
We all know the efforts made by the Java team to allow retrocompatibility (maybe… it’s even too much), but… it’s true… so if you just change your java version and compile as Java 8 everything is wonderful.

BUT I dare you to go to your largest project and replace the first (and horrible) loop with a (beautiful) lambda and see what happens…

{% img /images/parliamentExplosion.jpg %}

The lesson learned here is that:

>Java is retrocompatible BUT the frameworks not necessarily are…

Specially if you use “too much reflection”... let’s say javaasit or Spring or something who depends on Spring (almost everything!).

##2) Be prepared to upgrade

Yes… you have to upgrade your tools and dependencies and development environments and your application server… 

> Basically, you'll have to upgrade pretty much everything, as a consecuence of 1)

Here is a list of technologies that we had to upgrade:

- Spring
- Spring security
- Quartz
- Jacoco
- Jenkins
- Findbugs
- Sonar
- Tomcat
- Eclipse

##3) Lambdas are not closures

I don’t want to start a flame war here trying to define the concepts… but… 
> If you consider a *closure* the one that you use in Smalltalk and a lambda the one that you have in Java 8 then be aware of the differences

###Context awareness:

####Scopes...

You can NOT modify references outside the context, example:
Smalltalk code:

``` smalltalk
total := 0.
numbers do: [ :number | total := total + number ]
```

Java (ideal) but **not working** code (because you cannot modify non final variables in the local scope):

``` java
Integer total = 0;
numbers.forEach(number -> total = total + number);
```

Java **working** code :

``` java
final AtomicInteger total = new AtomicInteger(0);
numbers.forEach(number -> total.set(total.get() + number));
```

Summary:

> You have the same restrictions as inside an inner class

####Return...

When you are in a lambda you return from the lambda and not from the whole context:
Smalltalk:

``` smalltalk
theReturnCase
	|example|
	example := [ ^5 ]
	example value
	^ 10
```

If you invoke `theReturnCase()`, you get 5!

Java:

``` java
private static Integer theReturnCase() {
	Supplier<Integer> example = () -> {return 5;};
	example.get();
	return 10;
}
```

If you invoke `theReturnCase()`, you get 10!

> I think that the best strategy is the one in Ruby where I can choose the behavior I want. Anyway it's allways good to know the result

##4) New default options

There are a lot of changes, *even the default options for generating javadoc*. This is important specially if you use a plugin like maven-release-plugin, because maybe you are as lucky as us… and now your build fails because your javadoc have some errors...

{% img /images/fuuu.jpg %}

This is because we have a new default when generating javadoc! It's a strict check, so if you have some errors like for example: `@see com.tenpines.commons.persistence.entities.Fechable` and this class has been renamed now you have a failing javadoc, and your build is failing.

You can correct it, in fact you should correct it. But you can also change the option of fail on javadoc errors.

##5) Streams have too much potential

{% img /images/tooMuchPotencial.png %}

Yes, they can be asynchronous, parallel, lazy ….
But for a simple usage they are verbose, ok we are java programmers and we are used to that, but time to time I think why Oracle? why?

Let’s say you want to select the older people from a list:

In Smalltalk:

``` smalltalk
people select: [ :p | p.isOlder]
```

Now in Java 8:

``` smalltalk
people.stream().filter(p -> p.isOlder()).collect(Collectors.toList()))
```

> We have to obtain a new object that can understand **filter** (the famous Stream), then we have to declare the filter condition and finally obtain the filtered list

Stream of Java8 works with operations and terminators, **collect** is a terminator which obtains a list containing the result of applying all the operations, in our example a list of Person. 
If you are a little curious and explore the Collector's  API and documentation, at least for me, it's quite confusing but fortunately we are provided with usefull implementations. For our example we want an innocent list and we don´t care about other stuff, so it's ok.

#Good things are waiting for you

Yes, we had a hard time in the beginning (specially, because we did this like one year ago), but the truth is that our code has been improving a lot, specially with the new Date API and the existence of lambdas, we are able to build better abstractions, and yes:

- For simple usage streams are verbose, but it’s VERY easy to build something to improve that.
- You have to upgrade, but it’s a good practice for you to be up to date, specially with frameworks that include lot of updates and bug fixes.
- Maybe your javadoc is failing but it sounds to me that it's time to think Why do you have wrong javadocs? Also, if you are in a hurry you can change the default easily
- Lambdas are not closure and that is really sad (at least for me)… although at the end of the day I’m happy to have this concept and we should push for improve it.

>Java 8 it’s not all about lambdas! We have a lot of interesting new features that gives us more tools to keep improving.

With more time we can write another post giving examples of our code improvements and talking about our “experiments” simplifying the language, but that is another story. Just for now:

> Upgrade should be the path to follow. Maybe it’s a little rough, but today we are very happy with our decision.




