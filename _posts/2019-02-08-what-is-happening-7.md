---
layout: post
title: What is Happening 7
author: bvssvni
---

Follow [@PistonDeveloper at Twitter](https://twitter.com/PistonDeveloper)!

This blog post is a brief summary of what happened the past 8 months in the Piston project.

### [Piston (Core)](https://github.com/pistondevelopers/piston)

- [Added methods to set automatic window closing](https://github.com/PistonDevelopers/piston/pull/1272)
- [Added `FileDrag` to input enum](https://github.com/PistonDevelopers/piston/pull/1269)
- [Use `Box<Error>` instead of `String` for errors](https://github.com/PistonDevelopers/piston/pull/1266)
- [Make `Size` use `f64`](https://github.com/PistonDevelopers/piston/pull/1255)

[List of contributors (72)](https://github.com/PistonDevelopers/piston/graphs/contributors)

### [Conrod](https://github.com/PistonDevelopers/conrod/)

Conrod is an UI framework that makes it easy to program UIs in Rust.

- [Redesigned by moving backends into their own crates](https://github.com/PistonDevelopers/conrod/pull/1223)
- Vulkano backend
- Various bug fixes

[List of contributors (89)](https://github.com/PistonDevelopers/conrod/graphs/contributors)

### [Image](https://github.com/pistondevelopers/image)

Image is a very popular image library with pure-Rust encoders and decoders.

The versions 0.20 and 0.21 were released with changes you can read [here](https://github.com/PistonDevelopers/image/blob/master/CHANGES.md).
(Too long to be listed here)

[List of contributors (134)](https://github.com/PistonDevelopers/image/graphs/contributors)

### [Imageproc](https://github.com/PistonDevelopers/imageproc)

Imageproc is a library for image processing.

- [Added Cross Correlation to Template Matching](https://github.com/PistonDevelopers/imageproc/pull/311)
- Display Images to View Output [#307](https://github.com/PistonDevelopers/imageproc/pull/307) and [#298](https://github.com/PistonDevelopers/imageproc/pull/298)
- Lots of improvements to codebase and features

[List of contributors (26)](https://github.com/PistonDevelopers/imageproc/graphs/contributors)

### [Dyon](https://github.com/pistondevelopers/dyon)

Dyon is a scripting language with lifetime checker instead of garbage collection,
a similar object model to Javascript and lots of other features useful for gamedev.

- [Added 4D matrices](https://github.com/PistonDevelopers/dyon/pull/556)
- In-loops added for easier communication between threads
- Improved docs
- Improved edge cases in syntax, type checker and runtime
- Faster parsing overall due to Piston-Meta upgrade
- Internal refactoring of instrinsics

[List of contributors (10)](https://github.com/PistonDevelopers/dyon/graphs/contributors)

### [Turbine](https://github.com/PistonDevelopers/turbine)

Turbine is a long term project to develop a game engine with built-in editor.

Currently some components are developed separately and tested, for later be used in the game engine.

- [3D scene rendering](https://github.com/PistonDevelopers/turbine/tree/master/scene3d)
- [Reactive design framework](https://github.com/PistonDevelopers/turbine/tree/master/reactive)

### [AdvancedResearch](https://advancedresearch.github.io/)

A part of Piston project is research, which moved to its own organization when unrelated to game development.

The reading sequences have been re-organized into two parts:

- [Reading sequences for learning path semantics](https://github.com/advancedresearch/path_semantics/blob/master/sequences.md)
- [Reading sequences for Artificial Intelligence and Safety Research](https://github.com/advancedresearch/path_semantics/blob/master/ai-sequences.md)

On the software side, the most exciting news are:

- [Linear solver - a new generic automated theorem prover](https://github.com/advancedresearch/linear_solver)
- [Room - an experiment to test the "Room Hypothesis" of common sense](https://github.com/advancedresearch/room)

The work on the Control Problem of AI and Zen Rationality resulted in [LOST (Local Optimal Decision Theory)](https://github.com/advancedresearch/path_semantics/blob/master/papers-wip/local-optimal-safety-theory.pdf).
The environment which this algorithm performs optimally falls outside classical Instrumental Rationality and might therefore be used to study Zen Rationality (extended reasoning about higher order goals).
This decision theory describes an analogue of a "panic" state for formal agents.
One mathematical property of this algorithm is that there is no expected utility loss by changing the decision theory,
which means that it might serve as a building block in [Operator Triggered Intermediate Decision Theories](https://github.com/advancedresearch/path_semantics/blob/master/papers-wip/operator-triggered-intermediate-decision-theories.pdf).
Such agents might have higher level of mathematical provable safety.
It was also proven that AI boxing is only decidable if the world is divided into finite states ([paper](https://github.com/advancedresearch/path_semantics/blob/master/papers-wip/safety-interruption-oracle-for-artificial-intelligence.pdf)).

Some other areas of progress:

- [Probabilistic sub-types in path semantics](https://github.com/advancedresearch/path_semantics/blob/master/papers-wip/probabilistic-sub-types.pdf)
- [Upper and Lower Bounds on Harvesting Intelligence From Nature (Through Extreme Observer Selection Effects)](https://github.com/advancedresearch/observer_selection_effects/blob/master/papers-wip/upper-and-lower-bounds-on-harvesting-intelligence-from-nature.pdf)

### Example of ongoing topic: Semantics of choices

Due to the many new papers and ideas floating around, I choose here to focus on a particular topic to give you a taste of what kind of research is going on:
The semantics of making a choice and how it might be grounded physically.

In one way a choice is very simple, in another way it is very deep and complex.

The basic problem of understanding what choices are, is that they are very easy to understand starting from the axiom of path semantics,
but they are very hard to understand as something grounded in physical reality.
According to path semantics, this grounding must exist in order for mathematics to make sense, but we don't know what it is.

A choice might be thought of as an action where one forgets something (or erases a resource),
and then "feed in" some new information that is required to obtain a new resource, in order to make new choices.
Ideas from a discussion with Adam Nemecek resulted in the paper about [Adversarial Paths](https://github.com/advancedresearch/path_semantics/blob/master/papers-wip/adversarial-paths.pdf).

Much about adversarial paths is not easy to understand, but it is very useful as a building block for other mathematical languages.
For example, this syntax can be used formalize [Adversarial Discrete Topology](https://github.com/advancedresearch/path_semantics/blob/master/papers-wip/adversarial-discrete-topology.pdf)
which might be used to e.g. understand game theory and agents in environments of unknown complexity.

Some important property that is known informally about choices, is the following:

*The major difference between Turing machines and informal theorem proving is that Turing machines eliminate choices.*

The word "decidable" often appears related to Turing machines, but it might be misguided as it refers to some enabling property.
On the other hand, "decidable" might be better understood as a disabling property in terms of eliminating choices.
It goes much deeper than just talking about algorithms that are not decidable vs algorithms that are decidable.

Much of the semantics that makes mathematical useful is based on some concept of choice in one way or another,
but any realization of an automatic procedure of such semantics requires a reduction of choice into a decidable computable process.
Hence, viewing choices as a form of resource,
there exists some kind of game (as in game theory) that is intrinsic to the nature of mathematics and computers,
which does not arise from the choice of an opponent, but from semantics of expressions in various mathematical languages.

The semantics of choices relative to other parts of mathematics is hard, but physical grounding of choices is even harder.

In trying to understand [extreme anthropic observer selection effects](https://github.com/advancedresearch/observer_selection_effects),
it is speculated that this constraint of eliminating choices is an emergent phenomena.
We might be living inside that a universe that has strange underlying physical laws that permits some violation of the principle "making a choice",
while all semantics that we use in practice (including computers and physical machines) is a result of viewing this
physical reality through a lense of an extreme observer effect.

While the foundations for choices is a mystery at this point,
it serves as a useful concept for arguing for uniqueness of mathematical emergent semantics such as the [unit interval on real numbers](https://github.com/advancedresearch/path_semantics/blob/master/papers-wip/natural-continuous-paths.pdf).
Unit intervals on real numbers is fundamental for fields such as geometry, where it appears in the domain of homotopy maps.
We want to know whether basic assumptions about the physical universe are unique in some sense,
such that there can not exist fundamentally different universes with some kind of "strange semantics of geometry".
Better understanding of this topic could bring us closer to understand how to think about physical laws where choices are *not* eliminated.

### Improved Organizational Security

After [detected suspicious behavior on Christimas Eve 2018](https://github.com/PistonDevelopers/piston/issues/1257),
I (bvssvni) reviewed security policies and made the following organizational security improvements:

- Reduced attack surface by separating write access to repositories from crates.io publishing permissions
- Restricted conditions of getting write access to repositories
- Restricted conditions of getting crates.io publishing permissions
- Recommended maintainers to add collaborators to specific repositories when needed

Since everying goes through PRs, most people do not need the write access.

When reviewing logs of all projects, I concluded that most contributions are made by a handful of people with specialist knowledge.
It seems that most work in recent years requires specialist knowledge or familiarity with specific codebases,
such that the amount of available work for people who want to contribute in general, but is not able to specialize, is shrinking.
Most people would be better off just by making PRs to specific projects,
and the likelihood that they need repeated access is shrinking over time.

Due to the shifting focus from general contributions toward specialism,
there is less need for increasing the amount of members with write access.
The majority of projects is now entering a maintenance and stabilization period.

### Important Organization Notice: Expecting Decreased Available Work Without Specialist Knowledge During Next 5 Years

The two major activities of the Piston project - maintenance and research - where research always required specialist knowledge,
plus the choice of using a modular architecture, means that it is likely that in the next 5 years, more and more low hanging fruit will be picked
and the amount of available work without specialist knowledge is expected to decrease.

Most active contributors are heavily invested in specific projects and new contributors are often interested in existing specific projects
or creating new ones that require specialist knowledge.
This means that we might benefit from making it easier to do specialist work over time,
and general organizational development is expected to have less benefit beyond serving this role.

Even version updates have been done in batches without the need for major changes,
which is done most efficiently using Eco's output by one person and by passing on this information to projects
that might use it for integrating updates with plans for new releases.
This is expected to be done approximately once a month or less in the future.

The need for architect software design is practically zero, as it seems people find it economically efficient
to combine their own set of preferred libraries to solve a specific problem,
instead of making major design changes to an existing one to solve a more general class of problems.
Most changes in design are focused on keeping backward compability, to keep the same set of problems solvable through modularity.

The need for bug fixing is very low despite the size of the project.
Excellent language design of Rust and good developer tools are likely to be blamed.

This means that the Piston project must either:

1. Focus more on special knowledge and projects
2. Increase ambitions for the project overall

The economically efficient trend seems to be 1) from observing what people chose to focus on in the past.
It is therefore unlikely that we will increase the ambition overall and instead focus more
on maintenance with specialist knowledge plus research.

Research has most potential of opening up new available work, but this is a slow going process.

The characteristics of the need of specialist knowledge is the following:

1. Less need for marketing the project or attracting new members
2. More need for training people interested in becoming specialists

The organization might therefore benefit from work on documenting and representing topics
that makes it easier to specialize for new people.
