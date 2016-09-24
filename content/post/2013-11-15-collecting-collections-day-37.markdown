---
author: Dario Garcia
categories:
- collections
- research
comments: true
date: 2013-11-15T00:00:00Z
title: Collecting Collections - Day 37
url: /2013/11/15/collecting-collections-day-37/
---

[Previously on "Collecting Collections"](/2013/11/15/collecting-collections-day-37)

## Performance

I was designing in my head, analyzing how to implement each method, visioning dependencies between methods and I stumbled upon `size`.
Very simple to implement. Perhaps so simple that two different implementations popped in my head.

<!--more-->

The first option was something similar to Javas' with a cached integer for a fast response. If I add an element to the collection, I should add 1 to the integer representing the size. If the collection is asked about it size, the integer is returned. Super-fast!

But that meant that other methods (besides `size`) had to update that integer when modifying the collection. Mmmhh, something is not right if others have to do totally unrelated actions in their behavior for you to accomplish your responsibility, right?

The second implementation was similar to Smalltalks', counting each element every time that `size` is asked. No other method was involved to update cached values, behaviorally autonomous implementation, but iterating all the collection just to answer size? What if it's a really big collection, and I must ask `size` frequently?

It was a dilemma for a moment. Just for a moment.

I decided to discard performance, really, totally. Caching values and involving more than one method to maintain state didn't sound good at all. And I was concerned more about reusability, and semantic simplicity. Being able to define the behavior of one method in the simplest way and in terms of other definitions if possible.

I do care about performance, specially on collections. However in this visit to collections I prioritized the design principle of ["don't repeat yourself"](http://en.wikipedia.org/wiki/Don't_repeat_yourself). I tried to focus on reusing behavior while being semantically correct. I forced myself not to consider performance, not just a bit. I wanted correctness, simplicity, not performance. In all other methods to come, performance was out of the question.

If I took that restriction off, could I get some new ideas? If I rejected performance will I embrace better semantics? I accepted the idea and told myself: "Maybe later I could recheck some definitions to enhance and improve performance".

I decided to define `size` as a kind of `inject` where you start form 0 and increment one by each element. I didn't care if it was slower as the collection grew, the definition of size was elegantly simple, and reused the definition of inject.

The important thing was that while inadvisable in real world implementations, in my imaginary collections forgetting performance allowed me for semantically better definitions of some methods.

## Iteration

The other, real big dilemma was the kind of iteration. I mean, internal or external iteration? What's your favourite?

I found out that it was a good topic for arguments. People do seem to have a favourite option. It was strange for me because I liked both, and they are free. I mean, you can have both, right?

Luckily for rubists, Ruby followed that direction, and it lets you have it your way `each` behaves as internal and external depending on how you call it. In java you "could" probably have both, but the complexity of internal iteration without closures makes it a no-go. Smalltalk is also biased, but towards internal interation, as it's the choice made for `Collection` class. But of course, it is easier to implement external iteration in smalltalk, than internal in Java. At least prettier.

Aside from personal / subjective reasons I wanted to know what were the pros/cons of each option (as with any design decision).
I read from [here](http://gafter.blogspot.com/2007/07/internal-versus-external-iterators.html), [here](http://journal.stuffwithstuff.com/2013/01/13/iteration-inside-and-out/), [here](http://www.cs.ox.ac.uk/jeremy.gibbons/publications/iterator.pdf), [here](http://c2.com/cgi/wiki?InternalizeExternalIterators) and [here](http://okmij.org/ftp/papers/LL3-collections-enumerators.txt).

## Let's agree to disagree

When I talk about iteration I mean the process of traversing a resource (may be a collection, a file, a result set from a remote database, etc) in a specified order, to do something with it, applying an operation item after item.

An iteration has then a resource where you get items, a traversal that define which items and in what order, and an operation that defines what is done on each one.

Additionally you can view the iteration as an interaction between a producer code that knows how to generate the items, and a consumer code that knows what to do with them. This perspective is helpful in understanding the differences between iteration types.

## Differences

I was really surprised by the fact that they are not mutually replaceable. Though they have shared uses cases, there are a few use cases that only one type of iteration fits. Then why choose only one, if one doesn't fits all?

Let me tell you what I found about iterations:

### Pros:

* **External iteration: Reified traversal**

1. *Consumer decides when:* The traversal is abstracted as an iterator object given to the consumer, so the consumer of the iterated elements is in control of when the iteration must advance.

2. *Source abstraction:* Because traversal is abstracted, you can consume elements without knowing where they come from. You just need a reference to the iterator object.

3. *Deferment:* Because consumer is in control of the iteration, you can delay real iteration until the moment consumer ask for an item. This allows for lazy iteration of costly elements.

4. *Iteration sharing:* As an object, the iterator can be shared among consumers, passed as argument, decorated, augmented, etc. Thus allowing you to add, change, combine or share traversal semantics.

5. *Traversal reusability:* As traversal has to be an object, it is easier to detect recurring patterns for the way resources are iterated and reuse pre-existing definitions.

* **Internal iteration: Reified operation**

1. *Producer decides when:* The iteration is abstracted as a method that receives an operation to apply on each item. The producer decides when to advance the iteration, and when to apply the operation to each element.

2. *Destination abstraction:* Because the operation is abstracted to the producer, you can apply it to each element without knowing what is happening with them.

3. *Completeness:* Because the producer decides when the iteration advances, it knows when it is completed (started and finished), being able to manage resources associated to the iteration. It can close files, liberate connections, etc after the iteration is done without waiting for the producer.

4. *Operation sharing:* As an object, the operation can be shared among producers, passed as argument, decorated, augmented, etc.  Thus allowing you to add, change, combine or share operation semantics.

5. *Operation reusability:* As operation has to be an object, it is easier to detect recurring patterns for the actions applied to an item and reuse pre-existing definitions.

### Cons:

* **External iteration: Unfinished business**

1. *Uncompleted iterations:* Because the consumer decides when to advance the iteration, there's no way for the producer to ensure that the iteration is completed (started and finished). The consumer could potentially never finish the iteration.

2. *Resource leakage:* As a side effect of uncompleted iterations, if the iteration involves resources that must be freed, forever lasting iterations mean resource leakage. Unless the consumer notifies the producer when the iteration is done, there's no way for the producer to release the resources. (Specially with the default interface for Java iterators).

3. *Operation duplication:* Because the operation is not forcefully reified, distracted consumer code can repeat the definition of an operation. It's easy to express the same operation as inline code more than once on different methods, thus violating the [DRY principle](http://en.wikipedia.org/wiki/Don't_repeat_yourself), and making operations hard to reuse.

4. *Hard to implement:* As you want to make traversal an object, you have to retain traversal state in that object. Depending on the type of iterated resource, and the way you want to traverse it, that could be very hard to implement as you have to decompose a loop in steps, that only an method invocation advances (At least is difficult for me).

* **Internal iteration: One at a time**

1. *Non parallelizable:* Because the producer decides when to advance the iteration, there's no way for the consumer to make two coordinated parallel iterations (one step on both iteration at the same time). If you want to compare elements in order for two collections you will have a hard time trying to coordinate both producers. In languages with continuations or coroutines, I imagine you could achieve it by pausing each iteration on the operation, comparing elements and resuming, but it sure adds some complexity not for average joe.

2. *No way out:* The consumer doesn't have a normal way to stop the iteration in the middle. Unless the producer offers some kind of communication mechanism, the consumer has to use exceptions to abnormally cut the iteration, or be forced to consume every element produced until the end of the iteration.

3. *Resource eagerness:* As a side effect of "No way out" the producer could use costly resources not really needed by the the consumer, just to finish the partially needed iteration.

4. *Traversal duplication:* Because the traversal of the resource is not forcefully reified in the iteration, distracted producer code can repeat the definition of how you are iterating the resource. It's easy to express the traversal of a structure as an inline code more than once on different methods, thus violating the DRY principle.

## Conclusions

You don't have to choose one kind of iteration. Really! Try to keep both and use them where they fit best.
Adding internal from external iteration is trivial on many languages.
Adding external from internal can be impossible unless you have a "yield" construct.

I recommend you check how to do it in your language in case you need any of them.

Some of the cons I mentioned can be reduced by including concepts from one type of iteration on the other. For example you can reify operations though they are not needed with external iterations to avoid code duplication. The same for traversal with internal iterations, so you can detect recurring patterns. Again, depending on your language of choice, this can vary from trivial to horribly verbose.

Additionally add some form of communication between the consumer and the producer, so the consumer can tell the producer when to stop. On both options there's a negative consequence for not having this communication (Uncompleted iterations, Resource leakage, No way out, resource eagerness) so if you can, add it, and use it.