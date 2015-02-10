---
layout: post
title: "Duck Typing in Piston"
author: PistonDevelopers
---

### Summary

- The Piston core now lives in the "piston" repo and the "high level experiment" is moved to [bvssvni/start_piston](https://github.com/bvssvni/start_piston).  
- Duck typing traits are moved to [Piston-Quack](https://github.com/pistondevelopers/quack).  
- [Piston-Current](https://github.com/pistondevelopers/current) is no longer part of the core libraries,
the safety issues with higher level APIs are solved, and [Best coding practices with current objects](https://github.com/PistonDevelopers/current/issues/15) got more strict safety guidelines.  

### Other news:

- [Glutin window back-end for Piston](https://github.com/pistondevelopers/glutin_window)
- [Glium graphics back-end for Piston](https://github.com/pistondevelopers/glium_graphics)
- [Image got animated GIF encoder](https://github.com/PistonDevelopers/image/pull/286)
- [Hematite got new life, planning Minecraft client and server](https://github.com/pistondevelopers/hematite)

Also, notice audio libraries are moved to a new organization: [RustAudio](https://github.com/rustaudio/).
mindtree is working on these libraries frequently and probably would love some contributions!

### Experimenting with abstractions for game development in Rust

Disclaimer: *The opinions in this article are not meant to be taken seriously,
this is to just to make is sound less boring!
Neither does it represent the official view of the 90 other Piston developers...*

[The Piston project](http://www.piston.rs/) is an active open source organization that started out
with [a back-end agnostic 2D graphics library](https://github.com/pistondevelopers/graphics) followed by
[an pure Rust image format and processing library](https://github.com/pistondevelopers/image) and
[a Minecraft-world renderer in Rust](https://github.com/pistondevelopers/hematite) and 
[an immediate mode UI for Rust](https://github.com/pistondevelopers/conrod) and
[a Visual Studio plugin for Rust](https://github.com/pistondevelopers/visualrust),
[an AI behavior tree library](https://github.com/PistonDevelopers/ai_behavior),
[a Wavefront OBJ format library](https://github.com/PistonDevelopers/wavefront_obj),
[a sprite animation library](https://github.com/PistonDevelopers/sprite),
[bindings for PhysFS](https://github.com/PistonDevelopers/physfs-rs),
[bindings for FreeType](https://github.com/PistonDevelopers/freetype-rs),
[a texture packer library](https://github.com/PistonDevelopers/texture_packer),
[a library for better-than-globals, called "current objects"](https://github.com/PistonDevelopers/current/) and a [few more](https://github.com/PistonDevelopers).

There is new library in town: [Piston-Quack](https://github.com/PistonDevelopers/quack).

The safety of Rust is a huge win when you are maintaining a large codebase,
but to keep it modular, you need good abstractions.

A good abstraction has the following properties:

* It needs to be understandable, so you can fix it if it goes wrong
* It should not force you to reinvent the wheel for every library

Piston-Quack builds on the idea of [duck typing](https://en.wikipedia.org/wiki/Duck_typing).

Since Rust does not have a garbage collector, you have to use `&RefCell<T>`, `Rc<RefCell<T>>` or unsafe code to share objects.
If you are coming from garbage collected language, this is something you should pay attention to.
The lack of a universal sharing mechanism put restrictions on the abstractions you can do.
For example, if you write a structure like this:

```Rust
pub struct Foo<T: Bar> {
    bar: T,
}
```

You figure out that `T` must be shared, and you can change it into:

```Rust
pub struct Foo<T: Bar> {
    bar: Rc<RefCell<T>>,
}
```

You can't use `Foo<Rc<RefCell<T>>` because `Rc<RefCell<T>>` does not implement `Bar`.  
This means when writing generic code, you have to think about whether the object is shared or not.  

In game development, sharing objects is so common that this becomes an obstacle.
For most cases, you don't care about this at all, only that the object is accessed by one thread at a time.
If you do it wrong and get an runtime error, then that's an acceptable cost compared to the time it takes to reason about it.
At the same time, you care a lot about performance, so putting a smart pointer everywhere is not an option.

Imagine game developers as the "worst customer" experience for language designers:
Alice, a game developer, goes in a programming language store owned by Bob.
Bob puts the latest and shiniest language on the desk, and shows the amazing features.
Then Bob asks Alice what she thinks about the language.
Alice starts to point out "I don't want this, I don't want that, this shouldn't be there...".
Then Bob asks what Alice wants, and Alice says "I want to not think about it".

Abstractions that requires thinking under composition or refactoring leads to friction in the game development process.
If you try to make it better, it is still not good enough, because *you want to not think about it at all*...
except when it matters, and suddenly you want *full control*.
So we need a language that lets you do both.

**This is why Rust is an excellent language for game development!**  
(comment: needs photo of everybody jumping with their arms in the air)

Rust comes with a powerful type system.
Not only can you make it good, you can make it *nice*...

Let us look at piece of code from Conrod:

```Rust
quack! {
    env: EnvelopeEditor['a, E, F]
    get:
        fn () -> Size [where E: EnvelopePoint] { Size(env.dim) }
        fn () -> DefaultWidgetState [where E: EnvelopePoint] {
            DefaultWidgetState(Widget::EnvelopeEditor(State::Normal))
        }
        fn () -> Id [where E: EnvelopePoint] { Id(env.ui_id) }
    set:
        fn (val: Color) [where E: EnvelopePoint] { env.maybe_color = Some(val) }
        fn (val: Callback<F>) [where E: EnvelopePoint, F: FnMut(&mut Vec<E>, usize) + 'a] {
            env.maybe_callback = Some(val.0)
        }
        fn (val: FrameColor) [where E: EnvelopePoint] { env.maybe_frame_color = Some(val.0) }
        fn (val: FrameWidth) [where E: EnvelopePoint] { env.maybe_frame = Some(val.0) }
        fn (val: LabelText<'a>) [where E: EnvelopePoint] { env.maybe_label = Some(val.0) }
        fn (val: LabelColor) [where E: EnvelopePoint] { env.maybe_label_color = Some(val.0) }
        fn (val: LabelFontSize) [where E: EnvelopePoint] { env.maybe_label_font_size = Some(val.0) }
        fn (val: Position) [where E: EnvelopePoint] { env.pos = val.0 }
        fn (val: Size) [where E: EnvelopePoint] { env.dim = val.0 }
    action:
}
```

When we implement traits based on these types, the trait gets "lifted" to `Rc<RefCell<T>>`.  

```Rust
/// A trait that indicates whether or not a widget
/// builder is positionable.
pub trait Positionable {
    fn point(self, pos: Point) -> Self;
    fn position(self, x: f64, y: f64) -> Self;
    fn down<C>(self, padding: f64, uic: &UiContext<C>) -> Self;
    fn up<C>(self, padding: f64, uic: &UiContext<C>) -> Self;
    fn left<C>(self, padding: f64, uic: &UiContext<C>) -> Self;
    fn right<C>(self, padding: f64, uic: &UiContext<C>) -> Self;
    fn down_from<C>(self, ui_id: UIID, padding: f64, uic: &UiContext<C>) -> Self;
    fn up_from<C>(self, ui_id: UIID, padding: f64, uic: &UiContext<C>) -> Self;
    fn left_from<C>(self, ui_id: UIID, padding: f64, uic: &UiContext<C>) -> Self;
    fn right_from<C>(self, ui_id: UIID, padding: f64, uic: &UiContext<C>) -> Self;
}

/// Position property.
#[derive(Copy)]
pub struct Position(pub [Scalar; 2]);

impl<T> Positionable for T
    where
        (Position, T): Pair<Data = Position, Object = T> + SetAt
{

    #[inline]
    fn point(self, pos: Point) -> Self {
        self.set(Position(pos))
    }
    ...
}
```

Since widgets have many properties in common, and lot of functionality depends on a few properties,
building traits on top of these properties reduces the number of lines of code.
If the API lacks something, you can make your own trait, because this is Rust.

* You don't have to worry about an object is shared or not, it will *just work*
* It improves type safety, for example by distinguishing between `Transform` and `ViewTransform`
* It can be upgraded to support new smart pointers in the future, such as garbage collected pointers
* It fits great with APIs that typically run into the expression problem

Builder methods comes for free with a consistent look-and-feel across libraries.

```Rust
let foo = Foo::new.set(Bar([w, h])).set(Baz([x, y]);
```

If you don't like this style, then you can auto implement traits on top with all the bells and whistles.

There is also no performance overhead, the compiler will optimize it for different use cases.
The `quack!` macro inlines everything, so large methods should be refactored out of it.

A couple things that might be improved in the future:

* The ugly `Pair<Data = Position, Object = T>` might disappear when Rust gets fully equality constraints
* The brackets `[]` for `where` clauses might disappear, they serve as work-around for macro parsing rules

Since Rust now refines impls by generic constraints, you can also implement the traits for other types.

Piston-Quack was originally based on [rust-modifier](https://github.com/reem/rust-modifier) by reem, but expanded to work
nicer with generics and the API requirements we had in Piston.
It replaced the old type-currying API in piston-graphics, and got rid of macros in Conrod.

Thanks to `quack!`, you can write your own window back-end for Piston in a few hundreds lines of code.
Piston has currently 5 window back-ends:

* [glfw_window](https://github.com/pistondevelopers/glfw_window)
* [glutin_window](https://github.com/pistondevelopers/glutin_window)
* [sdl2_window](https://github.com/pistondevelopers/sdl2_window)
* [tcod_window](https://github.com/indiv0/tcod_window)
* [window-less comes with piston-window](https://github.com/pistondevelopers/window)

It is currently being tested in piston-event_loop, piston-event, piston-window, piston-graphics and Conrod.

### "I've heard get and set methods are bad! Don't like it!"

If you follow discussions about library design, then you might come across arguments against accessors (get & set).
Piston-Quack is not about accessors, it is about *duck typing*.
It this context it makes sense to use get/set, because it replaces the direct access to struct members.

Accessors in other languages, for example C#, often just wrap a member variable.
Duck typing on the other hand, gives you a small "interface" for each accessor.
These interfaces can be used to build larger interfaces,
where you in C# would use inheritance.
(might be changed now, it has been a while since I used C#)

When each property is defined as a type, you can add more fields to it,
or even expand it to its own struct that fits better with the use case.
You can also add lifetimes and generic parameters.

The argument also is focusing on building methods instead of accessors.
When you don't want `get/set`, you can use `action`.
Here is an example from sdl2_window:

```Rust
action:
    fn (__: SwapBuffers) -> () [] { _obj.window.gl_swap_window() }
    fn (__: PollEvent) -> Option<Input> [] { _obj.poll_event() }
```

If you have an better idea of how to do this, please open an issue [here](https://github.com/pistondevelopers/quack/issues).

### A thank you to the Rust core team

During my entire experience with Rust, the core team has been supportive and helped on a lot of occations.
One of the most important contributions to the Piston project,
has been the collaboration to make Cargo support this kind of infrastructure.
My favourite feature in Cargo is the ability to override dependencies locally.
This goes to the top of my time-saving feature list that makes Rust suitable for coding-in-the-large.
Just a reminder of how a small feature can be extremely useful in a large project.

I want to thank Mozilla for leading this effort and wish you good luck toward 1.0.
