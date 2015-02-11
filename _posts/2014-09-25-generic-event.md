---
layout: post
title: GenericEvent
author: bvssvni
---

Piston is a Rust game engine developed by the open source organization PistonDevelopers.

[http://www.piston.rs/](http://www.piston.rs/)

We are looking for contributors! [How to contribute to Piston](https://github.com/PistonDevelopers/piston/blob/master/CONTRIBUTING.md)

At the core of Piston is the event loop, which polls events from the window.
You can write your own back-end by implementing the `Window` trait.
There exists back-ends for SDL2 and GLFW already.
The library you need to write your own back-end is [here](https://github.com/PistonDevelopers/event).

This week I've been working on a new design for events, which uses a trick with Rust's `Any` and `TypeId`.

* Existing code should work as normal, but the new design is preferred because it is more generic
* The new design should be just as fast when compiled with "-O2"

### Old style

```Rust
use piston::Input;
use piston::input::Press;
...
match e {
    Input(Press(button)) => { ... }
    ...
}
```

### New style

```Rust
use piston::event::PressEvent;
...
e.press(|button| { ... });
```

### How does it work?

Each event is defined as a trait with a blanket impl for all `T: GenericEvent`.
When the method `press` is called,
it creates a unique id associated with the event trait by calling `TypeId::of::<Box<PressEvent>>()`.
This is then passed to `with_event` on `GenericEvent`.

```Rust
impl<T: GenericEvent> PressEvent for T {
    ...
    
    #[inline(always)]
    fn press(&self, f: |Button|) {
        let id = TypeId::of::<Box<PressEvent>>();
        self.with_event(id, |any| {
            match any.downcast_ref::<Button>() {
                Some(&button) => f(button),
                None => fail!("Expected `Button`")
            }
        });
    }
}
```

If the event supports press events, it will call the closure with an `&Any` argument.
This is then downcasted to the argument type of the event callback.
If the type does not match, it will fail with an error message.
The methods are inlined because Rust is smart enough to infer whether a `TypeId` matches.

Definition of `GenericEvent`:

```Rust
pub trait GenericEvent {
    fn from_event(event_trait_id: TypeId, args: &Any) -> Option<Self>;
    fn with_event(&self, event_trait_id: TypeId, f: |&Any|);
}
```

Since the event trait `PressEvent` and `GenericEvent` communicates through `&Any`,
we have to run unit tests to make sure they downcast to the right argument type.
The `assert_event_trait` function tests that a given instance of an event is implemented correctly.

```Rust
#[test]
fn test_input_event() {
    use input;
    use input::Keyboard;

    let ref e = PressEvent::from_button(Keyboard(input::keyboard::A)).unwrap();
    assert_event_trait::<InputEvent, Box<PressEvent>>(e);
    ...
}
```

### Why not just use traits?

When a function calls another function, you need to list all the generic constraints:

```Rust
fn foo<E: PressEvent + ReleaseEvent + UpdateEvent + ...>(e: &E) { ... }

fn bar<E: PressEvent + ReleaseEvent + UpdateEvent + ...>(e: &E) { foo(e) }
```

With this approach, if you want to add a `HeadTrackerEvent`, then you need to update all functions that call functions that call `.head_tracker(|head_data| { ... })`.

By using `GenericEvent`, we can do this:

```Rust
fn foo<E: GenericEvent>(e: &E) { ... }

fn bar<E: GenericEvent>(e: &E) { foo(e) }
```

The code won't break just because somebody wants a `HeadTrackerEvent`,
because all events are supported through `GenericEvent`.

* It solves the problem where native back-ends have custom types of events
* All code written on top of Piston will run on any platform
* New or custom hardware can be supported easily
* Container widgets do not need to be recompiled
* Window back-end can use generic wrappers, for example `enum HeadTrack<I: GenericEvent>`

### Can I impl GenericEvent for native event structures, like SDL2 and GLFW?

Yes.

You probably need to wrap it in a struct tuple, since you need to own either the trait
or the type to write an impl. For example `pub struct MyEvent(SDL2Event)`.
Then write a window back-end as `impl Window<MyEvent> for MyWindow`.

*Remember to inline the `GenericEvent` impl and write unit tests!!!*

### Further work

* Update [AI behavior trees](http://blog.piston.rs/2014/09/13/rust-event/) to use generic events
* Update [cam](https://github.com/PistonDevelopers/cam)
* Update [Conrod](https://github.com/pistondevelopers/conrod)
* Investigate event transformations
* Discuss future directions for input model and integration with [glutin](https://github.com/tomaka/glutin)
* Network events?

