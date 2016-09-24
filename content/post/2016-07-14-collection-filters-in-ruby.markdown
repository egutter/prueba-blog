---
author: Nahuel Garbezza
categories:
- oop
- ruby
comments: true
date: 2016-07-14T13:00:00Z
title: Collection Filters in Ruby
url: /2016/07/14/collection-filters-in-ruby/
---

**Note:** acknowledgements (and thanks!) to Máximo Prieto (our OOP guru) who had this idea and he implemented it on
Smalltalk.

## The problem

We want to have a collection that filters its elements as we add them using a given condition. For instance, we can
have an `Array` that only allows even numbers.

First, and important, is to understand what a filter *is*. "filter" is a very overloaded word, and sometimes used in a
technical way. Let's say that a filter is someone with the single responsibility of deciding if something has to pass
over or not.

<!--more-->

## First approach

The first, and more intuitive approach, is to subclass `Array` and override `<<` and any other "adding" methods if
needed. Something like this:

{{< highlight ruby >}}
class FilteredArray < Array
  def self.filter_with(condition)
    new(condition)
  end

  def initialize(condition)
    @condition = condition
  end

  def <<(element)
    if @condition.call(element)
      super
    end
  end
end
{{< / highlight >}}

Which problems does it have?

- It is specific to `Array`s. if we want to filter other kind of collections, we need to create specific subclasses for
each of them. We can move a step forward and try to define a `Filterable` module or something similar, but we will need
to touch collection classes anyway. So we will avoid mixins.

- It uses an `if`. Remember, \#RealDevsDontUseIf (it's in our T-shirts, we take this seriously). We are losing a
concept here if we limit the solution to just put an if, just because it _looks_ simpler.

- The filter is not reified. We just _hacked_ something into `Array`. We still do not _know_ what a filter _is_.

## The idea

There should be a filter object, and it should be placed between the user and the filtered object. Imagine you are
modeling a person using a pair of glasses. Does the eyes tell the glass what/how to filter? No, glasses do!. So why
a collection need to decide whether to filter or not? is it in its essence?

Light goes first to the glasses, then glasses "decide" which light pass to the eyes. In a similar way, a "collection
filter" decide which objects passes and which don't.

## Implementing collection filters

The two main responsibilities of a collection filter are:

- Intercept the "adding" messages, to evaluate the given condition and decide what to do based on the result.
- Delegate any other messages to the collection.

This collection filter should act as an "invisible proxy", so the user does not know if they are using a filter or a
collection. It should be a polymorphic object respect to collections. To implement it, I subclassed my proxy object from
`BasicObject`. Maybe there is a better solution, but it solved my concrete problem here. It's not the purpose
of this post discussing proxy implementations.

Introducing the proxy object, and the definition of `<<`, we end up with this code:

{{< highlight ruby >}}
class UndefinedCollectionFilter < BasicObject
  def initialize(collection, condition)
    @collection = collection
    @condition = condition
  end

  def <<(an_object)
    if @condition.call(an_object)
      super
    end
  end

  def respond_to?(selector, include_private=false)
    @collection.respond_to?(selector, include_private)
  end

  private

  def method_missing(selector, *args, &block)
    @collection.send(selector, *args, &block)
  end
end
{{< / highlight >}}

Are we done? Can we go home? No! we still have the `if`. How can we eliminate it? Well, we have 3 different filtering
decisions, so we should have 3 objects representing each one. Let me introduce you those objects:

* **Undefined filter**: it is the "passive" filter, the object that is waiting for an object to come. As it does not
know if should behave as open or closed, we call it undefined. It's the object that we will pass to the user and it
will act as an invisible proxy.
* **Open filter**: it is the object that should let the object pass.
* **Closed filter**: in opposition to the open filter, this object should not let the object pass. 

After implementing the filters, and the logic to decide which filter to use in each case, the code resulted in:

{{< highlight ruby >}}
# collection_filter/undefined.rb
module CollectionFilter
  class Undefined < BasicObject
    def self.filter_with(collection, block)
      new(collection, block)
    end

    def initialize(collection, block)
      @collection = collection
      @condition = block
    end

    def <<(object)
      filter_for(object).add(object, @collection)
    end

    def respond_to?(selector, include_private=false)
      @collection.respond_to?(selector, include_private)
    end

    private

    def filter_for(object)
      filters_provider.find_filter(object, @condition)
    end

    def filters_provider
      Base
    end

    def method_missing(selector, *args, &block)
      @collection.send(selector, *args, &block)
    end
  end
end

# collection_filter/base.rb
require 'error_handling_protocol'

module CollectionFilter
  class Base
    def self.find_filter(object, block)
      filter_implementation_for(object, block).new
    end

    def self.filter_implementation_for(object, block)
      filter_implementations.detect { |each| each.can_filter?(object, block) }
    end

    def self.filter_implementations
      [CollectionFilter::Open, CollectionFilter::Closed]
    end

    def self.can_filter?(object, block)
      subclass_responsibility
    end
  end
end

# collection_filter/closed.rb
module CollectionFilter
  class Closed < Base
    def self.can_filter?(object, block)
      !block.call(object)
    end

    def add(object, collection)
      # nothing to do
    end
  end
end

# collection_filter/open.rb
module CollectionFilter
  class Open < Base
    def self.can_filter?(object, block)
      block.call(object)
    end

    def add(object, collection)
      collection << object
    end
  end
end
{{< / highlight >}}

Look at the `detect` usage to find the filter, and the `if` was just gone. The collection triggered the decision, not
us. That's good.

How can we use it? Let's take a look at some simple tests, as well as the `Array` extension to build the filtered
collection easier:

{{< highlight ruby >}}
# undefined_collection_filter_test.rb
class UndefinedCollectionFilterTest < Minitest::Test
  def test_it_does_not_add_an_element_that_should_be_filtered
    array = array_filtering_even_numbers

    array << 3

    assert array.empty?
  end

  def test_it_adds_an_element_that_should_not_be_filtered
    array = array_filtering_even_numbers

    array << 12

    assert array.size == 1
    assert array.include?(12)
  end

  def array_filtering_even_numbers
    condition = ->(element) { element.even? }
    Array.filter_with(condition)
  end
end

# array.rb
class Array
  def self.filter_with(block)
    CollectionFilter::Undefined.filter_with(new, block)
  end
end
{{< / highlight >}}

## Wrapping up

- This solution can be criticized because it has several classes, and it could be implemented in much less lines of
code. But, are the responsibilities in other potential smaller solutions well splitted? Are filters part of the
domain model of those solutions?

- Which object do we need to touch for modifying the behavior when the filter does not let you pass? Well, one single
object with that specific responsibility.

- It's good to evaluate an implementation not seeing only how easy it is, or how many lines of code it has, but also
validating all the responsibilities belong to the proper objects and how easy it is to be extended in the future.

- To me, filters are relevant concepts, that is worth to have as separate objects. These relevant concepts (and
hidden most of the time) make a difference and they are a sign of mature, well modeled systems.

- There's more coming! how we can use this filter with other kind of objects? is there more filtering options rather
than open/closed? these are some of the questions I'll try to answer in my next post.

- You can take a look at the full code at the [collection_filter repo](https://github.com/ngarbezza/collection_filter).
