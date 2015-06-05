---
layout: post
title: "Piston-Window 0.2"
author: bvssvni
---

[Piston-Window](https://github.com/pistondevelopers/piston_window) is a convenience window wrapper for the Piston game engine.
It is completely independent of the piston core and the other libraries,
so no Piston library requires you to use Piston-Window.

Piston-Window is designed for only one purpose: Convenience.

We have now released 0.2!

* Reexports everything you need to write 2D interactive applications
* `.draw_2d` for drawing 2D, and `.draw_3d` for drawing 3D
* Uses Gfx to work with 3D libraries in the Piston ecosystem
* A smart design to pass around the window and application state

```Rust
extern crate piston_window;
use piston_window::*;
fn main() {
    let window: PistonWindow = WindowSettings::new("Hello Piston!", [640, 480])
        .exit_on_esc(true).into();
    for e in window {
        e.draw_2d(|_c, g| {
            clear([0.5, 1.0, 0.5, 1.0], g);
        });
    }
}
```

If you want another convenience method, create a trait for it,
and then implement it for `PistonWindow`.

`PistonWindow` uses Glutin as window back-end by default,
but you can change to another back-end, for example SDL2 or GLFW by changing the type parameter:

```Rust
let window: PistonWindow<Sdl2Window> = WindowSettings::new("Hello Piston!", [640, 480])
    .exit_on_esc(true).into();
```

Games often follow a finite state machine logic.
A common way to solve this is using an `enum` and a loop for the different states.
This can be quite buggy, since you need to resolve the state for each event.

Instead, you could pass around one `PistonWindow` to different functions that represents the states.
This way you do not have to resolve the state, because it is part of the context.

`PistonWindow` implements `AdvancedWindow`, `Iterator`, `Window` and `GenericEvent`.
The iterator emits new objects of same type that wraps the event from the game loop.
You can swap the application state with `.app` method.
Nested game loops are supported, so you can have one inside another.

```Rust
for e in window {
    if let Some(button) = e.press_args() {
        let intro = e.app(start_intro());
        for e in intro {
            ...
        }
    }
}
```

Ideas or feedback? Open up an issue [here](https://github.com/pistondevelopers/piston_window/issues).
