---
author: Miguel Duarte
categories: []
comments: true
date: 2014-10-20T13:55:12Z
title: Trust in objects. They have the right to decide.
url: /2014/10/20/trust-in-objects-they-have-the-right-to-decide/
---

*Real devs don’t use if.* This slogan was my first contact with 10Pines, much
earlier than the start of my apprenticeship residence. The first time I heard it
just made a lot of noise in my head. Why would somebody want to eliminate the `if`?
What wrong could this little poor word do to us?

My goal with this post is to demonstrate why a real developer wouldn't want to use
`if`. For this I will explain some of the benefits obtained by the usage of an
alternative: Let the object take the decisions.

<!--more-->

Just as it sounds. The objects are completely capable of handling the task, they
just need to be allowed. The key is to take advantage of their polymorphic nature to
create better collaborations. 

Just an example
---------------

Let’s see an exercise solved in __Ruby__ where we can replace all the `if` statements
by collaborations. We’ll go step by step improving our model and seeing the
benefits of this approach. 

This is our assignment:

*Make a model of the numbers in OOP. Considering the existence of Integer
numbers and fraction numbers. The numbers must respond properly the messages
`#add` and `#subtract`.*

Our task seems to be easy, we all know how to add and subtract. However we’ll
focus on the model instead of the arithmetic part.

First approach
--------------

Let’s do a short analysis of the problem. Our goal is to make a model that
contemplates different types of numbers(Integers and Fractions) and allows to
operate between them. Our model should accept operations like this:

  * 1 + 1 = 2
  * ½ - ⅓ = ⅙
  * 1 - ½ = ½
  * ⅔ + ⅓ = 1

It is important to note that the receiver of the message can be any type of
number. So does the external collaborator. In consequence we can’t always use the
same algorithm, we have to adapt it to the situation. For example: Adding two
integers is a straightforward operation, nevertheless adding an integer to a fraction is
not so simple. We have to do something extra to solve the operation (for example
multiplying the integer by the denominator of the fraction). In the case where the
two operands are fractions probably we look for a common denominator. In
conclusion, there are many correct algorithms for every situation, but we need
one per case.

Having all this in mind we can start build our model. It sounds pretty natural
to have a class diagram like this:

<figure style="text-align:center;">
  <img src="http://yuml.me/49d4db9b" alt="Class diagram">
</figure>

The class Number is abstract. Fraction and Integer must know how to add and
subtract any number. However the mechanism of the operation can’t be the same
for any type of number because the things we mentioned before. A First solution to
this problem could be the following:

```ruby
class Object
  def subclass_responsibility
    fail 'Should be implemented in subclass'
  end
end

class Number
  def + (anAddend)
    subclass_responsibility
  end

  def - (aSubtrahend)
    subclass_responsibility
  end
end

class Integer < Number
  def + (anAddend)
    if anAddend.is_a? Integer
      # returns the sum considering the integer
    elsif anAddend.is_a? Fraction
      # returns the sum considering the fraction
    else
      raise "Unexpected type of number"
    end
  end

  def - (aSubtrahend)
    if anAddend.is_a? Integer
      # returns the difference considering the integer
    elsif anAddend.is_a? Fraction
      # returns the difference considering the fraction
    else
      raise "Unexpected type of number"
    end
  end
end

class Fraction < Number
  def + (anAddend)
    #Implentation is similar to Integer case.
  end

  def - (aSubtrahend)
    #Implentation is similar to Integer case.
  end
end
```

The Number class is abstract. All his methods just fail an tell us
that is responsability of the subclass to implement it. The concrete
operations follow always the same patron: Ask the collaborator for
his class and then do the calc.

It works, but let's compare it with a solution without if.

Better Legibility
-----------------

The purpose of the if in this case is to decide what to do based on the class
of the collaborator.  However this isn't the only way to do it. The object
doesn’t know the external collaborators' classes by himself, but he knows
his own class, and therefore he can send the right message to the collaborator
with enought context information to do the right thing.

With this in mind  __1 - ⅓ = ⅔__ could be solved like this:

  1. We ask 1 to subtract certain number.
  2. The object 1 doesn't know what kind of number is the parameter, so he says
     to him “Hey, I’m an integer, please subtract yourself from me, having that
     in mind.”.
  3. The ⅓ know how to handle the specific situation of substrating from an
     integer, so the operation is performed and the expected result is returned.

Let’s see how it works:

```ruby
class Integer < Number
  def + (anAddend)
    anAddend.add_from_an_integer(self)
  end

  def - (aSubtrahend)
    aSubtrahend.subtract_from_an_integer(self)
  end

  def add_from_an_integer(anAugend)
    #returns the sum of two integers
  end

  def add_from_a_fraction(anAugend)
    #returns the sum of an integer plus a fraction
  end

  def subtract_from_an_integer(aMinuend)
    #returns the difference of two integers
  end

  def subtract_from_a_fraction(aMinuend)
    #returns the difference of a fraction minus an integer
  end

end

class Fraction < Number
  def + (anAddend)
    anAddend.add_from_a_fraction(self)
  end

  def - (aSubtrahend)
    aSubtrahend.subtract_from_a_fraction(self)
  end

  def add_from_an_integer(anAugend)
    #returns the sum of an integer plus a fraction 
  end

  def add_from_a_fraction(anAugend)
    #returns the plus of two fractions
  end

  def subtract_from_an_integer(aMinuend)
    #returns the result of an integer minus a fraction
  end

  def subtract_from_a_fraction(aMinuend)
    #returns subtract of two fractions
  end
end
```

Our Number class also has to be adapted to this new approach. Define the
methos in the abstract class is very important, because it help to understand
the model quickly and expand the behavior easy.

```ruby
class Number

  def + (anAddend)
    subclass_responsibility
  end

  def - (aSubtrahend)
    subclass_responsibility
  end

  def add_from_an_integer(anAugend)
    subclass_responsibility
  end

  def subtract_from_an_integer(aMinuend)
    subclass_responsibility
  end

  #{...}

end
```

Maybe at first sight it looks difficult to reads, however let's follow a simple
collaboration step by step to see that we obtain exactly what we are looking
for. 

```ruby
Integer.new(3) + Fraction.new(1,5).
```

  1. The message `#+` is sent to the object 3 with the external collaborator ⅕ .
  2. The 3 send the message `#add_from_an_integer` to the ⅕ with himself as external collaborator.
  3. The ⅕ know specifically what to do because they tell him to add an Integer.
  4. The correct result is returned.


Just taking advantage of the “go to definition” feature of our favorite IDE we
can follow this code without problem. Even debugging is more pleasant
because it's linear: You just need to follow the value of the variables, there is
no weird jumps in the code, you don't have to recreate conditions in your head to
find the next line of code to be executed. Note that the objects are handling the
decisions for us.

This point is the key. The objects are capable of handling this type of
situations (and others much more complex!) by themselves, they don’t need the
constant presence of a programmer manifested in the form of an if.Taking
advantage of polymorphism we can create objects that care themselves about the
situations.

Of course the essential difficulty of the problem doesn’t disappear. We need to
do the things right to get a correct result. For example in the message `#-` we
have to consider that the minus operation is not commutative. For this reason
the messages `#subtract_from_an_integer` and `#substract_from_a_fraction` must
consider the external collaborator as a minuend and not as a subtrahend. To
avoid problems the name of the message says it explicity.

Easy Expansion
--------------

Suppose we want to extend our model adding the class ‘IrrationalNumber’. In the
`if` approach this is just tedious. We would have to travel around all the
methods in every class and add a ‘elsif’ statement with the new behavior
(beside writing the new class). Moreover nothing ensures us that we didn´t miss
an `if` statement in another place.

Now we are gonna see how in the polymorphic version we can add all the necessary
code to make our new functionality work in an easy way without touching anything
of the existing code and being guided by the model itself.

Just to start we can create the class IrrationalNumber that inherits from
Number. At the beginning this class can define only the messages `#+` and `#-`.

```ruby
class Irrational < Number
  def +(anAddend)
    anAddend.add_from_an_irrational(self)
  end

  def -(aSubtrahend)
    aSubtrahend.subtract_from_an_irrational(self)
  end

  #{...}
end
```

Now is clear that any number should respond the messages `#add_from_an_irrational` and
`#subtract_from_an_irrational`, so we need to define this methos in the number class.

```ruby
class Number

  #{...}

  def add_from_an_irrational (anAddend)
    sublclass_responsibility
  end

  def subtract_from_an_irrational (aSubtrahend)
    subclass_responsibility
  end

end
```

At this point if we run any test with our new class the result will be the same:
“Should be implemented in the subclass”. And here it is: The model is not just telling us what
message should be answered, also it is telling us who has to answer it. “An integer
should implement the message `#add_from_an_irrational`”. The model itself
guides us over the expansion. It takes only writing some tests to know exactly how to
expand our model. 

```ruby
class Irrational < Number
  def +(anAddend)
    anAddend.add_from_an_irrational(self)
  end

  def -(aSubtrahend)
    aSubtrahend.subtract_from_an_irrational(self)
  end

  def add_from_an_integer(anAugend)
    # integer plus irrational
  end

  # {...}

  def add_from_an_irrational(anAugend)
    # irrational plus irrational
  end

  def subtract_from_an_irrational(aMinuend)
    # irrational minus irrational
  end
end
```

In consequence now all our classes need to know how to 
respond `#add_from_an_irrational` and `#subtract_from_an_irrational`.

The expansion was easy, we didn't need to touch any of the previous code and the
model guided us over the expansion. What else can we ask?

Conclusions
-----------

It’s true that at first, all this stuff of delegating the work from an object to
another “just to extract an if” can sound a little bit daunting. However… look all we have
accomplished. The final model is robust, simple to understand and pretty easy
to extend. We did a lot more than just “extracting an if”. In our polymorphic model
the objects are responsible of handling the problems and guiding us over the
extension of it.

The `if` is not "bad" by itself. In some situations it is just inevitable and
correct. Using the same example but with the `#/` message it's very difficult
to avoid the `if` that verifies that the divisor is not zero. To avoid that
`if` we would have to make a special class for zero and a specific dispatch,
that would make the solution too complicated. The `if` in this case is
completely justified.

I like to see the if as a "bad smell". Every time it appears I think
if I'm losing the chance of using polymorphism and improve my model.

Finally, I'd like to mention something important. In this post I put all my
effort to show the benefits of avoiding the `if`. However the
solution I use come out from nowhere, I have never explained the mechanism I used to
came out with the solution. So, there is basically two open questions:

  - Does a deductive method or an algorithm exist to remove the `if`?
  - Can we always replace the `if` with polymorphism? Is there a limit?


If you are interested in the answer I strongly recomend to see [this
webinar](https://www.youtube.com/watch?v=rnud1EjmHBM), where Hernán Wilkinson
explain this topic deeply.




