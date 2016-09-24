---
author: Juan Pablo Arnaudo
categories:
- ruby
- nil
- closures
- symbols
- exceptions
comments: true
date: 2014-09-29T14:29:50Z
published: true
title: 'Symbols: the new return codes? (Pt. 2)'
url: /2014/09/29/symbols-the-new-return-codes-part-2/
---

Flow control with exceptions and closures
-----------------------------------------

In [this previous post](http://blog.10pines.com/2014/09/23/symbols-the-new-return-codes/ "Symbols, the new return codes?") we've considered the benefits and inconveniences of using nil and symbols as return values in a method. 
Now we’ll evaluate the possibility of throwing an exception, and yet another alternative: using a closure.

<!--more-->

So, before jumping in, let's bring to memory the example with which we were working.

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

This method should find an instance of a class by it's id. If none is found, then `find_record_by_id` returns nil.

Throwing an exception
---------------------

If we were to work with exceptions in this scenario, the code for our method would look something like this:

``` ruby

class User
 
  def find_by_id(id)
    # retrieve something from the persistence layer
    user_record = persistence_layer.find_record_by_id(id)
    
    raise ObjectNotFoundError.new(id) if user_record.nil? 
    
    # build user from record
    parse user_record
  end
 
end

```

And this is how we would use it:

``` ruby

begin
  user = User.find_by_id(1)
rescue ObjectNotFoundError => e
  # do something to handle user not found
end
 
# do something with the user

```

This solution is usually more acceptable than the ones we have previously considered.
It allows us to send a meaningful message to those who are expecting a non-null object, telling them that it was not found, and it also provides a way to choose the scope in which we want to handle the object-not-found situation. 

Plus, we automatically have context: and almost as much as we want to! Besides from getting the full stack trace printed out to our noses, exceptions provide us a way to subclass and specify errors, to add descriptions, and even supply the id of the user that we were looking for.[^1]

And if we don't want to catch them, it's OK too. We can let them reach our global error handler, and forget about it.

This is fine if we don’t mind using a try/catch block each time we send the message, or any message that we know collaborates with this one.
But it comes with the burden of needing to know what, and if, this message throws an exception. Something that without meta-programming, we lack in dynamic typed programming languages.

Another downside is that we would be using exceptions as flow-control structures, when instead we should be using them for what they are really meant to do: handle exceptional cases.[^2]
If having an unauthenticated user is part of our business logic, then not finding him in our database should be something quite normal.

So, what can we do if we don’t want to sprinkle our code with try/catch blocks, nor remember that we have to?
What if we want to have an object that represents the decisions we make in the flow of the program?

Then, we arrive to our final destination: closures!

Using Closures
-------------

What benefit do closures provide for us, and why?
Let’s see how we would use them in our method:

``` ruby

class User
 
  def find_by_id(id, &user_not_found_block)
    # retrieve something from the persistence layer
    user_record = persistence_layer.find_record_by_id(id)
    
     # do something to handle user not found
    return user_not_found_block.call if user_record.nil? 
    
    # build user from record
    parse user_record
  end
 
end

```

So, what has changed?

First of all, let’s pay attention to the signature. This signature by itself will remind us that each time we send this message, we should provide a way to handle an inexistent result. 

``` ruby

user = User.find_by_id(1) { raise "Hey! I couldn't find an user with id: #{1}" }
 
# do something with the user

```

Or even better!

``` ruby

user = User.find_by_id(1) { AnonymousUser.new }

```

This way, we could return an object that is polymorphic with what we are expecting (a ```User```), as opossed to the nil/symbols scenario.[^3]

And of course, if we don’t want to provide a block each time we use this particular method, the class of the object itself could define a default way of handling this, which could be, for example, throwing an exception:

``` ruby

class User


  def find_by_id(id, user_not_found_block=method(:default_not_found_block))
    # retrieve something from the persistence layer
    user_record = persistence_layer.find_record_by_id(id)
    
     # do something to handle user not found
    return user_not_found_block.call(id) if user_record.nil? 
    
    # build user from record
    parse user_record
  end

  def default_not_found_block(id)
    raise "Hey! I couldn't find an user with id: #{id}" 
  end
 
end

```

The second advantage of using closures, besides the friendly reminder of the object-not-found possibility, is that __it eliminates the need of checking for nil each time we send the message__ `find_by_id`. This is what allows us to encapsulate the `if`, and get rid of the repeated pattern.

In this matter, it is similar to the “throw an exception” solution, but with the benefit that at the same time it solves the exceptional case from within the method, it does so using a block provided from an outer context. And apart from eliminating repeated code, closures are flexible enough to allow us to recreate any of the situations we've reviewed so far (which tells us they are a good generalization).

The third advantage, also similar to the exceptions solution, is that because closures bind to the context in which they are instantiated, there is no context loss. So you still have access to the execution context (e.g. you could make use of the user id, if you needed to).

The fourth advantage is that using closures you can create your own flow control sintax.[^4]

And finally, in systems where performance is critical, closures tend to behave better than exceptions.[^5]

So, where is the trick? What's the downside? 
Surprisingly enough, sometimes the only reason for closures to be unpopular is the mental effort that one must do before being able to incorporate them into the way of thinking. But the truth is we have been using this since Smalltalk's “detect”!

To summarize
------------

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

Exceptions

	+ Descriptive
	+ We can choose the solution's scope
	+ Automatic context information
	+ We can choose not to handle them

	- Try/catch blocks everywhere
	- Need to remember which methods throw them
	- Normal flow handled by exceptions

Closures

	+ Friendly reminder
	+ Default way to handle error
	+ Flexible
	+ No context loss
	+ You can create your own flow control sintax
	+ Performant

	- Mental effort?

[^1]: more on writing exceptions in ruby [here!](https://www.youtube.com/watch?v=nlvCYJodigM&list=PLMkq_h36PcLA4yY58tQgj5FAXRzMaZAaY "Implementando Excepciones con Ruby")
[^2]: see [Don't use exceptions for flow control](http://c2.com/cgi/wiki?DontUseExceptionsForFlowControl "Don't use exceptions for flow control")
[^3]: see [Null Object Pattern](http://www.cs.oberlin.edu/~jwalker/refs/woolf.ps "Null Object Pattern")
[^4]: see [Lambda: The Ultimate](http://library.readscheme.org/page1.html "The Original 'Lambda Papers' by Guy Steele and Gerald Sussman")
[^5]: more on exceptions performance in JRuby [here!](http://blog.10pines.com/2013/05/18/parallel-tests-on-travis/ "Parallel Tests on Travis")

