---
layout: post
title: "Happy New Year from the Piston Project!"
author: bvssvni
---

Welcome 2017, may the earth make another beautiful ellipse around the sun! I wish you all Happy New Year!

I just published [Dyon 0.14.0](https://crates.io/crates/dyon), now with a new feature "link loop".

### Making code generation and templating easier

You can read more details about the design of the link loop [here](https://github.com/PistonDevelopers/dyon/issues/418).

Yesterday, I was trying to figure out how Dan Amelang's [Nile](https://github.com/damelang/nile) programming language works.
It is part of the effort of ViewPoint Research Institute (VPRI) to reinvent and revolutionize computing.
VPRI does a lot of exciting stuff, and currently I am looking into Nile to get some ideas about 2D graphics.

The Piston project already uses a lot of meta parsing, inspired by OMeta2 used to create Nile.
[Piston-Meta](https://github.com/pistondevelopers/meta) makes it easy to build custom text formats,
and the meta documents can also be shared and reused between projects.

For example, [Eco](https://github.com/pistondevelopers/eco) is a tool for analyzing breaking changes in Rust ecosystems,
and uses the same meta data converter code on two different types of formats: Cargo.toml and JSON dependency graph.
Meta parsing does not have to be a fancy technique, it can be useful in bread-and-butter programming too!

Dyon also uses the same meta language to [describe its language syntax](https://github.com/PistonDevelopers/dyon/blob/master/assets/syntax.txt).
The benefit of using this technique is that you get closer to the intrinsic complexity of the domain you are working in.
Piston-Meta is a very dense language, just useful enough be used for parsing, and nothing else.
Nile was designed to describe how 2D graphics works, but it has some very promising ideas: What about 3D, sound, etc.?

Here is an example of a Nile function/stream ([source](https://github.com/damelang/gezira/blob/master/nl/bezier.nl)):

```
CalculateBounds : Bezier >> (Point, Point)
    min =  999999 : Point
    max = -999999 : Point
    ∀ (A, B, C)
        if ¬(A.y = B.y ∧ B.y = C.y)
            min' = min ◁ A ◁ B ◁ C
            max' = max ▷ A ▷ B ▷ C
    >> (min, max)
```

Functions in Nile are quite different than in a typical programming language.
Every function can have input and output streams in addition to arguments.
Because these are processed in parallel, they do not deal with state the same way as typical functions do.
One Nile function corresponds to 3 C-functions, divided into "prologue", "body" and "epilogue".

I am trying to figure out how to translate this technique to Rust,
but also want to see how it is like to write a small compiler in Dyon.
So far I had to change the syntax a little bit, since Piston-Meta has no support for whitespace sensitive syntax.
If somebody wants to port OMeta2 to Rust, please do it!

Dyon uses a `link` type to generate code, which is a list that can only contain
`bool`, `f64` and `str`.
It is used for code generation and template programming.

For example, here is some Dyon code that prints out Rust code:

```rust
fn main() {
    println(gen_struct("Foo", [{name: "bar", type: "f64"}]))
}

gen_struct(name, fields: [{}]) = link {
    "pub struct "name" {\n"link i {
    "    pub "fields[i].name": "fields[i].type",\n"
    }"}\n\n"
}
```

In the example above we have a `link { ... }` block and a `link i { ... }` loop (new in 0.14).
The block creates a link, and the link loop joins all link items created inside the body.

In case you wonder what `link i { ... }` means, it is sugar for `link i len(fields) { ... }`.
Dyon looks inside the body of the loop and figures out which list that uses `i`.

The performance is pretty good. On my laptop this program runs in 25 seconds:

```rust
fn main() {
    x := link i 100_000_000 {i", "}
}
```

It creates `0, 1, 2, 3, ...` up to 100 millions.

### Other things happening

There is a lot of discussion going on in the [Conrod](https://github.com/PistonDevelopers/conrod/) project.
You are welcome to join!

People are now starting to work more the [image](https://github.com/PistonDevelopers/image/issues/419) library.
If you want to help, see if you can solve some of the [issues](https://github.com/PistonDevelopers/image).
oivindln got now a working deflate encoder in Rust, perhaps we can get rid of C entirely?
