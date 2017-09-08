---
layout: post
title: What is Happening 5
author: bvssvni
---

Follow [@PistonDeveloper at Twitter](https://twitter.com/PistonDeveloper)!

Shush! It has been 4 months since last blog post, how times fly by when you do not notice!

In this post I will give a summary on some projects, and then go into more details about some new research!

### [Piston-Tutorials](https://github.com/PistonDevelopers/Piston-Tutorials)

- [First 6 chapters of a new Sudoku tutorial](https://github.com/PistonDevelopers/Piston-Tutorials/tree/master/sudoku)

[List of contributors (32)](https://github.com/PistonDevelopers/Piston-Tutorials/graphs/contributors)

### [Conrod](https://github.com/PistonDevelopers/conrod/)

Conrod is an UI framework that makes it easy to program UIs in Rust.

- New triangles primitive widget
- Improved touch experience
- Lots of bug fixed

[List of contributors (66)](https://github.com/PistonDevelopers/conrod/graphs/contributors)

### [Image](https://github.com/pistondevelopers/image)

Image is a very popular image library with pure-Rust encoders and decoders.

- Improved BMP support
- Lots of bug fixed

[List of contributors (95)](https://github.com/PistonDevelopers/image/graphs/contributors)

### [Imageproc](https://github.com/PistonDevelopers/imageproc)

Imageproc is a library for image processing.

- Support seam carving for color images
- Sobel gradient for color images
- Improved performance
- More tests and documentation

[List of contributors (16)](https://github.com/PistonDevelopers/imageproc/graphs/contributors)

### [VisualRust](https://github.com/PistonDevelopers/visualrust)

- Fixed incremental build

[List of contributors (14)](https://github.com/PistonDevelopers/VisualRust/graphs/contributors)

### [Dyon](https://github.com/pistondevelopers/dyon)

Dyon is a scripting language with lifetime checker instead of garbage collection,
a similar object model to Javascript and lots of other features useful for gamedev.

Starting a new project to make a Dyon to Rust transpiler: https://github.com/pistondevelopers/dyon_to_rust

[List of contributors (4)](https://github.com/PistonDevelopers/dyon/graphs/contributors)

### [Piston-Music](https://github.com/PistonDevelopers/music)

- Support for playing sounds in addition to music
- Change volume on both music and sound

[List of contributors (3)](https://github.com/PistonDevelopers/music/graphs/contributors)

### [AdvancedResearch](https://advancedresearch.github.io/)

AdvancedResearch is a collection of projects that explore new ideas and concepts.
This is moved to its own organization to not spam PistonDevelopers with emails.

Here are some things that happened since last blog post:

- Started [prototyping](https://twitter.com/PistonDeveloper/status/903992563392290816) a hard coded car in Rust,
  to see whether homotopy maps can be used to generate geometry
- [Quickbacktrack 0.3 released](https://github.com/advancedresearch/advancedresearch.github.io/blob/master/blog/2017-08-09-quickbacktrack-0.3-released.md)
- [An outline of idea for defining "perfect intelligence"](https://github.com/advancedresearch/advancedresearch.github.io/blob/master/blog/2017-06-03-perfect-intelligence.md)
- A new algorithm discovered for probabilistic paths

Homotopy maps are functions normalized between 0 and 1 on input and generate points that are continuously connected with each other.
I found this idea very cool because you can use them for rendering directly without any extra knowledge.
The challenge is to find the right API design so you get the best from both worlds of graphical editors and programming.

At perfect intelligence, problems get solved at the information theoretic optimal performance.
I used the tools of [path semantics](https://github.com/advancedresearch/path_semantics) to reason about how
this might work, but have not formalized it yet (I lack the right conceptual tools!).
Surprisingly it is kind of like binary search, but instead of sorting the algorithm need to arrange sub-types.
You can [order a T-shirt](https://github.com/advancedresearch/path_semantics/blob/master/README.md) with the symbols of the first steps `∃f{}` (it is called a "universal existential path").

#### Probabilistic paths: A new discovery

![formula for probabilistic paths](http://i.imgur.com/deoP869.png)

Here is a thought experiment designed to help you understand what it is about:

1. Take a lot of monkeys
2. Make them type randomly on a keyboard
3. What is the chance one of them recreate Shakespeare (or Harry Potter)?

Using standard probability theory, it is easy to compute this chance,
even we never will get the opportunity to test it out in practice,
because it is very, very tiny.

![monkey typing on keyboard](https://upload.wikimedia.org/wikipedia/commons/f/f1/Monkey-typing.jpg)

In principle, there is a *correct probability* for any similar question we can ask,
no matter how complex the experiment is and how long time it takes to complete.

If you put the same monkeys to play Super Mario, what is the chance one of them will win?
We do not know that yet, because the code of Super Mario is much more complex than the first example.
Using standard formulas for probability distributions will not get you very far.
What we need a different way of thinking about probabilities that can be interpreted from programs.

A probabilistic path is a transform of the source code of e.g. Super Mario,
such that you can compute how likely a monkey is to win the game.

1. You need some function describing how much a given input is likely
3. You need some function describing what is a winning condition from the output

A huge breakthrough in path semantics happened by extending the theory to probabilities of finite sets.
Now I got a higer order path semantical function that solves similar problems to the one above.
It is called "probabilistic path" in the language of path semantics.

I tested it on very simple things, because it is very hard to use on complex algorithms.
One open problem is how describe in a meaningful way why the algorithm is allowed sum positive and negative numbers
while always ending up in the valid probability range between 0 and 1.
