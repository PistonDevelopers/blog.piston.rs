---
layout: post
title: "The road to high level libraries"
author: bvssvni
---

*Notice! This is a work in progress and not the final design!*

[Piston-Current](https://github.com/pistondevelopers/current) is a library I have been working on
with the goal of exploring "current objects" in library design.

High level libraries is the idea that Rust libraries can be written in a such way for game engines,
that makes them very easy to use and can be composed together without adding complexity.
I think the expression "high level" is awfully inaccurate, but I have not yet come up with a better word for it.
Unfortunately I don't have the libraries yet to show what I mean, but I hope to explain something about it in this post.

Piston-Current is designed with the intention that code based on it
is easy to translate from back-end specific to generic,
or building a high level library on top of it.
Therefore, Piston-Current is considered one of the "core" libraries of Piston.
Having a "core" does not mean all library need to depend on it.
For example, "image" and "vecmath" are completely independent.
Libraries that depends on one of the core libraries have a `piston2d-`, `piston3d-` or `piston-` prefix in the package name.

To get closer to my dream of high level libraries,
I have to come up with a design that is a little like globals or singletons, but not quite the same.
It is also not entirely safe, but not unsafe either if used properly.
Since this is new, and people are used to think `#$!!??#%@#` about globals,
it takes some time to get used to the idea, but don't worry, it is completely optional to use.
If you want to chat with me, I am frequently at the #rust-gamedev IRC channel under the nickname "bvssvni".

It all begins with "current objects"...

### Motivation of current objects

In game programming, there are many kinds of "current" values:

* The current window
* The current device
* The current sound driver
* The current player object

By setting these up as "current" values, you don't have to pass them around to each method.
For example, you can write code like this:

```Rust
e.press(|button| {
    if button == SHOOT {
        unsafe { current_gun() }.shoot(unsafe { current_player() }.aim);
    }
});
```

*(If you worry about the unsafe blocks, these will be explained later in the article)*

This makes it easier to decouple data from each other in the application structure.

The major motivation for this library is to have a convention that works across libraries.

### Problems Piston-Current solves

* Make current objects and `&RefCell<T>` work with generics
* Composable high level libraries
* Solving cases when the borrow checker is not powerful enough
* Convenient way of storing application structure
* Stepwise setup and rollback of application state
* Allows `fn () -> T` to compute something from application structure, which reduces code
* The balance between safety and convenience that fits with game programming

### `Get`, `Set` and `Action`

To make current objects and `&RefCell<T>` work with generics,
the object must use the traits `GetFrom`, `SetAt` and `ActOn`.
`Get`, `Set` and `Action` are auto implemented from these traits.

These 3 traits simplifies library design, and it gives a consistent behavior
across Piston libraries. `Set` allows one to use the builder pattern.
It makes is possible to write generic code,
where you can choose between `T`, `Current<T>` or `&RefCell<T>`.

To make it work with generics, use `where` clauses like this:

```Rust
impl<W, I, E: EventMap<I>>
Iterator<E>
for Events<W>
    where
        ShouldClose: GetFrom<W>,
        Size: GetFrom<W>,
        SwapBuffers: ActOn<W, ()>,
        PollEvent: ActOn<W, Option<I>>
```

This works with duplicate constraints, because the type of the constrained object is concrete.
For example, you can have `Size: GetFrom<U> + GetFrom<V>`.

`Current<T>` is just a wrapper type that auto implements these traits.
You can implement these traits for your own custom wrapper type, if you want to.

### What are high level libraries?

This is something I am excited about!

When I say "high level" I mean different from "normal" or "low level" because of the way the library is used,
not because it is further away or closer to the hardware.
It is because such libraries usually are designed for higher concepts that involves bigger pieces of game programming,
and they can be combined to build the features you want.
So "high level" means something like "high level game library for Piston" and does not refer to programming in general.

A high level library requires just a few lines of code to set up,
and adds functionality to the application without adding complexity.

For example, the `piston` repo is currently a high level library
that sets up a window and OpenGL context through the function `piston::start`.

Example:

```Rust
piston::start(
    piston::shader_version::OpenGL::_3_2,
    piston::WindowSettings {
        title: "Deform".to_string(),
        size: [300, 300],
        fullscreen: false,
        exit_on_esc: true,
        samples: 4,
    },
    || start()
);
```

Because the library sets up current objects,
there are no objects to keep track of and pass to other functions.
If the `piston` library adds some features, you don't have to change your function signatures to use the new features.

Calling `piston::set_title` lets you change the title of the current window.
If you forget to call `piston::start` the task will panic with a message about which current object is missing.

It also lets you start a new game loop with the following code:

```Rust
for e in piston::events() {
    ...
}
```

A game loop can run inside another game loop to display a different scene temporarily.
It makes it possible to divide the application logic into logical parts and run them in separate functions.
You can also call `piston::start` again to pop up a new window.

You can also set up application state in steps and roll back.
For example, in the game [Sea Birds' Breakfast](https://github.com/bvssvni/ld31) I loaded in the assets
in a separate function from setting up application state.
When the player won or lost, it was not necessary to reset the state manually,
or reload the assets.

```Rust
fn load_assets(f: ||) {
    // Load assets and set them up as current objects
    ...

    // Restart level if not quiting.
    while !piston::should_close() {
        // Loop infinite times. 
        background_music.play(-1).unwrap();
        
        f();
    }

    // Drop current guards
    ...
}
```

Such functions can be reused in a new project with a similar setup of game assets.

All high level libraries share these properties for their domain of functionality.
For example, you can develop a library for rendering landscapes,
and change the landscape for a scope, or roll back changes made to the current landscape,
or reuse the landscape in another project.

### Conditional compilation

The `piston` repo lets you add `features = ["include_gfx"]` in the Cargo.toml to set up a Gfx.
If you want to compile code conditionally based on whether `piston` compiles with Gfx,
then you can add a `[features]` tag to your Cargo.toml:

```
[features]
include_gfx = ["piston/include_gfx"]
```

Now, you can use `#[cfg(feature = "include_gfx")]` for Gfx specific features.

### Composing high level libraries

Currently the `piston` repo is the only high level library,
but the plan is to extend the Piston ecosystem with new high libraries over time.

A high level library can be used with another library if they share a high level dependency,
even those libraries were not designed intentionally to work together.
This is because current objects are unique per type,
so no information is required to glue one library to another.

Generic libraries that uses `GetFrom`, `SetAt` and `ActOn` are not considered high level.
*It is only high level if you don't have to keep track of any objects.*
Piston-Current makes it easy to write high level interface on top of generic libraries,
and nothing except using the traits is required to make it work.

Using generic libraries on top of Piston-Current does not mean you have to use current objects.
Current objects are usually used in application or a high level library,
but only exposed in a high level library if there is a need to work around the limitations of the API.

For example, the `piston` library exposes the current window:

```Rust
pub unsafe fn current_window() -> Current<WindowBackEnd> { Current::new() }
```

You can use `piston::set_title` to change the title of the current window without unsafe block:

```Rust
/// Sets title of the current window.
pub fn set_title(text: String) {
    unsafe {
        current_window().set_mut(window::Title(text));
    }
}
```

Most functions in a high level library looks like the one above, with a few lines of code.
The major part of the functionality can be built in safe generic code.

When a library has a high level dependency, it should import the function prefixed with `current_` instead of the type.
This is because the dependency then can be recompiled and change the type without breaking code.
For example, if you fork `piston` and recompile it to use GLFW, then there will be no breaks because the library does the following:

```Rust
use piston::current_window;
```

The same thing happens when you use the conventions from Piston-Current in an application.
Application logic looks similar to high level library logic.
It also makes it easier to split the application into modules by functionality,
for example to keep rendering code separate from update code.

Example:

The [sea birds module](https://github.com/bvssvni/ld31/blob/master/src/sea_birds.rs)
can easily be refactored into a library, and then used to add similar entity behavior in other 2D games.
This is independent of how these entities are rendered.
A similar thing can be done for 3D entities that navigates through a "current map".

One consequence of this is that users that follow Piston-Current conventions,
produces code that is suitable for high library design while working on their application.
The learning curve to write a high level library is pretty low.
This is important, because there will always be new people using Piston,
and they can find ways to contribute without expert knowledge of the internals.
At the same time, high level libraries what we want most,
so the "new, cool features" can be developed by anybody.
This will put pressure off maintainers to come up with new features,
and they can focus on more important stuff for maintenance, for example stability, API design and problem solving.

A library can start out as high level and be redesigned as a normal library,
change the back-end specific code,
or be rewritten to use a back-end agnostic abstraction, often without breaking code.

High level libraries are in general much easier and faster to develop,
and does not require as much knowledge about library design,
because the typical interface usually consists of global functions.

This makes it easier to test the real world usage of the library at early stage of development,
so library authors might start out with a high level library for a specific use case,
then write a better abstraction with multiple high level libraries on top of it.

Many people can work in parallel on different libraries without having to know about each other,
which is very important in a larger community.

A high level library does not need to use Piston-Current,
so if somebody writes a high level library on top of a generic library,
and need it to work in an environment with specific platform constraints,
they can change the implementation entirely without breaking application code.

A typical high level library consists mostly of global functions,
which means it could be written in C or a scripting language.
The concept does not apply only to Piston, but Piston-Current makes it such that
any generic library using Piston-Current is easy to turn into a high level one.
For example, if the Piston project develops a new library and you want a more convenient
one for your games, you can write a high level library on top of it.
Later, if you want to reimplement it in Lua, that is your choice.

Because games tend to be very implementation specific and hard to combine,
the prospect of high level libraries might make it easier to reuse bigger parts of one project in another.
This is done without depending on a common Entity/Component system.

### Safety vs convenience

Current objects can not be used without unsafe blocks,
which makes them a bit hard to accept for people who only want to write in safe code.
It is easy to avoid the pitfalls, but since the compiler can not enforce these rules,
there is room for human error.

It is possible to use scoped thread local variable and `&RefCell<T>` to achieve a similar functionality safely,
but it is not as convenient to use.
Generic libraries based on Piston-Current can be also used this way, without current objects.
The major advantage of current object over scoped thread local variables
is that switching from current objects to normal references is easier.

When calling a function that gives you a mutable reference to a current object in a closure,
it is possible to create two mutable references in scope by calling the same function inside the closure:

```Rust
render(Some(white), |c, g1| {
    render(Some(white), |c, g2| {
        // there are 2 mutable borrows to the gl back-end in scope
    });
});
```

This is the only unsafe case in high level library design that uses current objects,
so in order to solve this a `DANGER` struct must be added as first argument:

```Rust
render(unsafe { current::DANGER::new() }, Some(white), |c, g| {
    ...
}
```

In all such cases, the documentation of the function must include a notice about what the unsafety is about.
This way, if there is no unsafe block in the closure, the code is guaranteed to be safe.

The scoped thread local solution could avoid this, but will cause task panic in the same situation.
It is harder to reason about task panic with `&RefCell<T>` and scoped thread locals,
which might be critical for some types of applications.
Piston-Current lets you choose what is right for your case.

Using current objects in library design requires some knowledge about the danger of mutable aliased pointers.
Dereferencing, borrowing and then assigning with `Current` multiple times in same scope is considered unsafe.
For example, the following prints out "bar":

```Rust
let foo = unsafe { & *current_window() };
let bar = unsafe { &mut *current_window() };
bar.set_mut(Title("bar".to_string()));
let Title(text) = foo.get();
println!("{}", text);
```

The unsafety aspect of current objects is only when they are brough into scope as reference.
Calling a method on a current object is not unsafe, but still, an unsafe block is required,
because it is possible to reach unsafe cases in safe Rust code.
In order to meet the standard of safety, `Current::<T>` can not be constructed safely.

For example:

```Rust
unsafe { current_gun() }.shoot(unsafe { current_player() }.aim);
```

By making the functions `current_gun` and `current_player` safe, it would look like:

```Rust
current_gun().shoot(current_player().aim);
```

The code would still be safe, but because it then would theoretically possible to
use those functions in an unsafe way, the convention is to mark such functions with `unsafe`.

```Rust
unsafe fn current_window() -> Current<Window> { Current::new() }
```

When working on your own game project, it is up to you wether you want unsafe functions or not.
This is easy to add or remove.

In generic code, unsafe blocks are not required and is safe without unsafe blocks.

Current objects should usually not be used within `impls`, but only within global functions.
For `impls`, use generics instead.

### Conclusion

The rules for reasoning about safety are clear and easy enough to avoid most errors.
Unfortunately, these rules can not be enforced easily with the compiler or without runtime task panic.

It will be important to communicate the guidelines for high level library authors.
Normal libraries should not use current objects, but the Piston-Current design makes it possible
to add this with a few lines code either in the application or a high level library.
Because current objects are optional to use, and the design of Piston-Current has other benefits,
I think it is worth continuing this direction.

In my opinion current objects boosts producivity significantly in the way I want to use Piston.
It is in important for quick coding, but I find it easy enough to reason about the safety to not change back.
The code can be structured such that I can easily see where current objects are used.
For example, I don't use it inside `impls`, but only within global functions.

I made a scoped thread local version to compare the two designs.
Scoped thread locals combined with `&RefCell<T>` is harder to refactor to safer code.
Because of the convenience of changing back and forth, I am in favor of the current design.

Besides, since `&RefCell<T>` is supported by Piston-Current,
it can be combined with scoped thread locals in generic code.
This is great for people that want better safety guarantees and choose to not use current objects.
The scoped thread local version does not add new features or makes it safer to use.
However, just in case, the "scoped" branch will be kept until Rust 1.0 is released.

High level library design is something I wish to explore further,
but I don't know how to achieve this without Piston-Current yet.
Perhaps we can build the libraries first and then see how we can change them to make them safer?
I picture when we get some high level libraries, which are easy to build,
then we understand better what we need and can make better and safer abstractions.

In summary, I feel the Piston core has a clear path for the architectural problem it is supposed to solve,
and I don't expect big changes in the future, but I am looking forward to more high level libraries!
