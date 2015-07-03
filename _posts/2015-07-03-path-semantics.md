---
layout: post
title: "Path Semantics"
author: bvssvni
---

During the Piston project, I (bvssvni) have developed a [mathematical notation](https://github.com/pistondevelopers/math_notation)
for reasoning about API design, which I figured out could be used for theorem proving.
I have tried to understand the semantics and how it relates to dependently typed programming languages.

### What is path semantics?

In a dependently typed language, you have to encode the "proofs" into the type:

```
append(vec(X), vec(Y)) -> vec(X+Y)
```

With path semantics, one can connect two function through another function to express the same relationship:

```
append(vec, vec) -> vec
len(vec) -> usize

append [len] (usize, usize) -> usize
append [len] [:] (X, Y) -> X+Y
```

The path `[:]` treats members of a type as paths from the type to themselves.

```
0(u32) -> 0
1(u32) -> 1
2(u32) -> 2
...
```

### Formal definition of paths

A "path" is a function with one argument and one return value.

```
g(x) -> y
```

When a such function exists, there can *possibly* exist other functions,
that can be inferred logically or experimentally to have the equality:

```
f [g] (g(x)) = g(f(x))
```

A such connection might exist partially or probabilistically.

Paths can also be used asymmetrically:

```
f([g] x, [h] y) -> [i] z
```

Because paths can be used asymmetrically, they can be computed with as values.
This is possible to execute using a form of pattern matching stack machine,
where operations are not encoded in a single instruction, but in a sequence of instructions on the stack.
Verifying such programs is most likely to be undecidable.

In the pure version of path semantics where types and values emerges from the connections,
an axiom of equality can be applied (analog to the [univalence axiom](https://en.wikipedia.org/wiki/Homotopy_type_theory#Univalence_axiom)):

```
F0(X0), F1(X1), F0 == F1
------------------------
X0 == X1
```

This axiom states how equality is to be interpreted, for example `[g] x` is not equal to `x`,
because the path `g` changes the identity of the function that takes it as an argument.
In a pattern matching state machine, this causes the first match to fail so it looks for other
functions in the sub tree.

### Semantics of pure paths

In a pure path theory, there are no "functions" in the normal sense, but only "terms" or "atoms".
Each term can have an associated function taking constant arguments and returning a constant.
For example:

```
add [:] (1, 1) -> 2
```

Which is equal to

```
add([1] 1, [2] 2) -> [2] 2
```

To compute on numbers with pure paths, you would need one rule for every possible input.
This is not practical in many applications, so a pattern matching and variable binding over multiple inputs can be used:

```
add [:] (X, Y) -> X + Y
```

### Probabilistic paths

The probabilistic version of a path can be interpreted as
"any bit of information about an object can be used to infer some probabilistic
knowledge about the object itself and its relations to other objects".
Even if this connection can not be computed exactly, it can be used as a general inference tool.

Alice is a person with red hair, Bob is a person with blue hair:

```
alice(person) -> alice
bob(person) -> bob

red_hair(person) -> bool
red_hair([alice] alice) -> [true] true
red_hair([bob] bob) -> [false] false
```

If there is a person with red hair, the probability it is Alice is 50%:

```
probability(bool) -> f64

is(person, person) -> bool
is([red_hair] [true] true, [alice] alice) -> [probability] [0.5] 0.5
```

Or, written in short form:

```
alice: person
bob: person

red_hair(person) -> bool
[:] (alice) -> true
[:] (bob) -> false

probability(bool) -> f64

is(person, person) -> bool
([red_hair] [:] true, [:] alice) -> [probability] [:] 0.5
```

Notice that there is no way to "type check" this connection without having saying what "probability" means.
Path semantics does not tell how to model the world, but it implies there exists possibly a such connection.
If there is a such connection, then there are some constraints or internal consistenty to follow
relative to what is already said.

The key here is that when the function `is` returns `[probability] [:] 0.5`
this changes how the information gets computed later on.

We could define a logical `and` gate operating on both `bool` and for probability:

```
and(bool, bool) -> bool
[:] (true, true) -> true
[:] (_, _) -> false
[probability] [:] (X, Y) -> X * Y
```

One might use path semantics to model "fuzzy" relationships between objects.
The ability to verify such programs is probably an undecidable problem.

### Why is this an interesting research project for Piston?

One big problem we have in the Piston project is to design APIs that satisfy some critera.
These criteria change over time, and it is easy to forget what the original intentions were.
Using a formal language helps guiding the design, and the ability to express "fuzzy" relationships
might solve some tricky cases.

Otherwise, I think it is interesting to learn more about the internals of a such language should work.
Perhaps some of these ideas might leak into some practical libraries?

I also keep an eye open to the [OpenCog](http://opencog.org/) project,
which seeks to build a human level artificial intelligence.
This project uses a knowledge database called "AtomSpace".
It would be interesting if some of the patterns used in AtomSpace could be encoded with path semantics.
One project currently going on in OpenCog is to make game characters smarter, which is very interesting.
This project has done a lot of testing in this area, and perhaps the Piston project will explore some things in this direction.

This project has also been important for Piston-Meta, which might turn out to be very useful for other projects.
