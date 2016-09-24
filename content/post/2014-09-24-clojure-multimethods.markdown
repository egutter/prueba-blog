---
author: Santiago Nodar
categories: clojure
comments: true
date: 2014-09-24T16:36:22Z
title: State and Polymorphism with Clojure
url: /2014/09/24/clojure-multimethods/
---

A couple of weeks ago, I started to take my first steps on Clojure as part of my apprenticeship residency. Having very little experience with functional programming, I quickly run into a few situations that I wasn't sure how to handle. Setting aside Clojure's syntax, which even though I like its simplicity and consistency, I still find the number of parenthesis somewhat uncomfortable. This short post will be about a couple of differences with OOP that caught my attention: how to approach state and how to handle polymorphism.
<!--more-->
Quick note on state
-------------------

I started by doing some simple exercises and examples, that mostly consisted of mathematical functions or processes with a very clear input and output - so far, so good. But when I moved to slightly more complex problems that involved some kind of change over time, I was tempted to imitate imperative programming and use some kind of variable. In fact, Clojure offers a number features that allow this: [Vars](http://clojure.org/Vars), [Refs](http://clojure.org/Refs), [Agents](http://clojure.org/Agents) and [Atoms](http://clojure.org/Atoms).
For example, one of the exercises, which I had already implemented in Ruby, was modeling a tennis scoreboard. With OOP, I had an object that represented the game, and when someone scored I  could just send a message to that object and it would change the players' score.

``` ruby

game.score_a_point_for(a_player) 

```

At first, I tried doing something like this by storing each player's score in an Atom, but by doing this, most of my functions would have side effects, therefore violating their [referential transparency](http://en.wikipedia.org/wiki/Referential_transparency_(computer_science\)).
I ended up with a simpler solution by having a structure that represented the state of the game which was passed as a parameter to most functions, and most of them returned a new structure with the new state as a result. I think this solution looks simpler and more aligned with the functional paradigm.

``` clojure

(score-point-for a-player game)

```

Polymorphism with multimethods
------------------------------

Continuing with the same example, this function that calculates the new score could easily be solved with a series of `IF`s that concentrate all the game logic. I started by doing this but I wondered if I could use some kind of polymorphism instead, like I did with Ruby. In Ruby I modeled each of the scores (0, 15, 30, 40, A) as a set of polymorphic objects that know the next result after a player scores, thus distributing this logic. For example the Thirty points score looks like this:

``` ruby

class ThirtyScore < Score
  def initialize(player, game)
    @player = player
    @game = game
  end
  def point_scored
    @game.new_score_for(@player, FortyScore.new(@player, @game))
  end
end

```

While doing TDD, I started with the simplest cases and then arrived to the __deuce__ case which is a bit more complex. To solve the next score I realized I also needed the opponent's score, so I refactored the points scored method like this[^1]:

``` ruby

class FortyScore < Score
  def point_scored_on(opponent_score)
    opponent_score.resolve_new_score(self)
  end
  def resolve_new_score(opponent_score)
    opponent_score.advantage
  end
  def advantage
    @game.new_score_for(@player, AdvantageScore.new(@player, @game))
  end
end

```

Eventually I found that something similar could accomplished using [multimethods](http://clojure.org/multimethods). This Clojure  feature allows multiple implementations of a single function, but what method to execute does not depend on the object receiving the message but on a user defined __dispatch function__  instead:

``` clojure

(defmulti calculate-next-score-for (fn [player game] (game player)))

(defmethod calculate-next-score-for :zero [player game] 
(assoc game player :fifteen))

(defmethod calculate-next-score-for :fifteen [player game] 
(assoc game player :thirty))

(defmethod calculate-next-score-for :thirty [player game] 
(assoc game player :forty))

(defmethod calculate-next-score-for :forty [player game] 
(resolv-possible-victory-for player game))

(defmethod calculate-next-score-for :advantage [player game] 
(assoc game :winner player))

```

`defmulti ` defines a new multimethod with the dispatch function provided, in this case the function simply returns the player's current score, and based on this result it executes the proper method. By doing this we can avoid hard coding everything inside a big `IF`, and the resulting code makes it easier to differentiate each case, making it more declarative[^2].

Conclusion
----------

Maybe a more experienced Clojure developer would have done things differently, however this exercise was very interesting for me, as it allowed me to explore different solutions and how some things in functional programming can be done similarly to OOP. After this short time exploring the language, I still feel a little foreign to the functional paradigm but it served as a way to learn several useful concepts, including immutability and referential transparency.


[^1]: The complete solution for the tennis scoreboard in Ruby can be found [here](https://github.com/nodarsan/TDD-TennisScoreRUBY)
[^2]: The complete solution for the tennis scoreboard in Clojure can be found [here](https://github.com/nodarsan/clj-tennis/)
