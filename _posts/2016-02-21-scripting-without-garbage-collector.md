---
layout: post
title: "Scripting without garbage collector"
author: bvssvni
---

[Dyon](https://github.com/pistondevelopers/dyon) is a rusty dynamically typed programming language, previously called "Dynamo".

This article is to explain a bit more in depth about the language.
In particular, we will see how the lack of a garbage collector impacts how you think when programming.

### Lifetimes

Dyon uses a lifetime checker instead of a garbage collector.

For example, consider the following program:

```rust
fn foo(a, b) {
    a.x = b
}

fn main() {
    a := {x: [0]}
    b := [5]
    foo(a, b)
}
```

Dyon won't let you run it, because storing `b` inside `a` requires it to outlive `a`:

```
Function `foo` requires `b: 'a`
2,11:     a.x = b
2,11:           ^
```

One way to fix it is to write `clone(b)`, but you can also do as Dyon says:

```rust
fn foo(a, b: 'a) {
    a.x = b
}
```

Now there is another error:

```
`b` does not live long enough
4,12:     foo(a, b)
4,12:            ^
```

This is because `b` is declared after `a`:

```rust
fn main() {
    a := {x: [0]}
    b := [5] // `b` does not outlive `a`
    foo(a, b)
}
```

By moving it before `a`, the program works:

```rust
fn main() {
    b := [5] // `b` outlives `a`
    a := {x: [0]}
    foo(a, b)
}
```

The Rust borrow checker behaves similarly.  
If you like a challenge, try fix this code: [http://is.gd/IvQdqS](http://is.gd/IvQdqS)  
Solution: [http://is.gd/SX3z1J](http://is.gd/SX3z1J)  

### Data oriented design

Dyon shares similar semantics to Rust, but there are some differences:

- Can not reference memory outside the stack
- Variables are mutable
- An object can be shared among other objects

If you thought Rust was picky on how you designed your data structures,
then Dyon is even more so!

How the code looks can have a great impact on performance.
To optimize, you have to carefully declare variables and reference them.

For example, whenever you declare a variable and assign it a reference, you make a copy:

```rust
a := [1, 2, 3]
b := a // creates a copy of the list `a`
```

If you have a long list of 10 000 items, this can be very expensive.

More examples:

```rust
b := {list: a} // cheap, since `a` is referenced
c := a.list // expensive, since `a.list` is copied
d := a.list[0] // cheap, since only `1` is copied
```

This has the consequence that Dyon does not like deep trees by design.  
You are forced to process things in flat arrays as much as possible.  

With other words, **Dyon forces programmers to use data oriented design**.

### Named arguments syntax

Since Dyon likes flat arrays, calls to functions tends to have lots of arguments:

```rust
update(physics_settings, dynamic_objects, static_objects)
```

The problem is to remember the order.

To solve this, named argument syntax is supported through snake case function names:

```
update_physicsSettings_dynamicObjects_staticObjects(p, d, s) {
    ...
}

fn main() {
    ...
    update(physicsSettings: physics,
           dynamicObjects:  dynamic_objects,
           staticObjects:   static_objects)
    ...
}
```

This design has the benefits:

- Code does not break when argument name changes
- A way to resolve conflicts between function names
- Helps understanding the code

### Dynamic modules

The way you organize code in Dyon is very different from Rust:

- There is no `mod` keyword to declare a module
- There is no `use` keyword to import a module

One file of Dyon script does not know that another exists!

Instead, Dyon uses dynamic modules:

loader.dyon:

```rust
fn main() {
    graphics := load("graphics.dyon")
    window := load("window.dyon")
    my_program := load(source: "my_program.dyon", imports: [graphics, window])
    call(my_program, "main", [])
}
```

The imported modules become part of the prelude in the loaded module.

my_program.dyon:

```rust
fn main() {
    // No need to import anything.
    window := create_window(title: "hello world!", size: [512; 2])
    ...
}
```

This design has the benefits:

- Controlling the prelude is convenient when prototyping
- By writing a "loader" script, you do not need to recompile
- Gamedev often results in lots of smaller programs experimenting with different ideas
- Flexible control of where the script is stored
- Easy to reuse code even you did not plan to
- Swapping out backends is as easy to changing a module with another
- Refresh modules to make instant changes while running

### My thoughts so far

I think of Dyon as a domain specific language for gamedev:

- It is very convenient for testing ideas rapidly
- It does not need a garbage collector
- It forces you to think in a data oriented way

It is a language for prototyping, and can not match Rust on performance.
However, it is fast enough to be used in simple games.
When an idea grows in a large program, you can refactor it out into modules.
Later, you can rewrite parts of the code in Rust to make it faster.
This combination of flexibility and power is exactly what I am looking for.

I hope you will enjoy Dyon as much as I have!
