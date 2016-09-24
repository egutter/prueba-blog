---
author: Alejandro Pulver
categories:
- jruby
- rspec
- parallel
comments: true
date: 2013-05-07T00:00:00Z
title: Parallel tests for JRuby
url: /2013/05/07/parallel-tests-for-jruby/
---

Recently, we've been porting one of our projects from Ruby 1.9.3 (MRI) to JRuby 1.7.3. There was a fair amount of work involved, mainly about Ruby gems and (in)compatibility with C extensions (which JRuby dropped support), but that will be left for another post. Here, I'd like to discuss the recent addition of JRuby support to the [parallel_tests](https://github.com/grosser/parallel_tests) gem.

Our tests take about 20 minutes to run (it's suite of ~3800 examples, which are currently undergoing refactoring to improve performance), but a lot of the machine's processing power is unused. So I thought we could start by actually running all the code we can, before starting to profile and tune it. Also, if the delays were caused by waiting events, it wouldn't help making the code run faster. This sounds good, but it didn't turn out to be that easy.

<!--more-->

First of all, parallel_tests depends on the parallel gem (which coincidentally doesn't support JRuby). But fortunately, it was only for monitoring the rspec subprocesses, and I switched to using threads (that also works on MRI, something I didn't expect and the author [discovered soon after](https://github.com/grosser/parallel_tests/pull/209)). Traditionally one could manage children processes by saving their PIDs, I/O file descriptors and use something like [select(2)](http://linux.die.net/man/2/select) (blocks execution until some children has output to read from). But in this case, additional processes were used just to monitor their only child.

A simpler solution than using *select(2)* was to use threads. These run simultaneously in JRuby, so it's no surprise it worked. It's interesting to mention that the reason it works in MRI is blocking: while one thread waits for I/O (like reading output from the children), other threads can continue. So, for example, if you have a program that waits a lot (something that can easily be seen with [time(1)](http://linux.die.net/man/1/time) if "user" plus "sys" are lower than "real"), it may run faster with threads even on MRI.

So creating processes was necessary, as we needed to launch more instances of rspec (this could be avoided if rspec supported threads or worker processes, but it doesn't). The second problem was related to JRuby's implementation of `Open3::popen3`; it doesn't pass the fourth argument to the provided block, and doesn't parse command-line options correctly. That also happens with *popen4* (a JRuby replacement for the open4 gem), and even if the normal "open" works with pipes, it doesn't capture stderr. So we ended up using `Open3::popen3` to spawn a shell, and pass the command through stdin (to avoid parsing issues). See the [pull request](https://github.com/grosser/parallel_tests/pull/208).

Finally, some tests failed because they had small delays to ensure correct ordering of operations, and the JVM delay was larger. So they were marked as pending for now. The performance gain wasn't much, partly because of another issue with rspec reporter (the class used by progress bars and formatters): shared examples are counted separately, instead of adding the time to the examples that used them (see [#164](https://github.com/grosser/parallel_tests/issues/164) and [#212](https://github.com/grosser/parallel_tests/issues/212)). But that's a story for the upcoming post...