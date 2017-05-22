---
layout: post
title: "Announcing Dyon-Snippets"
author: bvssvni
---

The Piston project is pleased to announce [Dyon-Snippets](https://github.com/PistonDevelopers/dyon_snippets),
a place to share Dyon source code and discuss library design!

[Dyon](https://github.com/PistonDevelopers/dyon) is scripting programming language started in 2016 by myself (Sven Nilsen, bvssvni).

Dyon started as an experiment in a period I had a lot of time, while waiting for some important Gfx redesigns.
After a week of coding, I discovered that it was possible to use a lifetime checker (like Rust),
but without borrow semantics (unlike Rust) instead of a garbage collector.
Combined with copy-on-write for non-stack references, this creates a very limited memory model
but sufficient enough for many practical applications.
It is difficult to write object oriented code in Dyon, but it is very nice for iterating over arrays.

- Built-in support for 4D vectors and HTML colors
- Packed loop for mathematical index notation
- Secrets (more about this later)

The language uses dynamic loading of modules to organize code,
where you have full control over module dependencies.
It is very common to write a loader script, a program that runs before you run the actual program.

Here is an example:

```rust
fn main() {
    foo := unwrap(load("foo.dyon")) // Load `foo`.
    bar := unwrap(load(source: "bar.dyon", imports: [foo])) // Load `bar` with `foo` as dependency.
    call(bar, "main", []) // Run the function `main` on the `bar` module.
}
```

This allows a kind of programming where you easily control how stuff gets loaded,
e.g. check for for updates or refresh a module every Nth second.

I often use Dyon for problem solving, because the language has a feature called "secrets".
A secret is a hidden array of values associated with a `bool` or `f64`. The type is `sec[bool]` or `sec[f64]`.
The indexed loops in Dyon are integrated with secrets.

For example, you have a 2D array `v` and compute the maximum value:

```rust
m := max i, j {v[i][j]}
println(m) // Prints maximum value of `v`.
```

Dyon infers the range from the loop body. The code above is equivalent to:

```rust
m := max i [len(v)), j [len(v[i])) {v[i][j]}
```

This is a packed loop which is equivalent to:

```rust
m := max i [len(v)) {max j [len(v[i])) {v[i][j]}}
```

The notation `max i, j {v[i][j]}` is inspired by mathematics.
In mathematics and physics it is very common to use indices and custom loops.
It is easy to translate back and forth between equations and Dyon code,
and it helps you learn mathematics as well!

The type of `m` is `sec[f64]`. You can write the following:

```rust
where_max := where(max)
println(where_max) // Prints `[i, j]`.
```

This is how it works:

1. The inner `max` loop returns the maximum value with a secret `[j]`.
2. The outer `max` loop finds the maximum inner value and changes the secret to `[i, j]`.

A secret propagates from the left argument in binary operators.
This means you can combine `any` and `all` loops with `max` and `min`:

```rust
// Is there any list which maximum value is larger than 10?
m := any i {max j {v[i][j]} > 10}
if m {
    println(why(m)) // Prints `[i, j]`.
}
```

In problem solving this is very convenient, because many problems
can be thought of as formulating a question.
When you know the right question to ask, the answer is often easy to find.
