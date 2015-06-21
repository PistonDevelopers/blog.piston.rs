---
layout: post
title: "Back-End Agnostic Design"
author: bvssvni
---

This blog post is about the general architecture of [Piston](https://github.com/pistondevelopers/piston).
I will try to give an overview of the things people have some trouble understanding,
based on things I have observed and stuff people have requested.
Please come talk to me (bvssvni) on #rust-gamedev (IRC) if you want more information,
or if you have ideas of how this can be represented more clearly.
There is a subreddit for Rust gamedev [here](http://www.reddit.com/r/rust_gamedev/) with link to the chat in the sidebar.

### Naming

Piston uses back-end agnostic design.
Back-ends for "graphics" ends with "_graphics" and back-ends for "window" ends with "_window".
For example:

- [opengl_graphics](https://github.com/pistondevelopers/opengl_graphics) => [graphics](https://github.com/pistondevelopers/graphics)
- [sdl2_window](https://github.com/pistondevelopers/sdl2_window) => [window](https://github.com/pistondevelopers/window)

### Motivation

The Piston project uses back-end agnostic design because:

* More choices when shipping a product
* Better for sharing source code
* Quickly fix problems by swapping a back-end and see if that works
* Easier to compare, debug and benchmark different design
* Composable with both cross and native platform programming

### Overview

To get started quickly with Piston, you can use:

- [piston_window](https://github.com/pistondevelopers/piston_window) (Convenience wrapper)

These are the main abstractions for window/events/graphics:

- [piston](https://github.com/pistondevelopers/piston) (core)
- [graphics](https://github.com/pistondevelopers/graphics) (2D graphics)
- [gfx](https://github.com/gfx-rs/gfx) (3D graphics, developed under gfx-rs organization)

The Piston project has many small libraries and projects running in parallel.
For a full list, see https://github.com/pistondevelopers/.

### Window

There is a `Window` trait for minimal features required for a game loop,
and there is `AdvancedWindow` which is intended for generic code.
This is provided by the [pistoncore-window](https://crates.io/crates/pistoncore-window) library.
Here is the `Window` trait:

```Rust
/// Required to use the event loop.
pub trait Window {
    /// The event type emitted by `poll_event`
    type Event;

    /// Returns true if window should close.
    fn should_close(&self) -> bool;

    /// Gets the size of the window in user coordinates.
    fn size(&self) -> Size;

    /// Swaps render buffers.
    fn swap_buffers(&mut self);

    /// Polls event from window.
    fn poll_event(&mut self) -> Option<Self::Event>;

    /// Gets draw size of the window.
    /// This is equal to the size of the frame buffer of the inner window,
    /// excluding the title bar and borders.
    fn draw_size(&self) -> Size;
}
```

### Events

In Piston, events are important to make libraries work together.

Events affects maintenance because of the way Rust's type system works.
In generic libraries, when you have a function `foo` calling `bar`, then `bar` will propagate constraints to `foo`.

```Rust
fn foo<T: Bar>(b: T) { bar(b); }
fn bar<T: Bar>(b: T) { ... }
```

So when you add a new constraint to `bar`, you need to update `foo`:

```Rust
fn foo<T: Bar>(b: T) { bar(b); } // <- error: the trait `Baz` is not implemented for the type `T`
fn bar<T: Bar + Baz>(b: T) { ... }
```

This problem scales with the size of the project.
To solve this problem in Piston, we use the `GenericEvent` trait, provided by [pistoncore-event](https://crates.io/crates/pistoncore-event):

```Rust
/// Implemented by all events
pub trait GenericEvent {
    /// The id of this event.
    fn event_id(&self) -> EventId;
    /// Calls closure with arguments
    fn with_args<'a, F, U>(&'a self, f: F) -> U
        where F: FnMut(&Any) -> U
    ;
    /// Converts from arguments to `Self`
    fn from_args(event_id: EventId, any: &Any, old_event: &Self) -> Option<Self>;
}
```

Normally, you do not use the `GenericEvent` trait directly, but some other trait built on top of it.
For example, `UpdateEvent` is implemented for all types implementing `GenericEvent`.
This means you only add the constraint `GenericEvent`, everywhere:

```Rust
// From the FirstPerson controller.
pub fn event<E>(&mut self, e: &E) where E: GenericEvent {
    use piston::event::{ MouseRelativeEvent, PressEvent, ReleaseEvent, UpdateEvent };
    ...
}
```

Events are not directly tied to the window abstraction,
because a common form of application logic is to transform the events.
Since generic libraries uses the `GenericEvent` trait, it makes them easier to use
when a higher level library performs the event transformation.
One such example is [AI behavior tree](https://github.com/pistondevelopers/ai_behavior) that
might change the update event to a shorter delta time according to the event logic.

### 2D Graphics

The project that started Piston was a [back-end agnostic 2D graphics library](https://github.com/pistondevelopers/graphics).
It meant that people could share 2D code across projects in the Rust community.
It is completely decoupled from the window and event abstraction.

Here is the `Graphics` trait:

```Rust
/// Implemented by all graphics back-ends.
pub trait Graphics {
    /// The texture type associated with the back-end.
    type Texture: ImageSize;

    /// Clears background with a color.
    fn clear_color(&mut self, color: [f32; 4]);

    /// Clears stencil buffer with a value.
    fn clear_stencil(&mut self, value: u8);

    /// Renders list of 2d triangles.
    fn tri_list<F>(&mut self, draw_state: &DrawState, color: &[f32; 4], f: F)
        where F: FnMut(&mut FnMut(&[f32]));

    /// Renders list of 2d triangles.
    ///
    /// A texture coordinate is assigned per vertex.
    /// The texture coordinates refers to the current texture.
    fn tri_list_uv<F>(
        &mut self,
        draw_state: &DrawState,
        color: &[f32; 4],
        texture: &<Self as Graphics>::Texture,
        f: F
    ) where F: FnMut(&mut FnMut(&[f32], &[f32]));
}
```

The `DrawState` is the that Gfx uses, such that there is no overhead and exposes fixed hardware pipelines features are available to higher level APIs.
This means that Piston can use the stencil buffer directly for things like clipping or simple blending effects.

The closure takes a closure, so the same state can be reused by the back-end between each call.

Here is how an image is rendered (taken from the graphics library):

```Rust
/// Draws the image.
pub fn draw<G>(
    &self,
    texture: &<G as Graphics>::Texture,
    draw_state: &DrawState,
    transform: Matrix2d,
    g: &mut G
)
    where G: Graphics
{
    use math::Scalar;

    let color = self.color.unwrap_or([1.0; 4]);
    let source_rectangle = self.source_rectangle.unwrap_or({
        let (w, h) = texture.get_size();
        [0, 0, w as i32, h as i32]
    });
    let rectangle = self.rectangle.unwrap_or([
        0.0,
        0.0,
        source_rectangle[2] as Scalar,
        source_rectangle[3] as Scalar
    ]);
    g.tri_list_uv(
        draw_state,
        &color,
        texture,
        |f| f(
            &triangulation::rect_tri_list_xy(transform, rectangle),
            &triangulation::rect_tri_list_uv(texture, source_rectangle)
        )
    );
}
```

Alternative designs are being considered, for example performing triangulation in the buffer of the back-end.

### Benchmarking

The event loop in Piston supports bench mark mode out of the box,
which is one of the few reliable ways to test the overall performance of a game engine.

This is activated by calling `.bench_mode(true)` on the event iterator.
When enabled, it will render and update without sleep and ignore input.

```Rust
for e in window.events().bench_mode(true) {
    ...
}
```

One of the benefits with back-end agnostic design, is that benchmark mode makes it easy to compare various APIs.

### Why use Piston?

Rust makes it easier and safer to program games, but there is more to game development than picking a programming language.

Piston is a project with a goal of building an game engine,
and this results in lot of interesting research projects and libraries that benefit the whole Rust community.
By providing a modular and open architecture, we hope you want to participate.

The people working on the Piston project have different goals, but share the cost of maintaining important libraries together.
Everyone gets write access to all the libraries, but we use PRs to make it easier to follow the changes.

[How to contribute](https://github.com/PistonDevelopers/piston/blob/master/CONTRIBUTING.md)
