---
author: Dario Garcia
categories:
- java
- config-management
- spring
comments: true
date: 2012-02-23T00:00:00Z
title: automatically change configuration based on current environment
url: /2012/02/23/automatically-change-configuration-based-on-current-environment/
---

What if the application **were aware** of its different running environments so it could change its configuration **automatically** based on the current one?

<!--more-->

If we wanted to do that, we had to solve 2 issues:

{% img http://2.bp.blogspot.com/-eiWPoCGfqzQ/Tz7KWbP8pLI/AAAAAAAAAEM/XzBpA6-RV3Y/s1600/steps.png %}

Fortunately **for the second issue** we have *Spring 3.1* which introduced the concept of profiles to bean definitions. This is similar to what maven does to files but applied to beans.

Now, we can define different set of beans according to the current spring profile. If we structure our configuration in different sets of beans for each desired configuration we can change it easily.

The problem remains in how to detect the current environment to tell which spring profile to use.
[(keep reading to know how...)](http://goodenoughpractices.blogspot.com/2012/02/automatically-change-configuration.html)