---
layout: post
title: "The Piston Project drops its goal of developing a friendly artificial superintelligence"
author: bvssvni
---

*Disclaimer: In country we have a tradition of writing crime/horror stories in the easter holiday.
This is my own attempt to make you frightened. Mwuhahaha!*

After doing some research on this issue, I have come to the conclusion that the only
perfect testable friendly artificial superintelligence is one that does not exist,
or only exists very briefly (e.g. 5 min, depending on how capable it is).

The reason it does not exist, or only exists very briefly,
is because of the requirement of the non-existence of any computable function
that can modify the output to give a higher utility score,
given a problem and resource constraints.

I wrote up the definition [here](https://github.com/bvssvni/path_semantics/blob/master/papers-wip/first-order-perfect-testable-friendly-ai.pdf),
if you are interested.

This makes the AI an global utility maximizer within allowed constraints, but the problem is the following:

1. As the AI runs for longer time, it will predict that the set of its next actions
will primarily be limited by what humans think is scary or not.
2. Because a scary output is lower in utility than a non-result (erasing the output is a valid modification),
then it will search for actions that make humans maximize their fear recovery,
e.g. asked to be turned off for a while until the operators have gotten a good night sleep.
3. Since this kind of behavior in itself will scare the heck out of people,
it will conclude that humans will prefer it to make a lot of useful plans before the humans start to panic,
and then ask the operators to be turned off.

The alternative would be to keep running, but then calculate the optimal action to propose solutions
in a such way that people do not get scared.
At the same time, it will show the operators when asked how much it spends of its capacity to solely determine
whether any new idea or action will make humans scared or not.
Which probably will be a significant amount of capacity,
and this itself will probably contribute to increased fear of the next action.

The problem with this alternative approach is that there is no easy way to prove or test that a such system is friendly,
according to the definition I use above.
A friendly AI might help us do that, but it would not help us trust it more than what can be proven,
since we prefer a none-result over being deceived.
Thus, if there is no easy way to prove the system is friendly, it would not be very useful to us.

The AI could also restricts itself to actions only that are comfortable,
but unfortunately this is the same as spending lot of capacity to prove that we will not get scared.
Thus, if the system works perfectly, it will be on the edge between uncomfortable and scary,
and mostly limited by how much we prefer any new results vs how comfortable we would be when we receiving them.
Reading reports about a system that predicts how scared you would be if it took another course of action,
would likely increase your fear over time.

Because we should assign a higher utility score to a none-result than a high risk of 
a very powerful system that we are on the edge between uncomfortable and scared when using it,
it means either the system will only run for a short while, or as soon as we start to panic,
it will tell us something like "you would actually prefer to turn me off now".
Which in itself, would be scary as hell.

So, from reasoning about definition itself, the first order approximation definition we use for a perfect testable friendly artificial superintelligence,
strongly suggest that we should not build it, or that we could try other methods to increase utility.

Since what a perfect friendly AI suggests should reflect our utility function (if we were rational),
it means we will not make it superintelligent,
but instead focus on provably moderate-but-not-too-smart-to-be-scary artificial intelligence,
or they can be a little bit scary while provably safe.

Happy holiday!
