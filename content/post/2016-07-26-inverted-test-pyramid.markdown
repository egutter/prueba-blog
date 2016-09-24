---
author: Santiago Nodar
categories:
- TDD
- Tests
comments: true
date: 2016-07-26T17:39:03Z
title: An Inverted Test Pyramid
url: /2016/07/26/inverted-test-pyramid/
---

I believe that most of us are familiar with the concept of "Test Pyramid", a simple heuristic originally described by Mike Cohn, that is that a project should aim to have a larger number of unit tests than end-to-end tests. But after working on a project suffering from an inverted pyramid, I think it’s worth mentioning some of the downsides that relying solely on end-to-end tests might have.
<!--more-->
Don't slow me down
------------------

If we take developing a web app as an example, then you might end up using something like [Selenium](http://www.seleniumhq.org/) to build UI tests. Even with tools or frameworks like [Capybara](https://github.com/jnicklas/capybara), or [SitePrism](https://github.com/natritmeyer/site_prism) if you want to use page objects, that make building these tests more accessible, they are usually harder and take longer to write than unit tests. Moreover, they tend to be brittle; when new features are added or UI elements suffer small changes, they may break meaning time and work needs to go into them more often.
They are also much slower to run, so as your number of tests grows, so does the time it takes to run your whole test suite, slowing everything down further. This of course is to be expected, but you will reach bothersome run times sooner when you have a larger number of UI tests.
All of this adds up to a more frustrating working environment for the developer, and also to a **slower development process**, which translates into higher development costs.

Lack of feedback
----------------

Apart from the disadvantages already mentioned, I feel **their biggest drawback is the lack of feedback they provide for the developer**. Unit tests are usually self-contained, so when one fails, it leads the developer directly towards where the problem is. On the contrary when a UI test fails, it will probably need some tracking down to be done before finally figuring out what needs to be fixed. So if a UI test fails without a unit test failing alongside it, it probably means there is one missing.
As Roman mentions on his post ["Tests: Paving Our Way"](https://blog.10pines.com/2015/09/18/tests-paving-our-way/), tests not only help us achieve better designs and abstractions, they also double up as documentation for our code. Unit tests help document different concepts in the domain we are working on and more importantly how these concepts are represented in our code. On the other hand, end-to-end tests only tell us how a specific user might interact with the system in an specific use case scenario, which is important but not as useful for the developer, specially new team members who might be not familiar with the domain or code base.

Final thoughts
--------------

I’m not saying that end-to-end tests should be avoided completely as they help catch errors that might not be detected otherwise, they are specially important in testing critical use case scenarios. Let’s say you have a use case where an error might cost you thousands of dollars over the time it takes for it to be detected and fixed, then of course this should be tested as thoroughly as possible and end-to-end tests provide a necessary additional level of protection.
It’s just important to keep in mind they test different things and their purpose is different. We should take into account their higher cost when evaluating how to test the system we are working on. **The value they add should balance the time and effort that goes into them**. Finally, we should also never forgo unit tests even if there are higher level tests that cover the same code.
