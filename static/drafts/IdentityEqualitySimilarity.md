---
layout: post
title: "Identity, Equality, Similarity and why collections are doing it wrong"
author: Dario Garcia
date: 2014-10-20 13:55:12 -0300
comments: true
categories: []
---

It is attributed to **Heraclitus** that
> You could not step twice into the same river

If the river flows and the water that you touched the first time is not there anymore, can you say that you entered the same river?  
What is sameness?  And why is it important when programming?
This post is about equivalence relations (identity, equality, and similarity) and how to properly use them in our designs.

##Collections

I'm not going to tell you why now, but you are probably using some form of collection that is incorrectly
implemented. Is not the performance. It's the design. I'm sure of it.  
And I could bet you can't fix it without breaking everything or re-writing them again.
The problem: hardcoded equivalence relations.

Let me first, dig a little into equivalence relations. When you first encounter them, they seem the same. Equality, identity, similarity but they are not.
They are all about differences.

##Identity
To talk about identity, I need to discriminate between the logical notion of identity, and the psychological or sociological.  
Let me start with the logical notion

##Abstract identity
In abstract terms, identity is the relation each thing bears just to itself. The law of identity states that "each thing
 is the same with itself and different from another".   
So there are two important pieces here:   

1. identity is a *relation* (something that relates, or links two)
2. but only with one object (it's a reflexive relation)

Which is somehow funny. We can only relate one thing to itself, and nothing else. What can we use this relation with, if
it's only applicable to one element? What is the value of the expression "A is A", we know that "A" is "A"!

A modern definition by Gottfried Leibniz held that x is the same as y if and only if every predicate true of x is true of y as well.  
So if we could write all the possible predicates and compare their value of true for x and y. We could say "x is y" if
there is not predicate whose value of true is different for x than for y.  
But I find one predicate that will never be true for both: "x is named 'x' in this text", which is false for y.
To give you another example. Are these two apples, the same?:  

/images/apple_identity.jpg "Same apple"

Before you try harder, I can help you spot the difference: one is on the left. So it couldn't possibly be the same as the one the right!  
Can you see where I'm going? I can always find a difference!

The ancient greeks called essence to the minimum set of characteristics that composed the identity of an object.  
If we were to describe x with its properties "x is red", "x is big", "x is heavy" and all those statements were true for
y as well, then x and y would be the same.  
Things that have the same essence are the same thing, and if they have different essence then they are different things.  
So you could argue that the name 'x' and the position of the apple, are not part of their essence. We shouldn't care about those differences because they are not part of their identity, and thus irrelevant to the discussion.

I can accept that. But you have to accept that in order for an identity relation to be useful in a given context you need
to skip some of the possible predicates of that context. You need to ignore some of the real observable differences.  
Maybe that's why Wittgenstein wrote "to say of two things that they are identical is nonsense, and to
say of one thing that it is identical with itself is to say nothing.", or Kai Wehmeier has argued that
a binary relation that every object bears to itself, and not to others, is logically unnecessary and metaphysically suspect.  
Also Russell and Frege, among others have questioned the notion of identity.

### Identity on programming languages

Being so philosophically questionable, how is it possible that the identity operator is so useful in your language of choice?  
You probably didn't have to delve into meta-physics questions to use it. If you want to know if two variables are the same
object you just compare them by identity right?

Let me rephrase that. When you cannot tell if two objects are the same, the identity operator allows you to
confirm it, without checking the object characteristics.   
Then, it's a question of uncertainty: Identity relation is useful in a state of knowledge where you (as a programmer)
**ignore** if what *seems* two, is really one. So you reach for the programming environment to answer that.  
The programming environment (or language) will rely on a pre-defined definition of identity, comparing variables by
memory address, object identifier, UUID, or whatever.  
By disregarding some of the observable variable characteristics (like the name), it will focus on the ones that you care
about (points to the same memory address?).

So identity, in programming, is a very special type of pre-defined comparison that is dependent on the language
implementation, and it is based on properties that only the language has control over.  
If you mess with them, you may break identity completely. Like, for instance, persisting an object in the database, and
then retrieving it. Are they, the same object? If so, why the identity operator says different?


##Psychological identity
The other notion of identity is related to live beings and their definition of self.  
If you believe in the essence of things, defining the essence of a person over time is going to be really complicated.
Specially with cases of schizophrenia and other types of personality disorders (who is ordered anyway?)  
However it's rare for an adult person to look at herself into the mirror and think that the image is from another person.

Even if we [change all our cells](http://www.nytimes.com/2005/08/02/science/02cell.html) like the
[Ship of Theseus](http://en.wikipedia.org/wiki/Ship_of_Theseus), we still can recognize ourselves and say "this is me".
Even if we loose parts of our bodies. Even if the body is a video game character, we still recognize ourselves.

The notion of identity, in this case is a completely arbitrary and totally subject to subjects. There's no way to have
an objective definition identity for humans. And that's ok, because subjects identities are defined by the subjects. As opposed to objects identities (that are also defined by subjects).

And how do we identify ourselves? We ignore differences like the age, or the clothing, skin tanning or glasses, etc,
and we focus on characteristics we care about.

##Equality

If you accepted that identity is about ignoring the non essential (or irrelevant) characteristics of something and focusing on the essential (or relevant) to compare them, then you already know what equality is.  
Equality is a relation like identity, but is not restricted to essential characteristics. It can be
based on any characteristic.   
What's the difference between essential and non essential? Let me explain something first.  

In set theory an equivalence relation between elements defines equivalence classes (groups of equal objects),
by which we can split the group into partitions.  
If you have a set of objects and define their equivalence by color, then your classes could look like:

img /images/Equality_partition.jpg "Equality Partition"

If you define equivalence by number instead you can have something like:

img /images/Identity_partition.jpg "Identity Partition"

If you can think of an equivalence relation that partitions the group in such a manner that there's
only one element per partition, and one partition per element, then you have an identity relation!

The main difference between equality and identity is that equality does not guarantee uniqueness.
We could say that "each thing is the equal to itself and may be equal to another".  
Identity is the special case of equality that, in a given context, grants you uniqueness.

In fact, identity is the type of equality where your uncertainty level doesn't allow you to tell the difference
between two objects (it doesn't mean that there's no difference, you just can't see them).   
If you cannot find a difference, then they are the same to you.  
With equality, you can see the difference but it's not relevant to your context.

## Context
Equality and identity are both subject to context. If you change the context in which the object characteristics are
evaluated, equality can become identity and vice-versa, or nothing at all.

You may be happy with your pre-defined language identity, and that can work on most of your programs without problems.  
But identity, in most programming languages is conceived in the context of one running process (or machine).  
If you change that, let's say a distributed object environment with multiple running instances, your definition of identity needs to change to adapt to the new context. You will need some kind of UUID and use that as object identity.  
Again your identity operator is broken (useless).

What is unambiguous in one context, becomes ambiguous in another. The relevant characteristics change with the context.  
There's no object capable of retaining its characteristics in every context. Thus equivalence relations are subject to context.  
And what's more important equality is not an intrinsic property of the object, but a relation of the object and the
subject in a given context.

##Similarity

The last one in the set is also an equivalence relation. You can think of similarity as a super-type of equality.  
Identity and Equality are restricted to a set of characteristics that are equal to both objects. If the selected
characteristics differ, then the compared entities are not the same object, or are not equal.  
Similarity, however has some degree of tolerance to differences. You can think of it as the fuzzy side of equality.   
Something can be 90% equal to something else, and that's what you will call 'similar'.

Objects may have something in common that is relevant to a given context, but is not enough to be considered equal
 in other context. That's how hashcode works. You build a similarity function to group objects by same value,
 but then compare with an equality function to discriminate them.

Instead of rejecting and ignoring mis-matching characteristics, with similarity you accept them to a certain degree.
Just as fuzzy logic, truth is not a 100% true.


##Collections continued

After doing that long introduction, we are now able to talk about collections.   
(Did you find that hardcoded equals by the way?)

Most of the collection implementations I've seen fail to understand that equality is context dependent. They use the
concept of "natural equality" defined by the language for certain kind of objects and they rely on that definition as
a dogma. Disabling you to change if your domain is in another context.   
If you are lucky enough, your domain will stay on the safe context of "natural equality".

So you add an integer "1" to a list, and then you can retrieve its index.  
You put a string "hello" in your set, and you can ask later if that word is present.  
But what if the word we are looking into the set is "HELLO"? Should we consider it to be the same that be have? Equal? Similar?!  
You see where I'm going, right?

Databases have already dealt with this kind of problem using string collations, you can pick your character equivalence
relation between some nice predefined standards. And what about numbers?  
Is 1.0 equal to 1? What about 2.01111111111 equal to 2.01? Does your domain include fuzzy numbers? Then you need 3.9 to
be equal to 4. And what about complex objects that don't have a natural equality?  
For those objects you can probably override one equality relation, but that ties you to just one domain.  

We must accept that the solution is not to have different collection types (each per equivalence), or try to capture
the best equality definition, but instead leave that to the client code.
> Collections should be agnostic of the type of equivalence relation they are used with,
> and they should be open to different definitions.

When you ask a set if it contains an element, you are relying on the implicit equivalence relation.
When you ask a collection to find elements by a predicate, you are giving it a new equivalence relation, just for that
question. If the collection use some form of hashcode, then it's working with a similarity relation.
But usually, none of these concepts is reified within the collection, so they can't be changed without changing the source.

If collections don't allow you to change their equivalence relation, they restrict the context in which the can be used,
and the operations you can do with them.  
Many collection implementations can abstract their equivalence relation without affecting performance.

If you are designing new collections (or object containers) don't use "natural equality".  
Allow your user to supply his own definition for equality and set your client code free.
