---
layout: post
title: "Error handling in Dyon"
author: bvssvni
---

[Dyon](https://github.com/pistondevelopers/dyon)
is an experimental scripting language without garbage collector that I work on.
It borrows ideas from Rust, Javascript and Go.
Instead of a garbage collector, it uses a lifetime checker.

Example:

```rust
fn foo(a, b: 'a) { // 'b' outlives 'a'
    ...
}
```

The object model is similar to Javascript, but has no `null` value.

Version 0.3.0 added option and result for error handling.

In this article, I will explain the difference between `null` and `option/result`,
and how Dyon steals an idea from an accepted [RFC](https://github.com/rust-lang/rfcs/pull/243).
Rust does not support this feature yet, but I think it is brilliant and looking forward to it.

For an introduction to Dyon, see the two previous blog posts:

- [Dynamo](http://blog.piston.rs/2016/01/23/dynamo/) (this is the old name, now changed to Dyon)
- [Scripting without garbage collector](http://blog.piston.rs/2016/02/21/scripting-without-garbage-collector/)

### What is wrong with `null`?

Everything:

- The program suddenly crashes with a mysterious error message
- It can happen everywhere in the code base where an object is referenced
- Other language features can make it unneccessary

The inventor called it his billion-dollar mistake.

Disclaimer: I do not mention the name because it feels wrong to associate an awesome person with a tiny mistake.
However, if I am going to mention anyone, for completely unrelated reasons,
then [Jeff Rulifson](https://en.wikipedia.org/wiki/Jeff_Rulifson) is a hero of mine.

### What is right about `option/result`?

For the same reasons:

- When the program crashes, it tells you what happened
- It can only happen where these types are used, no need to suspect the whole code base
- Does exactly what it is supposed to do

Rust and Haskell are languages where you can code without `null`, which is great.

So, how does this work in Dyon?

### Result

Let us look at a simple program:

```rust
fn foo(a) -> {
    if a {
        return err("error!")
    } else {
        return ok("success!")
    }
}

fn main() {
    x := unwrap(foo(false))
    println(x)
}
```

This prints out "success!".

Change `foo(false)` to `foo(true)` and you get an error:

```
 --- ERROR --- 
error!
10,17:     x := unwrap(foo(true))
10,17:                 ^
```

What if we want to add a function that changes "success!" into "victory!"?

```rust
fn bar(a) -> {
    x := foo(a)?
    return ok(if x == "success!" { "victory!" } else { x })
}

fn main() {
    x := unwrap(bar(true))
    println(x)
}
```

When `foo(a)` returns an error, The `?` operator propagates the error, returning from the function.

When `foo(a)` returns `ok(_)`, it unwraps the value.

Today, you do the same in Rust by using the `try!` macro.
One problem is that it leaves no trace, making it hard to figure out where the error comes from.

An idea I got was to push a trace error message when using the `?` operator:

```rust
 --- ERROR --- 
error!
In function `bar`
10,10:     x := foo(a)?
10,10:          ^

19,17:     x := unwrap(bar(true))
19,17:                 ^
```

In Dyon all errors are of the same dynamic type, so adding this feature was not difficult.
I created a struct `Error` that wraps the error message, with an extra field for the trace:

```rust
pub struct Error {
    message: Variable,
    // Extra information to help debug error.
    // Stores error messages for all `?` operators.
    trace: Vec<String>,
}
```

The trace is hidden from the user, only visible when using `unwrap`.
When using `unwrap_err`, you only get the error message without the trace.

### Option

An object is `HashMap` under the hood, so you can remove and add fields by need.

However, there are situations this is bad:

- When adding wrong keys leads to hard-to-find bugs
- When expressing that a field is present, but has no value

Rust uses an `Option` type, with `None` and `Some(x)`.
Dyon uses `none()` and `some(x)`.

The `?` operator converts option into result.
Change the example into the following:

```rust
fn foo(a) -> {
    if a {
        return none()
    } else {
        return some("success!")
    }
}

fn bar(a) -> {
    x := foo(a)?
    return ok(if x == "success!" { "victory!" } else { x })
}

fn main() {
    x := unwrap(bar(true))
    println(x)
}
```

This given an error:

```
 --- ERROR --- 
Expected `some(_)`, found `none()`
In function `bar`
10,10:     x := foo(a)?
10,10:          ^

19,17:     x := unwrap(bar(true))
19,17:                 ^
```

Change `bar(true)` into `bar(false)` and it prints "victory!".

### External functions

Since Dyon uses dynamic modules, it is not possible to document it statically.
The only way to tell which functions are available is by inspecting it from the inside.
`functions()` gives you a sorted list of all functions with their lifetimes.

Example:

```rust
fn main() {
    fs := functions()
    println(fs[0])
}
```

This prints:

```
{name: "acos", type: "intrinsic", arguments: [{name: "arg0", lifetime: none()}], returns: true
```

There are 3 categories of functions:

- intrinsic (part of standard Dyon environment)
- external (custom Rust functions operating on the Dyon environment)
- loaded (imported and local functions)

Here is an example for writing a custom Rust function:

```rust
extern crate dyon;

use std::sync::Arc;
use dyon::*;

fn main() {
    let mut dyon_runtime = Runtime::new();
    let dyon_module = load_module().unwrap();
    if error(dyon_runtime.run(&dyon_module)) {
        return
    }
}

fn load_module() -> Option<Module> {
    let mut module = Module::new();
    module.add(Arc::new("say_hello".into()), dyon_say_hello, PreludeFunction {
        arg_constraints: vec![],
        returns: false
    });
    if error(load("source/test.rs", &mut module)) {
        None
    } else {
        Some(module)
    }
}

fn dyon_say_hello(_: &mut Runtime) -> Result<(), String> {
    println!("hi!");
    Ok(())
}
```

test.rs:

```rust
fn main() {
    say_hello()
}
```

### Some thoughts so far

I have a lot of fun working on Dyon.
It has a very simple syntax, so my brain does not have to process a lot to read the code.

First class functions are problematic because they require some type checking to be safe.
One idea is to limit them to arguments, without the ability to move or live inside objects.

Since you can not reference memory outside the stack,
it is very limited of how you can structure the code.
I wonder what happens when programs get larger...

I hope you enjoyed this article, and perhaps you might even try Dyon out a bit!
Do not recommend using it yet, because there will be plenty of breaking changes.
