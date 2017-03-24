---
layout: post
title: "Proving Non-Existence of Monoid Symmetrci Paths"
author: bvssvni
---

This post is about a recent result from the research on [path semantics](https://github.com/bvssvni/path_semantics).
I will give a short introduction, since most people do not know what it is.

Path semantics is an extension of type theory that grounds meaning (semantics) using functions.

For example, a list of length 2 has the type `x: list` but also the sub type `x: [len] 2`.
It means that there exists some input `x` to a function `len` which returns `2`.
Another common notation for binary operators is `x: (> 2)` which means "x is bigger than 2".
You can combine them, e.g. `[len] (> 2)` means "a list with length larger than 2".

In dependently type theory, you can write `x: list 2`, by storing extra information in the type.
Path semantics deals with this information outside the type, using arbitray functions.
This forms a naturally abstract space, called "path semantical space", which describes how functions are identified by each other.

When a function is connected to another function using functions, it is called a "path".

A symmetric path has the following axiom:

```
f[g](g(x)) ?= g(f(x))
```

It says that for a function `f` and a property defined by `g`, there sometimes exists a function `f[g]`
that predicts the property of the output for function `f`.

Symmetric paths occur in many beautiful equations, for example in [De Morgan's laws](https://en.wikipedia.org/wiki/De_Morgan%27s_laws):

```
and[not] <=> or
or[not] <=> and
```

Here are some other examples:

```
concat(list, list) -> list
concat[len] <=> add
concat[sum] <=> add

even(nat) -> bool
odd(nat) -> bool
add[even] <=> eq
add[odd] <=> xor
eq[not] <=> xor
xor[not] <=> eq
```

This notation has algebraic rules, such that we can do things like this:

```
concat[len] <=> add
concat[len][even] <=> add[even]
add[even] <=> eq
concat[len][even] <=> eq
```

The holy grail of path semantics is to create an algorithm that finds paths automatically,
with as little input from humans as possible.

You might wonder why this is interesting:
Since paths is *everything you can predict* about functions,
it means that a such algorithm would be extremely useful for *everything that can be modelled as functions*.

So far I have algorithm that extracts both symmetric and asymmetric paths,
but it requires test data and human intuition to narrow down the search space.

Today I am getting a step closer to this dream, by stumbling over this golden nugget:

```
∃ i, j { ( g(x[i]) = g(x[j]) ) ∧ ( g(f(x[i])) ¬= g(f(x[j])) ) } => ¬∃ f[g]
```

It says that you can prove the non-existence of a symmetric path to functions in the monoid category
from as little as 2 objects.
In comparison, the current algorithms requires checking an infinite number of functions!

With some creativity, this can be used to prove things about other functions as well,
e.g. binary operators by binding one argument to a constant.

For example, I used this to prove:

```
¬∃ add((> 0))[is_prime]
```

It says there is no way to predict a prime by adding a positive number.

Proof:

1. The primes 2 is even and 3 is odd
2. When you add any positive number to both of them, at least one is even.
3. Because all prime numbers above 2 are odd, then at least one number is not a prime.

To explain the second part, we can write one argument as a set:

```
add({[even] true, [even] false}, [even] x) = {[even] (x = true), [even] (x = false)}
```

So, at least one element in the set is `[even] true`.

`2 + (> 0) = (> 2)`

All primes `(> 2)` are `[even] false`.

Therefore, at least one element is `[is_prime] false`.
Since you can add something to a prime to get another prime,
there exists two objects that are not equal by `g(f(x[i])) ¬= g(f(x[j]))`.

For example, `is_prime(add(2, 2)) ¬= is_prime(add(2, 3))`.

Therefore, there is no symmetric path for `add((> 0))[is_prime]`.
It is no prime prediction function to learn from addition.
