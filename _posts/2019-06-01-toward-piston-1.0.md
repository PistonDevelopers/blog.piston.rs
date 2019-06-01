---
layout: post
title: Toward Piston 1.0
author: bvssvni
---

Follow [@PistonDeveloper at Twitter](https://twitter.com/PistonDeveloper)!

Update: There is a Rust gamedev meetup in Oslo coming up:

https://www.meetup.com/Rust-Oslo/events/261774885/

Sadly, I'm on vacation at that time, but you can meet people from the [Amethyst](https://amethyst.rs/) project.
Happy coding!

### About the Piston core

The Piston core is a set of libraries that defines a core abstraction for user inputs, window and event loop.  

If you take a look at the dependency graph in the [README](https://github.com/PistonDevelopers/piston/blob/master/README.md), it might look a bit scary:

![piston dependencies](https://raw.githubusercontent.com/PistonDevelopers/piston/master/Cargo.png)

In this post I will go through each library, explain what it does and describe the status of stability.

### [Serde](https://crates.io/crates/serde)

Serde is a library for serializing and deserializing Rust data structures.

The reason Serde is used by the Piston core, is to support:

- Sending events over network
- Store logs of events
- Record and play back player input for testing and debugging

Today, there is no functionality built into the core for these features.
The intention is to handle these cases in separate libraries, if necessary.
Piston's design philosphy is to move complexity away from the core,
since it is easier to keep the ecosystem stable when the core is stable.

By using Serde, these libraries gets much easier to write later.

Status: 1.0 (stable)

### [Piston-Float](https://crates.io/crates/piston-float)

This is a set of traits to abstract over `f64` and `f32` in Rust.

Piston-Float was designed to stabilize Piston's ecosystem in a time where we needed stability at the bottom of the dependency graph.

Before Piston-Float, we used numeric traits from Rust's standard library.
However, when Rust got close to 1.0, it was decided to move numeric traits into a separate library.
At the time, these trait were put together with lots of unstable stuff.
They were designed/changing to be more generic than for the primitive float types we needed.
The need for stability, plus restricted requirements, made it easier to create a new library.

Status: 1.0 (stable)

### [Piston-Viewport](https://crates.io/crates/piston-viewport)

A viewport is a rectangle inside the window sets up the coordinate system for a graphics API.

This library has a simple struct `Viewport` that stores viewport information.
It is used to pass viewport information from render events to 2D graphics backends.

Piston's 2D graphics library is designed to be decoupled from the core.
You can use it without the core, or you can use the core without the 2D graphics libraries.
By using a common viewport structure, it is easier to make these libraries work together.

Status: 1.0 (stable)

### [Bitflags](https://crates.io/crates/bitflags)

A Rust macro to generate structures that behave like a set of bitflags.

A set is a mathematical structure that supports operations like intersection, union, etc.
Normally, a C-programmer uses some tricks with bit masks to do work with bitflags.
However, in idiomatic Rust we like to keep things type safe, which is why this macro exists.

This macro is used by the structure for modifier keys in the core library for user input.

Status: 1.0 (stable)

### [PistonCore-Input](https://crates.io/crates/pistoncore-input)

This library is separated from the others in the core, since a lot of libraries
only need to know how to handle user input and
do not need to know about the window or event loop abstractions.

For example, when you press a keyboard key or a click a mouse button,
this information is stored in a structure that is defined in this library.

The same goes for gamepad or joystick events, touch events, render, update and idle events etc.

While designing the Piston core, we did some research about the problems of modelling user input.
What we found was the following:

- New hardware for user input is not widespread, because software does not support it
- Software does not support new hardware, because it is not widespread

The way we solve this problem, is to not restrict software to a particular model,
but to use traits when handling events.
This is why this library contains a lot of event traits, in order to make libraries work with new hardware.

The current design has been kept unchanged for a long time.
There are no unstable dependencies.
Without further redesign, it is ready to be stabilized.

There might be a few changes in the future, before stabilization.

Status: 0.25 (unstable)

### [Shader-Version](https://crates.io/crates/shader_version)

A helper library for detecting and picking compatible shaders.

Graphics APIs, such as OpenGL, Vulcan, Metal, DirectX etc. comes with shader languages.
The shader languages have their own version numbers, which works with various versions of the API.

To pick which shader to use can be tricky, so this library does this for you.

Today, only OpenGL/GLSL is supported by this library.
It is because OpenGL was the primary graphics API used at the time of design.
Besides, for newer graphics APIs, it is also common to use cross-compiling.

In the future, it is likely that the library becomes replaced by another in the core.
New graphics APIs come out all the time, so it will be difficult to stabilize this library.
Also, detecting and picking compatible shaders is not needed in the core, only graphics API versions.

Status: 0.3 (unstable)

### [PistonCore-Window](https://crates.io/crates/pistoncore-window)

This library contains the window abstraction.

The most common usage is window backends for GLFW, SDL2, Glutin etc.

However, a window in Piston might be more abstract and higher level than a cross platform window API.

It can be a custom designed backend for server-client logic,
or it can be a convenience wrapper, such as [Piston-Window](https://crates.io/crates/piston_window).

This library requires some changes before stabilization.

Status: 0.38 (unstable)

### [PistonCore-Event Loop](https://crates.io/crates/pistoncore-event_loop)

This library contains a game event loop.

The event loop supports fixed time step updates, extrapolated time for rendering, benchmark mode etc.

A common problem I see people have, is figuring out the right event loop settings for their game.
The defaults are designed for a First-Person-Shooter, which is pretty bad for many other kinds of games and applications.
In order to address this issue properly, some changes are needed.

Status 0.43 (unstable)

### [Piston](https://crates.io/crates/piston)

This library re-exports the core libraries. That's all.

Status: 0.43 (unstable)

### Notes

The Piston core has been in use for so long time, relatively unchanged, so it has not been much focus around it.

- Most of the interests experienced people have are in various modular libraries, not in the core
- Experienced people don't think too much about the core, which makes it hard to see through the eyes of beginners

For example, I noticed that some projects uses Piston-Window in libraries.
It is designed to be a convenience wrapper, not an abstraction.
The Piston core is more suitable for generic libraries.

I tried to put information about this in various places, but not sure if it is efficient enough.
This is a common problem because the project is so big:
Even if something is documented somewhere, it does not necessarily mean people will see it.
