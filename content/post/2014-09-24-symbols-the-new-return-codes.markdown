---
author: Juan Pablo Arnaudo
categories:
- ruby
- nil
- closures
- symbols
- exceptions
comments: true
date: 2014-09-24T16:45:03Z
published: true
title: 'Symbols: the new return codes?'
url: /2014/09/24/symbols-the-new-return-codes/
---

Recently I’ve read a [tweet from Yehuda Katz](https://twitter.com/wycats/statuses/504043642869542912 "Use symbols") in which he suggested an interesting idea, as an alternative for using nil as a result from a method: to use a symbol instead.

Well, this post is about the reasons why I think you shouldn’t use a symbol, and why even returning nil is a preferable option. 

<!--more-->

To better illustrate this, let’s work with an example. Suppose we have the following method:

``` ruby

class User

  def find_by_id(id)
    # retrieve something from the persistence layer
    user_record = persistence_layer.find_record_by_id(id)

    # build user from record
    parse user_record
  end

end

```

This is a common method when working with persistence, and that should find an instance of a class by it's id. If none is found, then the `find_record_by_id` method returns nil.

Using Nil
---------

OK. So what if we choose to return nil?
When we are trying to find an object by it’s id, a possible and valid outcome is that we can’t find it at all. So it makes sense that if we don’t find it, we return nothing:

``` ruby

class User

  def find_by_id(id)
    # retrieve something from the persistence layer
    user_record = persistence_layer.find_record_by_id(id)
    
    return nil if user_record.nil?
    
    # build user from record
    parse user_record
  end
  
end

```

But this oblige us to deal with a non-existent result.
The basic way to handle this is to check for nil each time we call `find_by_id`, like so:

``` ruby

user = User.find_by_id(1)

if user.nil?
  # do something to handle user not found
end
  # do something with the user
end

```

The problem with this implementation is that each time we want to find a user by its id we will have to check _if_ we found it, or _if_ we got a null result. (And we already know that [real devs don't use if!](https://www.youtube.com/watch?v=rnud1EjmHBM "Webinar: Real Devs don't use IF")).

A positive aspect of this implementation is that we can find out if we make a mistake: nil does not know how to answer messages. So when we try to use it, it will explode.
But with this little benefit, it comes that nil does not tell us where the problem originates, and it will only explode __if__ we try to use it. This means that a long time can pass between when we get the nil result, and until we decide to use it. Therefore, a nil related error is usually harder to debug.[^1]

So let’s leave this as it is for a moment and move on to the symbols implementation, to see if it helps us in any way. 

Using symbols
-------------

One positive aspect about symbols is that they are useful to describe what they represent, just as any variable name can and should. This allows us to be as descriptive as we want when we declare it. 
So, in our example, we could create a symbol called `:noUserFound` and use it as a return value.

``` ruby

class User

  def find_by_id(id)
    # retrieve something from the persistence layer
    user_record = persistence_layer.find_record_by_id(id)

    return :noUserFound if user_record.nil?
    
    # build user from record
    parse user_record
  end
 
end

```

This way, it would be much more descriptive than returning nil, because we can know that this symbol is the specific result of the `find_by_id` method (or any method that queries for an object that fits a particular condition), which is way less generic than the nil answer, given that nil can appear anywhere in the code, for whatever reason.

But besides the small benefit that we gain from a more descriptive reification of our empty result, not much has changed. We still have to ask if the result is empty, the difference being in that this time we should ask it like so:

``` ruby

user = User.find_by_id(1)
 
if user == :noUserFound
  # do something to handle user not found
end
  # do something with the user
end

```

So now lets consider the not-so-happy aspects of this implementation, which are bound to the symbols nature.
There are a few things that may be dangerous if not handled with care:

1.  First of all, symbols, as null, lack context. They don't provide information about the stack trace, or the collaborators, etc.
2.  Adding context to this scenario is an effort that the programmer, rather than the programming tools, must do. And to do this, the symbol must be given a descriptive name. So in the end, what we would be doing is using a symbol's name to try to represent and describe a certain context, which could be quite complex.
3.  IDEs usually do not provide the auto-complete feature when we write a symbol. And even if they did, the comparison between symbols is case sensitive, so you got to be really careful when you write them. This is particularly annoying when working in group, as the team has to follow yet another convention. And this is not a minor thing, combined with the following point:
4.  Symbols, as opposed to nil, know how to answer some messages. 

So what happens if we combine both last points mentioned above? Why is this bad?
Suppose that in our beloved ```find_by_id``` method we return a symbol, and somewhere after sending that message to ```User```, we try to validate the result. But when writing the validation, someone not that familiar with our fancy symbol convention writes the symbol with a different case, or spells it wrong:

``` ruby

user = User.find_by_id(1)
 
if user == :NOUserFound
  # do something to handle user not found
end
  # do something with the user
end

```

As `:noUserFound` is different to `:NOUserFound`, then this validation will be skipped, for the comparison result will return `false`.

The danger with this is that, later on, the symbol that escaped from our clumsy validation could continue to elude us, because it knows how to answer messages. 

So, suppose that our intention in the first place was to find an object by id so we could find it's size (for whatever reason).

Then, when we send the `size` message to our object, if that object is a symbol instead of the desired object, it will silently do as commanded, and no one will ever find out!

This can be avoided if we use nil. First of all because there is only one possible way in which you can write it. Plus we can count on the IDE to write it for ourselves, eliminating the risk of misspelling when validating.
And second, but not less important, is because nil does not understand messages. So the instant we send him a message, it will blow up with an exception and we can find out our mistake right away. Or at least we can instantly know that something is wrong.

From my point of view, this is still preferable to symbols.

To summarize
------------

So far, this is what we've got:

Nil

	+ Does not answer messages

	- Will explode only if (and when) we try to use it
	- Hard to debug
	- Repeated code

Symbols

	+ Descriptive

	- Be careful when you write them!
	- Annoying conventions
	- They answer messages, they can be evasive


But since we still haven’t found a proper design towards solving this common problem, I encourage you to see if we can find a better solution in the next post: [flow control with exceptions and closures](http://blog.10pines.com/2014/09/29/symbols-the-new-return-codes-part-2/ "Symbols, the new return codes? (Pt. 2)").

[^1]: see [Null References The Billion Dollar Mistake](http://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare "Null References The Billion Dollar Mistake")




