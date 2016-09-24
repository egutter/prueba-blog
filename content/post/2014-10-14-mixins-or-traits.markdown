---
author: Lucas Giudice
categories: trait mixin java inheritance behavior interfaces
comments: true
date: 2014-10-14T14:03:36Z
title: Mixins or Traits? That is the Question
url: /2014/10/14/mixins-or-traits/
---

A couple of days ago, a discussion came up in an [uqbar foundation](http://www.uqbar-project.org/)[^1] mailing list about the Java 8 Interfaces Default Methods. They were named as «mixins», but I corrected them and called «traits without flattening».

After sending the mail, that last sentence kept ringing in my head, so I proposed myself to try to understand why was I calling them like that.
But first things first.

<!--more-->

#Mixins? Traits?

First of all, when I say **mixin**, I'm using the definition given by Bracha and Cook in their paper «Mixin-Based Inheritance»:

>A mixin is an abstract subclass that may be used to specialize the behavior of a variety of parent classes.


What does this mean? Some relevant things that came to my mind:

1. A mixin _is a class_, so it can define behaviour and state (i.e. methods and variables).
2. In particular, it's a _subclass_. We can call `super`  (even though we don't know yet who the parent class is)
3. It gives us a form of multiple inheritance, and it handles the [diamond problem](http://en.wikipedia.org/wiki/Multiple_inheritance) with linearization: The mixin is "embedded" in the inheritance chain, right before other class that understands the message that is implementing.

This is kinda difficult to explain with just some words, so let's write some code :)

_Note: I will be showing mixins in Scala. The problem is that Scala names them «traits», so it can be a little confusing. Please bear with me through this._

{% img right /images/mot-diamond.png 160 470 class and mixins hierarchy example %}

``` scala

trait BaseMixin {
  def m1() = println("this is m1 in BaseMixin");
}

trait MixinA extends BaseMixin {
  override def m1() = {
    println("this is m1 in Mixin A")
    super.m1
  }
}

trait MixinB extends BaseMixin {
  override def m1() = {
    println("this is m1 in Mixin B")
    super.m1
  }
}

class BaseClass {
  def m1() = println("this is m1 in ParentClass")
}

class SomeClass extends ParentClass with MixinA with MixinB {
  override def m1() = {
    println("this is m1 in SomeClass")
    super.m1
  }
}

```

The diagram on the right sums up the hierarchy of  the code above. So, what happens when we do `new SomeClass().m1`?

As we stated before, the method-call conflict is resolved by linearization. The console output of the evaluation hence is

{% img left /images/mot-linearized.png 100 600 classes and mixins linearized %}

```

this is m1 in SomeClass
this is m1 in Mixin B
this is m1 in Mixin A
this is m1 in BaseMixin

```

As we can see, the MixinB is executed before MixinA. If we have some conflict, we just put some mixin above the other. In the case of Scala, mixins are linearized from the outside to the inside of the declaration[^2], as shown in the image on the left.
This means that, if we change SomeClass as shown below:

``` scala

class SomeClass extends BaseClass with MixinB with MixinA {
  override def m1() = {
    println("this is m1 in SomeClass")
    super.m1
  }
}

```

when we call `new SomeClass().m1`, what we get is

```

this is m1 in SomeClass
this is m1 in Mixin A
this is m1 in Mixin B
this is m1 in BaseMixin

```

#So, what about traits?

I first heard the term **trait** in Ducasse, Nierstrasz, Scharli, Wuyts and Black's paper named  «Traits: A Mechanism for Fine-grained Reuse».

As we know, classes have two responsibilities: creating instances and defining behaviour. The idea behind the Traits approach it's to separate those responsibilities (let's follow the [single responsibility principle](http://en.wikipedia.org/wiki/Single_responsibility_principle) :) and let the classes as instances generators while traits defines the behaviour.

Things for taking into consideration:

1. Methods defined in classes precede the ones defined in traits.
2. The methods defined in the trait «have the same semantics» as the method defined in the class (Flattening property). It's like they were copied into the class.
3. All the traits have the same precedence. If a conflict arises, we have to explicitly handle it.
4. Traits provides methods, but can require methods too. This allow us to use methods that are not defined in the same trait.

Some code please! (For this I'm going to use Smalltalk)

``` smalltalk

Trait named: #BaseTrait uses: {}
	foo
       ^ 'foo in BaseTrait'

```

And we have another trait that uses the described above.

``` smalltalk

Trait named: #TraitA uses: BaseTrait

```

What happens if we want to define `#foo` in this new trait, and call `BaseTrait#foo` + something else?
We don't have `super`, because traits aren't classes. The `uses:` implies that the methods will be flattened, so there is no class hierarchy. And as far as we know, if we define `#foo` we are overriding the method that came with the trait.
In the end, we need some kind of conflict resolution, so we can call both `BaseTrait#foo` and `TraitA#foo`.

``` smalltalk

Trait named: #TraitA uses: BaseTrait @ {#fooBase->#foo}
"first conflict resolution strategy: aliasing the method
#foo as #fooBase (this is read as «fooBase will call foo»)"
    foo
	  ^ self fooBase, 'foo in Trait A'


Trait named: #TraitB uses: BaseTrait @ {#fooBase->#foo}
"same as above"
	foo
  	^ self fooBase , 'foo in Trait B'

```

And finally, we want a class, because we need to instantiate objects.

Of course, I want all the behavior of the traits described above, but if I just use them I will have conflicts in two methods: `#foo` (this one is the easy one to spot) and `#fooBase`. Why? because `#fooBase` is defined in both traits. It doesn't matter if it is just an alias, the object that use the trait behaviour will understand `#fooBase`.
All said, let's resolve some conflicts.

##The #foo problem
The thing is, I want both `TraitA#foo` and `TraitB#foo`. So I will take the same approach as before and define an alias for each trait:

``` smalltalk

Object subclass: #SomeClass 
       uses: TraitA @ {#fooA->#foo} + TraitB @ {#fooB->#foo}

```


##The #fooBase problem
Again, I want that behaviour in my class, as my `#foo` will have to call it. We could take the same strategy and define an alias in both traits, and the magic is done.

But if I declare an alias, I will be having two different methods with the exact same behaviour. I really don't know if this is bad, but it adds some extra complexity (and I don't handle complexity very well :P).
The other strategy is to exclude the method in the trait. If one trait excludes the method, then there is no conflict at all :)

``` smalltalk

Object subclass: #SomeClass uses: TraitA + (TraitB - {#fooBase})
"the method #fooBase will be excluded of the TraitB"

```

Therefore, applying both strategies, we have `#SomeClass` defined

``` smalltalk

Object subclass: #SomeClass
	   uses: TraitA @ {#fooA->#foo} + (TraitB @ {#fooB->#foo} - {#fooBase})
	foo
  	  ^'foo in SomeClass ', self fooA, self fooB, self fooBase

```


Then, when we call `SomeClass new foo` we get

```

'foo in SomeClass foo in Trait A foo in Trait B foo in BaseTrait '

```

As you can see, I resolved by hand every possible conflict, and it doesn't matter how do I write the traits in the `uses:` (`TraitA + TraitB` is equivalent to `TraitB + TraitA`). If I wanted the `TraitB#foo` before the `BaseTrait#foo`, I would simply change the method `SomeClass#foo` and it's done.

So, we have seen what mixins and traits are. We have now the tools to face the original issue; How can we define, using these terms, the Interfaces default methods?

But since this is already a loooong post, we will answer that question in the next one :). Don't forget to check the blog!

Lucas.-



[^1]: If you wanna know more about uqbar, mail me or anyone in the foundation (contacts are in the site)

[^2]: In Scala you can explicitly call the method you want, doing `super[MixinA].m1`
