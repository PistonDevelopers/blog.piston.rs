---
layout: post
title: "Solving Economic Inequality in MMOs"
author: bvssvni
---

Piston is a modular game engine written in Rust.
The project was started in 2014 when Rust was pre-1.0,
and during the last years we worked on various libraries and collaborated with other projects.

The Piston project is primarely about 2 things: Maintenance and research.
By sharing the maintenance burden between many developers,
we get more time to focus on the projects we like to work on.
This saves time for the people involved.

This post is about the research part. In particular: Economic inequality.

### Why do research at economic inequality?

Economic inequality is one of the world's biggest problems, besides climate change, poverty, deseases.
It is also interlinked with all the others.
One could say that economic inequality is partly causing all the other big problems we have.
Despite being such an obvious and huge problem, so far nobody have come up with a good solution. Why?

I do not know for certain, but after thinking a little, at least I can point to two major reasons:

1. Lack of an environment where you can test ideas
2. Difficult to measure effects precisely

So I was thinking: As games increases in size and complexity, maybe you could use them as a test environment?

Another motivation: Balancing economics in MMOs is hard. Perhaps we could do something about it?

### Reducing the problem to a simpler case: MMOs

I am not an economist, and I have no intention of touching the real world economy (yuck!).

So, I came up with a simple plan, which might work:

1. It turns out that [economic inequality in MMOs is worse than ever measured in the real world](https://en.wikipedia.org/wiki/Virtual_economy)
2. If we can solve the problem for MMOs, then it might be possible to solve it in the real world

I also had to come up with a concept for trying out this idea, and did a lot of background research.
If successful, this could be a mind blowing experience!

I won't go into the benefits about using this in MMOs, because the Mix-Economy project explains it in the [readme](https://github.com/PistonDevelopers/mix_economy/blob/master/README.md).

### The Mix-Algorithm: Combine Ideas!

Last year I came up with an algorithm that seems to work OK using a combination of:

1. Universal Basic Income
2. Progressive Negative Fortune Tax
3. Burning Money

Those are 3 ideas that each would crash an economy.
However, if you combine them in a clever way, they balance each other out and stabilizes the economy.

Since the algorithm uses normalized currency, you can mix it on top of an existing economic system.
This also makes it possible to turn it on and off to measure effects on the economy.

[Link to project](https://github.com/pistondevelopers/mix_economy)

So, how does it work?

Imagine as a factory that leads the pipes back the floor.
If people do not work hard enough on the factory floor, they die of the toxic smoke coming out of the pipes.
This is how a free market with 0% tax would work, because by random chance somebody ends up with too little money
and is excluded from the economy until somebody gives them something.
When nobody wants to hire somebody or provide for them, they die.

I think the smoke analogy fits because people die as a direct result of living inside a such system,
which becomes a systematically biased and impersonal as the system grows in size.
Because of this effect, no country has ever run successfully without some form of wealth distribution, and need to
plaster many systems all over to keep things running.

*Severe economic inequality is the result of a systematic bias of a system that does not scale*

The way I see it: It is not a problem with people, it is an SOFTWARE BUG.
Swap people with genetically identical ants, everyone equally smart and capable, and you get the same problem!

OK, back to the imaginary factory:

One day an engineer comes and visits the factory,
and rebuilds the pipes to let the smoke go up in the air. Wow!
It leads to a large improvement for the workers.
They are happier and more productive, which increases the efficiency of the factory.

Basically, that is what the mix-algorithm does.
It makes a simple regulatory design choice that makes the economy more efficient to rid of "waste".

Every economy needs an outlet, whether it is the poor, the rich, or a mix of both.
If you print money, somebody has to loose value.
However, if everyone looses value, then prices will increase and eat up the gain.

A way to solve this is as simple as it sounds horrifying terrible:
To control inflation, burn the money in the pocket of the rich!

Frightening, right?

However, when you consider a universal basic income,
it is not that simple, and perhaps not that horrifying: The rich get more because people have more money.
Since the money is not spent when the rich have less needs, you have to get rid of it! Burn it!

In the mix-algorithm, the money that is burned comes from the fact that you are inflating the currency by distributing through negative fortune tax.
You are not "taxing" the rich who "pays for" the poor.
The money is printed and distributed, and the outlet is money burned at the top.
In mere amounts of capital, the rich becomes far richer (up to 6x) when turning on the mix-algorithm.
So, it is not unfair to get rid of that money at all!

The money at the top are burned independently of the needs of the many.
This is how the factory lets the smoke up in the air, instead of the floor!

### Who "pays" for the economy?

It is the lack of money under the soft limit that charges the inflation.
Those at the bottom charges the rewards for those above, so it is the BOTTOM who carries the economy.

It is strangely intuitive, because common sense in the real world says that workers DO THE WORK.
Yet, now there is an algorithm that supports this argument! (I think that's cool!)

### Incentivize people to work

In a 0% tax environment (remember this is progressive negative fortune tax, not ordinary tax),
people work primarily to get the money they earn in the instant of the transaction.
Secondarily, they work for interest rates by putting the money in the bank.
However, this is not true for people at the bottom, who almost never have capital surplus.

Therefore, you end up in a situation where people at the bottom are less motivated to work.
With a universal basic income, this situation gets worse, because with all needs covered, people can just stay home.
How do we fix this problem?

The mix-algorithm does something clever: It gives you more money, the more money you have, up to a soft limit.
When people can earn just a little bit, they are more motivated because they gain on it over time.
The gains are larger at the bottom, and follows the square root function up to the soft limit.

### The Mix-Algorithm is not enough: You need a Gini solver

Economic inequality is measured by the [Gini coefficient](https://en.wikipedia.org/wiki/Gini_coefficient).

The problem with the mix-algorithm is that it incredibly hard to reason about it.
How do you set the tax rate when new players join, quit and change transaction patterns?

First I tried studying the long term behavior of Gini, which took 45 minutes of simulation per data point.
After two weeks, started thinking this would never get me anywhere.
I estimated that to gather enough data to find a reasonable approximation of the mathematical inverse,
it would require a super computer with 100_000 cores running for 1000 hours.

There had to be another way!

So, I came up with the idea of using a solver, who decides the tax rate directly, and near perfectly.

The Gini solver uses convergent binary search, because inequality might increase sometimes when increasing the tax.
When people have more money, they spend more, so rich people get a lot more money!
As a thumb rule, distributing money decreases inequality, but this is not always the case.

Here is a result of an early test with random transactions.
It shows that the algorithm works pretty well for various start fortunes (the money players get when joining):

![Mix Gini](https://cloud.githubusercontent.com/assets/1743862/19704277/629a3b42-9b07-11e6-9894-21ef6f98b436.png)

The important thing about this graph, is that you can set a Gini target for a broad range, that covers
the range of inequality measured in real world economics.

### Don't be optimistic

Even though this algorithm seems to work in early tests, I expect there are many tweakings needed before it can be used in a game.
Perhaps this is an overconstrained problem, or there are other effects that are not apparent under random transactions.

