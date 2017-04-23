---
layout: post
title: "Existential Paths"
author: bvssvni
---

In this post I will report a big breakthrough in the ongoing research on [path semantics](https://github.com/bvssvni/path_semantics).

Link to paper: [Existential Paths](https://github.com/bvssvni/path_semantics/blob/master/papers-wip/existential-paths.pdf) (work in progress)

Since people asked me what the heck is path semantics,
I made an [Illustated History of Path Semantics with Stick Figures](https://github.com/bvssvni/path_semantics/blob/master/papers-wip/history-of-path-semantics-illustrated.pdf).

I also made an introduction to the notation in the last blog post about [non-existence of monoid symmetric paths](http://blog.piston.rs/2017/03/24/proving-non-existence-of-monoid-symmetric-paths/).
Since then I have figured out how to expand the technique to asymmetric paths.

Path semantics is the idea that some functions have secrets,
and those secrets can be expressed as functions,
which in turn can have other secrets,
and those secrets can also be expressed as functions etc.

This is very different from how most computer treats functions,
because they only use them for computation.

For example, if you define a function:

```rust
add(a, b) = a + b
```

In most programming languages, the computer only see this function as a computational procedure.
You give it some values as arguments and you get a value out.

However, in path semantics, you can write things like this:

```rust
add[even] <=> eq
```

This means that `add` hides secretly the behavior of `eq` that is unlocked using the `even` function.

When you have functions connected to other functions by functions,
it gets confusing, so you say `add` has a path to `eq` by `even`.

A path is a function in some mysterious space called "path semantical space" where all functions are connected.
When you explore and learn about this space,
you can "backproject" paths as patterns to tell something about large number of functions.

It also happens that these paths are predictors, so you can use them in machine learning.

Path semantics is actually closely related to type theory,
because you can use the notation to define sub types:

```rust
fn concat(a: [], b: []) -> [] { ... }
fn len(a: []) -> usize { ... }

// When concatenating two lists, you can think of it as sub types `[len] n` and `[len] m`
// returning a sub type `[len] n + m`.
concat(a: [len] n, b: [len] m) = [len] n + m

// Short form.
concat[len] <=> add
```

Creating a type checker for path semantics turned out to be a seemingly impossible problem...

...until today!

The new breakthrough is the discovery that there are *other kinds of paths* out there,
and one particular kind called "existential path" is closely connected with normal paths.

In fact, it is so closely connected, that it might be possible to make a type checker using them some day.

Simply put, if you have e.g. `a: [len] 0` then there exists a function `len'` such that:

```
a: [len] 0

0: [len'] true
```

This checks the sub type for all `a`. In this case, we know that there are lists of all lengths:

```
len'(x) = x >= 0

// Alternative notation.
0: (>= 0)
```

Since we have a concrete value `0`, we can just evaluate `len'(0)` and check if it returns `true`.

Actually, this `f'` function corresponds to the post-condition of `f`,
and a lot of smart people have already worked on this already.
This means we can reuse current tools for theorem proving!

You might notice that there should also be a function `len''` such that:

```
0: [len'] true

true: [len''] true
```

This is correct! It could go on forever, right? Luckily this leads to a repeating sequence:

```
// There are only two options for all functions.
f'' <=> {id?, true_1?}

id' <=> true_1
true_1' <=> id
```

So, after 2 steps, we can just hard code everything in the type checker there is to know about these functions.

This is a very recent breakthrough, but I am already thinking about how to set up the inference rules.

For example, if you type this:

```
a: [len] 0 & [len] (< 2)
```

Since these sub types are using the same function `len`, and `0` is a concrete value, one can reduce it to this:

```
0: (< 2)
0: [len'] true
```

