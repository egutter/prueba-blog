---
author: Nicolás Papagna
categories:
- ruby
comments: true
date: 2014-03-04T00:00:00Z
title: 'Rock Your Ruby: The Value of Value'
url: /2014/03/04/rock-your-ruby-the-value-of-value/
---

Check out the Ruby Gem at [GitHub](https://github.com/10Pines/value_protocol)

## Introduction

Go ahead and ask the developer sitting next to you what is the thing that loves
the most about Ruby. It should come as no surprise that simplicity, flexibility
and expressiveness are the main reasons Ruby junkies just can't get enough of
it.

In Matz own words, seems like the 'go with the flow' philosophy was present
right from the start in the Ruby community:

{% blockquote %}
"Actually, I'm trying to make Ruby natural, not simple."
{% endblockquote %}

<!--more-->

## Cuttin' loose from the language cliché

I do believe in the principles and values Ruby was built upon, but I don't like
clichés. We should be critic about the restrictions a programming language
impose on us, and really pay attention when things don't feel right.

Here's a concrete example to show you what I'm talking about. Check out these
uses of the `:detect` message:

``` ruby

discount_rules.detect(lambda{ default_discount_rule }) { |rule| rule.applies_on? a_product }
students.detect(lambda{ anonymous_student }) { |student| student.is_named? 'John' }

```

Do they 'feel natural' to you? Do you think there is something wrong with them?
Take another look.

## Noise & Duplication

There are two things that fire my alarms about these examples.

First, the unnecessary noisy syntax[^1] needed to pass the block that gets
evaluated when no object matched the `:detect` condition[^2]. I feel that
departs from the original intent of Ruby (to feel natural to the developer) by
forcing the use of 'tech' terms.

Second, the presence of repeated code. I don't mind writing blocks if I really
need to, but they seem overkill when all I want is to return an object to handle
the "if none" case.

## Dealing with duplicated code

In the Object Oriented paradigm repeated code is a symptom of a missing
abstraction, a concept that is not being modeled. As a consequence, what would
be its implementation is scattered around the methods that were supposed to use
it.

It should be clear from the examples that repeated code does not mean repeated
text, but repeated collaborations of message sends. These examples were choosen
to demonstrate that repeating code involves repeating collaborations of message
sends, instead of simply repeating text[^3].

Let's take a look at what is repeated, and what is not, between the two uses of
`:detect`:

``` ruby

discount_rules.detect(lambda{ default_discount_rule }) { |rule| rule.applies_on? a_product }
students.detect(lambda{ anonymous_student }) { |student| student.is_named? 'John' }

# the collaboration pattern seems to be:
FOO.detect(lambda{ BAR }) { BAZ }

```

Whenever we want to return an object when no object matches a condition we're
wrapping it with a `lambda`, because that is what the `:detect` message expects
as first collaborator.

One way to go would be to add a new method to the `Enumerable` module that does
the dirty lambda wrapping for us, but that solution would only work for
`:detect`.

It would be great to come up with a more generic solution, one that works for
any method that expects a block to be passed in.

## Designing a generic solution

We would like is to be able to express these examples in a more natural way,
getting rid of the extra `lambda` parts:

``` ruby

# look ma' no lambdas!
discount_rules.detect(default_discount_rule) { |rule| rule.applies_on? a_product }
students.detect(anonymous_student) { |student| student.is_named? 'John' }

```

Let's start solving this "challenge" not only by writing a test first, but
writing the test's assert first:

``` ruby

it 'should be possible to pass any object when a block without arguments is expected' do
  apple = Object.new
  fruits = []
    
  fruit = fruits.detect(apple) { |fruit| false }
  
  fruit.should == apple
end

```

After running it (and watching it turn red) the failure message reveales the 
root of the problem:

    NoMethodError: undefined method `call' for #<Object:0x00000001390d68>

Besides the missing message implementation, what this really means is that
instances of `Object` are not polymorphic with lambdas (instances of `Proc`)
with respect to the `:call` message.

To make this tests pass, let's do the simplest thing possible and add the
`:call` method returning `self` to `Object`.

But hey, we can do better! What about being able to pass an arbitraty object to
the `:select` method (which expects an _implicit_ block that takes one external
collaborator)?

``` ruby

it 'should be possible to pass any object when an implicit block with arguments is expected' do
  apple = Object.new
  orange = Object.new
  fruits = [apple, orangee]
  apples_only = FruitFilter.new apple

  selected_fruits = fruits.select &apples_only

  selected_fruits.should have(1).item
  selected_fruits.should include apple
end

```

_FruitFilter[^4] is a test class I used to express better the intent of the
test._

This time, the failure message is a little more cryptic:

    TypeError: wrong argument type Object (expected Proc)

We need an instance of `Proc` to be passed in to `:select`. The unary `&`
operator converts blocks to procs, but `Object` is not a `proc` and neither
knows how to respond to the `:to_proc` message that gets sent when using `&`.

Let's do the simplest thing to pass the test, and implement `:to_proc` in
`Object`, returning a `proc` that evaluates `self.call` (that was implemented in
the first test).

Here's how `Object` looks like after passing the tests:

``` ruby

class Object

  def call *args
    self
  end

  def to_proc
    proc{ |*args| self.call *args }
  end

end

```

*There are still some test cases left to be consider to make sure everything
works as expected/nothing was broken (like backwards compatibility with `Proc`,
`Method` and `Symbol` `:call` and `:to_proc` methods)[^5].*

## What's the deal with the "Value of Value"?

The need to reduce the friction when working with blocks was motivated by the
way Smalltalk solves the problem. Any object knows how to respond to the
`#value` message (which behaves in the same as the `:call` message just
implemented).

Even though the implementation is pretty straight forward, just returning
`self`, relying on `#value` in Smalltalk allows us to express domain concepts
better, making the code easier to read by dealing with objects or blocks in the
same way[^6].

From my point of view, it is a little method that adds great value.

## Conclusion

By working the solution step by step through TDD, we're now in a position to
explain and tell the cause of repeated code in the initial examples: instances
of `Object` were not polymorphic with instances of `Proc`. That forced us to
wrap objects in lambdas so `:detect` could work as expected.

Two missing abstractions (methods, in this case) were implemented in `Object` to
get rid of repeated code, `:call` and `:to_proc` methods.

Besides the immediate benefit of the implemented feature, I really enjoyed the
oportunity to strictly follow the OO paradigm and TDD to see how far they can be
taken. I valued the fact that Ruby can be modified to make it fit my needs.

Do not take for granted a language is expressive or feels natural just because
that's what the documentations says and the community accepts it without
questioning. Learn from the past (as we did from Smalltalk in this case) to
avoid reinventing the flat tire.

Don't get trapped in the language cliché. Build your own Ruby.

[^1]: I'm talking about the `lambda`/`proc` keywords preceding the braces needed to create a `Proc`.
[^2]: see `:detect` method documentation at [RubyDoc](http://ruby-doc.org/core-2.1.0/Enumerable.html#method-i-detect)
[^3]: If you want to understand or learn more about this concept, enroll in one of our courses! See [Diseño Avanzado de Software Con Objetos I](http://www.10pines.com/training/listado-cursos/diseno-avanzado-de-software-con-objetos-i) and [Diseño Avanzado de Software Con Objetos II](http://www.10pines.com/training/listado-cursos/diseno-avanzado-de-software-con-objetos-ii)
[^4]: FruitFilter [source code](https://github.com/10Pines/value_protocol/blob/master/spec/fruit_filter.rb)
[^5]: Check the whole test suite at [GitHub](https://github.com/10Pines/value_protocol/tree/master/spec)
[^6]: Dont' just stand there! Go ahead, grab a copy of [Pharo](http://pharo.org/) (a Smalltalk dialect) and browse for implementors and senders of `#value`
