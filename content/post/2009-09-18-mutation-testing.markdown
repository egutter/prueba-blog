---
author: Hernan Wilkinson
categories:
- testing
- smalltalk
- esug-2009
- esug
comments: true
date: 2009-09-18T00:00:00Z
title: Mutation testing
url: /2009/09/18/mutation-testing/
---

During the 70s, mutation testing emerged as a technique to assess the fault-finding effectiveness of a test suite. It works mutating objects behavior and looking for tests to “kill” those mutants. The surviving mutants are the starting point to write better tests. Thus, this technique is an interesting alternative to code coverage regarding test quality.

<!--more-->

However, so far it is a “brute force” technique that takes too long to provide useful results. This characteristic has forbidden its widespread and practical use regardless new techniques, such as schema-based mutation and selective mutation. Additionally, there are no mutation testing tools (to our knowledge) that work on a meta-circular and dynamic environments, such as Smalltalk, so the compile and link time are the current technique's bottleneck.

This presentation will introduce the notion of mutation testing, analyzing its advantages and disadvantages with a Smalltalk-based tool. The tool uses the Smalltalk's dynamic and meta-programming facilities to notably reduce the time to get valuable output and help to understand and implement new tests due to its integration with the rest of the environment.

{% youtube omvClOja4Js %}

This presentation was presented at [ESUG 2009](http://www.esug.org/Conferences/2009). You can see the handouts here:

{% slideshare 2049016 %}

*View more [presentations](http://www.slideshare.net/) from [10Pines](http://www.slideshare.net/silvajorge).*