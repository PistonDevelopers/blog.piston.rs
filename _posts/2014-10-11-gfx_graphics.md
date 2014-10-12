---
layout: post
title: Introducing: gfx_graphics
author: bvssvni
---

[gfx_graphics](https://github.com/PistonDevelopers/gfx_graphics) is now ready for real world testing!
The repository contains example code.

### What is it?

gfx_graphics is a back-end for piston-graphics that uses Gfx for rendering.

This is how it looks like:

```Rust
let mut g2d = G2D::new(&mut device);
for e in EventIterator::new(&mut window, &event_settings) {
    use piston::event::RenderEvent;
    e.render(|_| {
        g2d.draw(&mut renderer, &frame, |c, g| {
            c.rgb(1.0, 1.0, 1.0).draw(g);
            c.rect(0.0, 0.0, 100.0, 100.0).rgb(1.0, 0.0, 0.0).draw(g);
            c.rect(50.0, 50.0, 100.0, 100.0).rgba(0.0, 1.0, 0.0, 0.3).draw(g);
            c.trans(100.0, 100.0).image(&image).draw(g);
        });

        device.submit(renderer.as_buffer());
        renderer.reset();
   });
}
```

![screenshot](https://raw.githubusercontent.com/PistonDevelopers/gfx_graphics/master/screenshot.png)

### About Gfx

[Gfx](https://github.com/gfx-rs/gfx-rs) is a safe, low level 3D API on top of OpenGL
that makes rendering on multi-threads possible by using `Renderer` objects.
It is designed to be fast at modern 3D APIs like Metal, Mantle, DirectX 12, next-gen OpenGL etc.

When you write a shader, you can bind the data with normal Rust structs,
and Gfx checks the parameter types for you.

The Piston project is working closely together with Gfx hackers
to make 2D and 3D as fast as possible for cross platform development in Rust.

### About piston-graphics

[piston-graphics](https://github.com/pistondevelopers/graphics) is a library for
rendering sprites and simple 2D shapes.
It uses immutable context types with compiler optimization to remove state overhead.
This was the library that started the Piston project.

You don't have to push and pop matrix transforms,
and you can fork a context as you wish.
The context is not bound to the graphics back-end,
so whenever you call `.draw(back_end)` it will render.
This means you can render to multiple back-ends at the same time, if you need to.

Writing your own back-end is easy, by implementing the `BackEnd` trait.
piston-graphics is designed to work with the simplest shaders possible,
and performs triangulation on the CPU.
Texture support is optional, so implementers can test it gradually.

piston-graphics is used in [Conrod](https://github.com/pistondevelopers/conrod),
an immediate mode graphics API.

One goal of the Piston project is using Rust's type system to write back-end agnostic libraries.
Nothing in the Piston core depends on anything besides the Rust standard library.
At the moment we lack a pure Rust font library,
so if you want to help out with Conrod, please see if you can do something about it!

### How fast is gfx_graphics?

Nobody knows exactly how fast it is yet. But it is fast!

Measuring performance in a graphics API is a hard problem,
because if you optimize for a single case, then it might run slower in total.
One way to find out is writing real games and applications,
and swap the back-end while running.

The only benchmark available is a game called [Sea Snake Escape](https://github.com/bvssvni/rust-snake).
It renders 65536 triangles on the sea snakes each frame, updated through a dynamic buffer.

On my computer, it runs at 60 fps with Gfx and 30 with OpenGL.
The reason Gfx is faster is most likely because it uses state caching for alpha blending.
For Gfx, this is 3932160 triangles per second.
Every single one is triangulated and transformed on the CPU.
Which means, for things like a GUI framework or simple 2D games, this is pretty good.

Still, there is room for optimization both in Gfx and piston-graphics.
