---
author: Alejandro Pulver
categories:
- jruby
- parallel
- ci
comments: true
date: 2013-05-18T00:00:00Z
title: Parallel tests on Travis
url: /2013/05/18/parallel-tests-on-travis/
---

Since the last post, a lot of things happened: some things were fixed, some tests got faster and I learned that some aspects of JRuby are just slow (for now).

First of all, the author of parallel_tests was very kind to not only accept my pull requests, but to provide feedback and even fix my broken code instead of rejecting it. There was a way in RSpec to correctly calculate time spent on each file, instead of doing it per example (which resulted in the problems mentioned in the previous post). This resulted in increased CPU throughput (at least in MRI) as all processors could be loaded at the same time, instead of having them working together only a fraction of the time (because some of them finished earlier and remained idle).

<!--more-->

## Ruby MRI

Unfortunately, the good part ended at that point, as there was another problem. We were disabling the garbage collector during each test, and re-enabling it after, an idea apparently taken from a famous success story for improving tests speed. It worked well for some time, but at some point (maybe because deeply nested contexts), it started to consume a lot of memory (like 2 or 3 GB, instead of < 1), which of course was inconvenient for running many of them in parallel.

One could easily disable those instructions, but it made our tests run even slower (like half an hour). So I looked for tuning parameters relevant to the garbage collector, and found this [interesting article](http://meta.discourse.org/t/tuning-ruby-and-rails-for-discourse/4126). After trying some parameters, both the time and memory consumption were substantially reduced, combining the best of both worlds. Even if Google's tcmalloc and Ruby 2.0 gave a little boost, the main trick was to run the GC less often (but frequently enough to avoid eating all RAM). Also keep in mind that the mentioned article was mostly interested in latency, while in this case I only cared about throughput, even if the optimizations we used were the same.

Finally, we merged our test categories "fast", "slow" and even the cucumbers into a single Travis CI task, which ran in 8 minutes (most of which is spent installing gems and setting up the environment)!

## JRuby

In JRuby that problem didn't even exist, as it just outputted a few warnings indicating that "GC.enable" and "GC.disable" weren't implemented. The problem in this case was speed (as before), so I suspected the equivalent of the previous issue but for Java. So I immediately started reading about the JVM garbage collector options, doing some profiling with jconsole, visualvm (and the great VisualGC plugin), and logging, I found nothing. Really, the old generation wasn't even used (as tests just create objects and throw them away) and the collection of the young generation was very fast (I even tried using all my 8GBs of available RAM to reduce the number of collections, but it gave no improvements).

Just about to give up, I tried to find an answer at the #jruby IRC channel. There, someone said it was JRuby's implementation of exceptions, which was about 18 times slower than MRI. The conversation is archived [here](http://logs.jruby.org/jruby/2013-05-16.html) (look for "alepulver"). Fortunately, with parallel_tests the Travis task took 23 minutes, which is not (that) bad. In the end, before JRuby 2.0, this is all we can get. I couldn't increase the number of processes beyond 5 because Travis killed them (even considering that with default settings, parallel_tests says the machine has 32 cores!).

By the way, most of the work I did for the initial JRuby patch was replaced by the standard Ruby "open" which supports pipes. It's simpler, but doesn't support reading from both stdout and stderr separately. I know it's for the better, but it's a little sad too (considering the hours it took)...

That concludes my efforts to make tests run faster as a bulk. Maybe later I'll try Zeus (for running one test instantly when developing) or pry-rescue (for doing TDD in a way inspired by Smalltalk).