---
layout: post
title: "Beware upcoming Dyon v0.8 - It will be awesome!"
author: PistonDevelopers
---

[Dyon](https://github.com/pistondevelopers/dyon), a scripting language for Rust game engines, is getting close to v0.8.
For the v0.8 release, I want to polish the type checker a bit before publishing on crates.io, which might take some time.
This is why I write this article about the features beforehand, so you know what to expect from the next release.

### A summary of Dyon v0.7

Dyon does not use a garbage collector, but a lifetime checker.
There is no borrow checker like in Rust.
You annotate arguments with the lifetime of another argument that it outlives:

```rust
fn foo(a: 'b, b) { ... } // `a` outlives `b`
```

There is a `'return` lifetime to tell that an argument outlives the return value:

```rust
fn foo(a: 'return) -> { ... }
```

You can assign to `return` like it was a normal local variable, then let the function exit the scope.

```rust
fn foo() -> { return = 3 }
```

Dyon has a similar object model to Javascript. You declare variables with `:=`:

```rust
a := [1, 2, 3] // array
b := {x: 0, y: 0} // object
c := true // bool
d := 5.3 // float with 64 bit precision
```

A variable with same name shadows the previous one:

```rust
a := 0
a := false
println(a) // prints `false`
```

To mutate a variable, you use `=` which performs a check that the type is the same:

```rust
a := 0
a = false // ERROR: Expected assigning to a `bool`
```

Objects can override the type of a property with `:=`, which also inserts a key if it does not exists:

```rust
a := {}
a.x := 0 // Inserts `x: 0`
```

There is no `null`, but you can use `opt` or `res`, which are similar to `Option` and `Result` in Rust:

```rust
a := some(5.3)
b := ok(5.3)
```

You can propagate errors with `?`, which is similar to the `try!` macro in Rust:

```rust
fn foo() -> {
    x := bar()? // return error if something wrong happened in `bar`
    return ok(x > 3)
}

_ := unwrap(foo()) // Shows a trace if something went wrong.
```

Example of an error message:

```
 --- ERROR --- 
main (source/test.dyon)
unwrap

Something went wrong!
In function `foo` (source/test.dyon)
6,10:     x := bar()?
6,10:          ^

2,17:     _ := unwrap(foo())
2,17:                 ^
```

Dyon supports declaring functions the same way as in mathematics:

```rust
f(x) = x/(x-1)
pi() = 3.14
```

When mutating an argument, you put `mut` in front of it:

```rust
fn foo(mut a) { ... }
```

When calling a function that mutates an argument, you also put `mut` in front of the argument:

```rust
a := [1, 2, 3]
foo(mut a)
```

Named argument syntax based on snake case:

```rust
foo(bar: x) // named argument syntax
foo_bar(x)
```

If expression:

```rust
a := if b < c { 0 } else { 1 }
```

Traditional For loop:

```rust
for i := 0; i < 10; i += 1 { ... }
```

Short For loop:

```rust
for i 10 { ... }
for i [2, 10) { ... } // with offset
```

Mathematical loops, with unicode symbol alternatives:

```rust
x := sum i { list[i] }
x := ∑ i { list[i] }
y := any i { list[i] == 0 }
y := ∃ i { list[i] == 0 }
z := all i { list[i] > 0 }
z := ∀ i { list[i] > 0 }
min_val := min i { list[i] }
max_val := max i { list[i] }
```

Infinite loop, like in Rust:

```rust
loop { ... }
'name: loop { break 'name } // break out of loop `name`
```

Dynamic modules allows a flexible way of organizing code:

```rust
m := unwrap(load("script.dyon"))
call(m, "main", [])
```

Import modules to the prelude when loading a new module:

```rust
m := unwrap(load(source: "script.dyon", imports: [window, graphics]))
```

Optional type system, that complains when proven wrong:

```rust
fn could(list: []) -> f64
```

Go-like coroutines for multi-threading, using the `go` keyword:

```rust
t := go do_something(a, b)
res := unwrap(join(thread: t))
```

4D vectors:

```rust
a := (x, y) // same as `(x, y, 0, 0)`
b := #ff00aa // HTML hex color encoded as 4D vector
```

### Upcoming features in v0.8

dobkeratops had a brilliant idea: What if For loops inferred the range from the body, just like in mathematical notation?

```rust
// No need to type `len(list)`, because Dyon figures it out by looking at the code.
for i {
    foo(list[i])
}
```

This gave me another idea, which was to pack loops of same kind together:

```rust
// Set random weights in neural network.
for i, j, k {
    tensor[i][j][k] = random()
}
```

Then to make `∃`/`any` and `∀`/`all` loops work nicely `min`/`max`, I added a feature called "secrets" that
gives you the indices from any composition of these loops:

```rust
x := ∃ i { max j { list[i][j] } < 0.5 }
if x {
    pos := why(x) // `[i, j]`
    ...
}
```

A secret propagates from the left argument of binary operators.

There is a `why(bool) -> [any]` and `where(f64) -> [any]`.

You can add information to a secret with `explain_why(bool, any) -> bool` and `explain_where(f64, any) -> bool`:

```rust
x := any i { explain_why(did(person: person[i], said: "Doh!"), "Are you Homer Simpson?") }
if x {
    pos := why(x) // `[i, question]`
    ask(person: person[pos[0]], question: pos[1]) // Asks "Are you Homer Simpson?"
}
```

Dyon does not have globals, but uses a `~` to mark variables as "current object" for its scope:

```rust
fn main() {
    ~ settings := init_settings()
    foo()
}

// `foo` calls `bar` without knowing about `settings`
fn foo() {
    bar()
}

fn bar() ~ settings {
    // Can use `settings` as if it was a function argument.
}
```

To make code scale with size of project, but work nicely with dynamic modules, I added ad-hoc types:

```rust
// `Character` is not declared, but has an inner type `{}`.
fn new_character(name: str) -> Character {} {
    return {name: name}
}

fn greet_character(character: Character {}) {
    println("Hi " + character.name + "!")
    println("How are you doing?")
}

// Can pass values of the inner type.
greet_character({name: "Homer"})
```

This works for checking physical units:

```rust
fn main() {
    println(km(3) + m(5))
}

fn km(val: f64) -> km f64 { val }
fn m(val: f64) -> m f64 { val }
```

Addition of same ad-hoc types are allowed, but multiplication is not allowed for same ad-hoc types,
since this often changes the physical unit.

```
Type mismatch: Binary operator can not be used with `km f64` and `m f64`
2,21:     println(km(3) + m(5))
2,21:                     ^
```

Dyon has a `link` type, which stores `bool`, `f64` and `str` efficiently.
It also puts them together faster than using arrays.

```rust
a := link { "hi" 5 " "true" man show" }
println(a) // prints `hi5 true man show`
```

You can use `head` and `tail` to process a `link`:

```rust
fn main() {
    a := link {"hi" 5 " "true" man show"}
    loop {
        head := head(a)
        if head == none() { break }
        println(typeof(unwrap(head)))
        a = tail(a)
    }
    /*
    string
    number
    string
    boolean
    string
    */
}
```

A link inside a link gets flattened, which is useful when generating text documents.

For example, you could have a smart `html` santizing function that understood tags,
and then you could generate a web page any way you liked, as long as you use separate tokens for tags.
Perhaps a nice idea for a web framework?

```rust
fn main() {
    title := "Welcome to my website"
    data := get_data()
    loop {
        wait_for_request()
        respond(web_page(html(link {
            "<html>"
             "<body>"
              "<h1>"title"</h1>"
              menu(data)
              overview(data)
             "</body>"
            "</html>"
        })))
    }
}

// Sanitize html.
fn html(input: link) -> link { ... }
fn menu(data: Data{}) -> link { ... }
fn overview(data: Data {}) -> link { ... }
```

4D vectors now has a "unpacking" feature:

```rust
fn main() {
    v := (1, 2)
    // Call `foo` with 2 arguments.
    foo(xy v)
}

fn foo(x: f64, y: f64) { ... }
```

This works well with snake_case named argument syntax:

```rust
fn main() {
    v := (1, 2)
    foo(x_y: xy v)
}

fn foo_x_y(x: f64, y: f64) {}
```

You can also swizzle vector components:

```rust
v := (1, 2, 3)
u := (zy v,) // (3, 2)
```

And you can use `vec2`/`vec3`/`vec4` un-loops to unroll the loop and set rest of components to 0:

```
v := vec3 i i+1 // `(1, 2, 3, 0)`
```

There is a `s(v: vec4, ind: f64) -> f64` function that returns a component of a 4D vector.

I also added some macros to make embedding with Rust easier:

```rust
dyon_fn!{fn say_hello() { println!("hi!"); }}
```

You still need to register functions, because Dyon needs type + lifetime information.
I recommend checking out the [functions](https://github.com/PistonDevelopers/dyon/blob/master/examples/functions.rs)
example for more information.

That is all for now!
