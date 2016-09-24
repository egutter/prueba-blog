---
author: Federico Zuppa
categories:
- agile
- estimations
comments: true
date: 2014-03-21T00:00:00Z
title: Rethinking Estimations
url: /2014/03/21/rethinking-estimations/
---

<blockquote class="twitter-tweet" lang="en"><p>When someone asks for an estimate I always think of this.&#10;&#10;<a href="https://twitter.com/hashtag/NoEstimates?src=hash">#NoEstimates</a> <a href="http://t.co/siZsVxs3D4">pic.twitter.com/siZsVxs3D4</a></p>&mdash; Aaron Griffith (@Aaron_Griffith) <a href="https://twitter.com/Aaron_Griffith/statuses/444547284416475136">March 14, 2014</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

It's not like estimations are like this, right? This post is a trigger to
rethink estimations. To rethink why we do them, how much do we invest on them,
what is their ROI. Should we estimate or not? If you think why you estimate in
real life, you wouldn't think there could be something dangerous about them. We
all estimate how much it would take us to get to the bus station before going,
right?. However, things get much more complicated in software projects and in
organizations.

<!--more-->

Here's a few problems I've seen lately that may turn estimations into dangerous,
deceiving practices and could deviate them from their original purposes:

1. Translating points to hours
2. Re-estimating stories
3. Treating estimations as commitments
4. Investing more time estimating than actually doing it

## Translating points to hours

This means using a story point that is converted to a number of hours (e.g. 4 or
8 hs). You estimate then using these 'story points'. But are they really story
points? I think they aren't,  they are just a broader unit than the hour to
perform time estimations. Let's dig a bit further into the concept of story
point. A story point is a relative measure of effort. The idea is really simple.
If I have Task A estimated in 1 point and we think Task B would take double the
time, we put 2 points to Task B. We don't know how long it will take A, but we
know B would take double the time. Is that simple. The story point is an
abstract concept because it amalgamates a few different factors in one, such as
the effort, the uncertainty, the risk, etc into a number, the story point. But
the most important thing is that this number is relative, it doesn't represent
anything per se. It represents something when compared to the other estimated
stories in the same backlog. The rationale behind this way of estimating is that
"Humans are actually better at relative estimates, than precise measurements".

With this relative measurements, the team's velocity can be calculated over the
first few iterations and with this velocity, a more precise schedule can be
constructed. The only pre-requisite for it to work is that these estimates are
consistent among them. As Mike Cohn says in his
["Agile Estimating and Planning" book][book], velocity is the great equalizer.

{% blockquote %}
<strong>Velocity Is The Great Equalizer</strong>
What's happened here is that velocity is the great equalizer. Because the estimate for each feature is made relative to the estimates for other features, it does not matter if our estimates are correct, a little incorrect, or a lot incorrect. What matters is that they are consistent. We cannot simply roll a die and assign that number as the estimate to a feature. However, as long as we are consistent with our estimates, measuring velocity over the first few iterations will allow us to hone in on a reliable schedule.
{% endblockquote %}

But this work for story points that are relative. I am a bit eery of using the
same name to identify something that is different in nature because is basically
another measure of time. If velocity is measured, it needs to be clear that we
are relativizing these numbers, we are converting them into this abstract number
which is the story point. A practice that I have been using pretty successfully
for a few years is that of relativizing estimations during the progress of the
project. Teams that are used to estimate releases in days, they could do so.
However, when I measure the velocity, I measure how many of these 'points' the
team is able to complete (averaging them over the sprints). Same with the
sprints estimations. If the team estimates all its tasks in hours, they could
keep doing it but measure the actual number they are able to complete and use
this number as [yesterday's weather][weather].

## Re-estimation

When should we re-estimate? In short, re-estimation should only occur **when we
are certain we gained knowledge about the effort to come**. We re-estimate
because we understand better what lays ahead. Consider you are working on a
Sprint and you realize the story you are working on is bigger than what you
originally thought it was. Should you re-estimate it? Let's look at a couple of
examples (from real life!) before answering the question.

Let me tell you something that I've seen many times (simplifying the numbers to
make it simple): We had a backlog of 100 story points and we committed to
deliver 10 story points in the iteration (the projection was we were going to
need 10 sprints to finish). Of course things went awfully bad during the
iteration and we were able to finish just half of what we committed to, 5
points. Thinking about the reasons, we came to the conclusion that these stories
that we completed were much larger that what we originally thought (of course we
found things we didn't foresee. It always happens) and therefore we re-estimated
them (the stories we completed) to be 2 times bigger. After this re-estimation,
the numbers showed that:

* We completed 10 story points
* We had 95 story points remaining on the backlog
* We needed 10 more iterations to finish

What is the problem with this? We did it without thinking carefully about the
remaining stories. In hindsight, we cheated ourselves into believing that these
stories were bigger than others we estimated in the same size and therefore we
needed, at most, one extra sprint to finish. Of course, this wasn't true. Most
stories that came afterwards contained unforeseen things as well and the number
of sprints required to finish was closer to 20, not 11. In other words, our
velocity was near 5, not 10 in the first place and we shouldn't have done any
re-estimation at all.

The last story was using story points, but I've had my experiences with hours as
well. Let me tell you this (imaginary) one: we included 10 20hr stories in our
sprint (200 hrs of a total backlog of 2000 hrs). We started working and realized
the first stories were bigger than what we thought they were going to be. Twice
as big. We re-estimated them in 40hrs instead and we completed 5 stories. So we
'burned' 200 hrs out of 2100 hrs. Wait wait.. We burned 200 hrs? And we worked
200 hrs? We knew we were going to work 200 hrs. before starting the sprint
anyway, right? But did we get any insight about the remaining work?

Has any of this happened to you? I believe we fall too often into the trap of
believing that what we have just completed is bigger than estimated and what
lays ahead will not. We, generally, are optimists. Or we just re-estimate
without caring about the rest of the backlog. This is a real mistake. So the
question is: when should you re-estimate? I believe that you should re-estimate
when you are sure you understand how the current work impacted the remaining
one. In other words, if you are certain that the feature you built is double
than others with the same size, do it. **When in doubt, don't re-estimate. Let
the iterations flow. Measure and average your velocity** (velocity is the great
equalizer, remember?).

## Treating estimation as commitments

Collaboration over contract negotiation. It's among the
[4 agile values][values]. We all know collaboration between the business and the
team is of paramount importance in agile development. It doesn't work without
it. We all know that estimations are just another criteria that helps the
business prioritize and build plans. We all know as well that estimations are
just that... estimations. They can be wrong (the error will be bigger as
uncertainty is bigger). People that do plans are used (or they should) to make
them with this in mind, playing with probabilities and statistics. So why do so
many people try to treat them as commitments? Treating estimations as
commitments could trigger a myriad of dysfunctional behaviors that could kill
collaboration very easily. Padding estimations to be on the 'safe side'.. Or
'refactoring some code, since we are below the estimate and there is time..'.
Or the worst of all. The development team unwilling to change things because
they will go over their estimations. Those examples are the result of lack of
trust & collaboration. And these innocent estimates could be the culprit.

So how should we treat estimations? We should treat them as that, estimations.
The development team does estimations as good as they can. Business understands
estimations are made with uncertainty and have error margins. The objetive after
all is creating value, as a collaborating team and estimations are just another
tool that helps us deliver better value.

**So please, please: Don't ever ever treat estimations as commitments!!**

## Spending More Time/Money than actually doing it

Imagine you want to buy a laptop, and you aren't sure whether to buy the top of
the line apple mac book pro or the Dell one that seems to have the same features
at a cheaper price. Apple is $2000 and the Dell is $1500. As you aren't sure if
the Mac is worth those $500, you hire a consultant who charges you $500 to give
you a consultancy report. There you go! The report says you should buy a Dell.
And you just spent $2000!! :P This is really silly, but believe me, it happens
much more than we think. Yeah, **sometimes we spend more time/money estimating
a feature than actually coding it**. This happens without us realizing,
disguised in endless meetings with many programmers discussing how to implement
a tiny bit of functionality (perhaps a couple of guys discussing and the rest
looking/sleeping/etc). Just think about this: Imagine there's a small story,
around a day long, and 10 guys estimating it with planning poker. How much does
this estimation cost? With a few minutes, it could be near the cost of building
it. So always bear in mind how much time you are spending (actually investing)
estimating and make sure it's worthy!! If there's a lot of people in the room
and **the story that is about to be estimated is small and doesn't have much
importance, don't worry that much about it!** You think is about a day. Fine.
Don't waste time trying to figure out if it will be 6 hrs or 10 hrs! Invest your
money where it's worth!

## What about not estimating?

There is an interesting wave of thinking that proposes dropping estimations
completely ([#NoEstimates][no-estimates]), focusing on delivering the most
important features quickly and measuring the cycle time. If the team knows it's
capacity is to deliver X features weekly, there's no need to perform any
estimations (even better if those X features can be deployed to production).
Estimating is waste, after all. It doesn't add any value to the product. Working
this way fits perfectly in pull processes and leads to leaner ways of managing
the backlog and a more mature way of working. This fits perfectly in projects
with a lot of churn as well. Why trying to predict what is going to happen in 2
months if we can't predict what will happen in 2 weeks? Probably is better to
select the most important features, implement them and then ask again the same
question as suggested in [this post][no-estimates-post]. After all, ["the ideal work planning process should always provide the development team with best thing to work on next, no more and no less. Further planning beyond this does not add value and is therefore waste."][scrum-ban].

## Conclusions

To conclude, here's just a few advices you need to bear in mind when performing
estimations:

* Estimations are just a tool and so they should be used when it is appropriate
and for what they are useful for.
* Estimations are made with a level of uncertainty and they have an error
margin. This should be understood by everyone and plans need to be built with
this in mind and adjusted as more is learned.
* Estimations should not be treated as commitments. Doing so may hinder
collaboration and will certainly trigger dysfunctional behaviours that will turn
them useless.
* Don't lose a lot of time estimating. You are not adding any value while
estimating and it may be an expensive endeavour.

And finally. Something you should do for every practice and tool you use in your
methodology. **Evaluate if it's working. Retrospect. Ask yourself what is the
value it is providing and try to improve!**.

## References

* http://thepeoplesscrum.tumblr.com/book/dontestimate
* http://www.slideshare.net/twallet/no-estimars
* http://programmers.stackexchange.com/questions/36810/whats-the-best-explanation-of-what-story-points-are
* http://www.youtube.com/watch?v=BAGain3T-xU
* http://www.mountaingoatsoftware.com/blog/tag/story-points
* http://zuill.us/WoodyZuill/2012/12/10/no-estimate-programming-series-intro-post/
* http://leansoftwareengineering.com/ksse/scrum-ban/

[book]: http://www.amazon.com/Agile-Estimating-Planning-Mike-Cohn/dp/0131479415
[weather]: http://martinfowler.com/bliki/YesterdaysWeather.html
[values]: http://agilemanifesto.org/
[no-estimates]: https://twitter.com/search?q=%23noestimates&src=tyah
[no-estimates-post]: http://zuill.us/WoodyZuill/2012/12/10/no-estimate-programming-series-intro-post/
[scrum-ban]: http://leansoftwareengineering.com/ksse/scrum-ban/
