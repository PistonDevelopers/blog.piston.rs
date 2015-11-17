---
layout: post
title: "What is happening"
author: bvssvni
---

It has been a long time since I wrote about what is currently happening in the Piston project.

So, here it is!

### [Conrod](https://github.com/pistondevelopers/conrod)

Conrod lets you program user interfaces in a way that makes it easier to change it on the fly.
Let's say you need a button, and instead of creating a button object and keep it somewhere,
you "render" the button to the user interface.

This technique is called "immediate user interface".
A benefit of this technique is all the properties of the interface can easily be interlinked,
avoiding the need to track the state of the user interface itself.
One can think of it as a way to give the programmer more explicit control.

A drawback of immediate user interface has been how to handle more complex widgets.
For example, a canvas containing other widgets, scrolling etc.

mitchmindtree found a way to model the internal relationships using a graph,
thereby solving the problem of how to do complex widgets.
He also figured out how to deal with unique ids that are dynamically generated,
making it simpler to reuse UI code between projects.

Currently, mitchmindtree is working on another redesign of Conrod.
The new design might allow to simplify the code further and improve the performance.
An idea is to instead of using Elmesque to construct the user interface primitives,
the primitives are widgets themselves.

One challenge is to solve issues such as moving between text boxes using tabs,
and how to use a UI on top of other content that handle user events, such as a 3D editor.
The new design will be more capable of handling such cases.

Thanks for all the people involved in Conrod! Without your feedback we would never get this far.

### [Imageproc](https://github.com/pistondevelopers/imageproc)

The Image library is a very popular one, but people ask about more features that not everybody else need.
theotherphil has started a new library Imageproc which focuses on image processing.

There will be some overlap between the image processing algorithms in the new library and the Image library,
but in the long term we might move stuff to the new library.

If you are interested in Rust and image processing, then this might be a perfect project to help out with!

### [OpenGEX](https://github.com/pistondevelopers/opengex)

[OpenGEX](http://www.opengex.org/) is a format for exchanging complex 3D scenes between game engines.
It has plugins for Blender, Maya and 3D Studio Max.

The idea is to handle 3D content in a way that is efficient for game engines,
without sacrificing the control for the artists.
One thing is designing an assets pipeline for a specific game,
but when you want to reuse the same pipeline for multiple types of games, it becomes much harder.
Binero and I discussed requirements of a Rust library for OpenGEX and what goals we should prioritize.

One idea was to use meta parsing (Piston-Meta) and extract data directly.
This allows greater control of the content in game engine.
All you need is the meta syntax of OpenDDL/OpenGEX and you are done.

The problem with meta parsing is that the entire file needs to be loaded into memory.
For large scenes this can be a problem, and this can be fixed by either extending Piston-Meta or write a custom parser.
After discussing this for a while, we figured out using the Nom parser combinator library might solve this.

Another problem with meta parsing is that meta data is not good for describing relations within the same structure.
There is a lot of stuff, such as animation interpolation, that goes into the OpenGEX standard.
To handle this you either need a library that works against any meta data or a proper Rust structure.
Yet, in many cases one might use a custom Rust structure in the game engine,
and we do not know what is best of transforming meta data or via Rust structures.
We decided that we had to try something get a better idea of the direction to go.
In any case, we have the meta data alternative to fall back to if Rust structures fail.

Currently, Binero is working on a structure for representing OpenGEX scenes.
I started a new project, Turbine, to get a picture of the requirements we need.

### [Turbine](https://github.com/pistondevelopers/turbine)

The goal of Turbine is to develop a 3D game engine with a built-in editor.
There are several motivations for working on it, mostly to push other projects forward.
My plan is to work on this in my spare time for the next 2 years, expecting to get something working,
but nothing impressive.
We want to test Conrod on top of 3D, and figure out the requirements for 3D assets pipeline.

At the core, we design the engine on top of a very simple Entity-Component-System.
The plan is to start out with a maximum of 10 000 entities, and then see later how we can expand it for larger worlds.

Each component is activated using a `u32` bit mask per entity.
All memory required for storing components is allocated upfront.
Each entity can activate any of the components available.
The benefits of this approach are:

- We know how much memory is required to run the engine
- We can easier predict how the engine performs in worst case
- It allows great flexibility when changing the behavior of entities

The physical state of entities will not store velocities, but instead extrapolate it from the previous update.
Since we use a fixed time step, the difference between the current and previous update equals
the linear motion for the next update.
In addition we need the initial state to reset entities when editing.

Here is how we initialize a world:

```Rust
World {
    mask: vec![Mask::empty(); ENTITY_COUNT],
    init: Physics::new(),
    prev: Physics::new(),
    current: Physics::new(),
    next: Physics::new(),
    aabb: vec![AABB::empty(); ENTITY_COUNT],
    name: vec![None; ENTITY_COUNT],
}
```

At each update, the physical states are swapped such that the previous state becomes the next state.

```Rust
pub fn swap_physics(&mut self) {
    use std::mem::swap;

    swap(&mut self.prev, &mut self.current);
    swap(&mut self.current, &mut self.next);
}
```

Johnathan Blow wrote an article about [concurrent world editing](http://the-witness.net/news/2011/12/engine-tech-concurrent-world-editing/) which we intend to test in Turbine.
Each entity has its own text file, which allows multiple people to work on various parts of the world at the same time.
When synchronizing the changes, a normal version control system can be used.
Piston-Meta is really great for this, because we can design a text format that works exactly the way we want.

### [Hematite](https://github.com/pistondevelopers/hematite)

The Hematite project has a goal of making a client/server Minecraft clone.

One problem is that Mmap is broken on Windows, a technique used for loading chunks from the disk as if it was in memory.
toqueteos is looking into alternatives.

Phrohdoh is looking into first person navigation and collision.

### [TrueType](https://github.com/PistonDevelopers/truetype)

One annoyance for users have been to set up FreeType.
In the long term, we would prefer a pure Rust solution for reading TrueType fonts and rasterize glyphs.

I am in the progress of porting stb_truetype from C to Rust, but I do not know whether it will work.
It compiles, but is far from usable.

There are some other people working on similar projects, so we are sharing ideas.
If you are interested in this, please contact me (bvssvni).

### [VisualRust](https://github.com/PistonDevelopers/VisualRust)

vosen is working on Cargo support and better syntax highlighting for Rust in Visual Studio.

cmr is working on toolchain managment.

If you are a C# expert with experience with Visual Studio plugins, we could use your help!

### [Pluto](https://github.com/PistonDevelopers/pluto)

Pluto is a game competition website using Rust in the whole server stack.

We changed name from Redox to Pluto to avoid confusion with the Redox OS project.

Haggus has made a cool design, but also thinking about new themes related to the new name.

A simple version with no dynamic functionality is working.
We plan to set up a server to test Pluto live.

Looking for contributors!

### Joystick events

Currently joystick events are only supported by the SDL2 window back-end.
Glutin and Glfw do not support such events yet.

We know that joystick events alone is not sufficient, because of controller feedback.
The plan is to start a cross platform library.

If you want to work on this, let me know (bvssvni).

### Collaboration across projects

I am collaborating with the Gfx and Glium projects, which are the safe 3D alternatives for Rust.

Gfx is used in several Piston projects.
Since it has back-end agnostic design, we do not plan to develop our own API.
However, we will try to make as much as possible code reusable for other 3D apis.
We are looking forward to DirectX 12 and Vulkan support.

Piston-Graphics uses the same draw state as Gfx, but is now supported by the Glium 2D back-end.
To make this work, I added support in Glium for separate blending equations.
tomaka has been great to work with and given useful feedback.
I plan to improve the integration with Glium which has some minor issues.
An idea is to add a convenient window wrapper like Piston-Window, but for Glium.

Servo, the research browser developed by Mozilla, is now using the Image library.

### Reliable benchmarking

Benchmarking game engines is not easy, because the wide variety of user experience,
hardware and operating systems.
Graphics drivers often contain hacks to fix bugs in popular games, so one method that is slowest on one machine
can be the fastest on another.

We would like to measure and predict how fast something will run in the released version.
This is important for project planning, which hardware to target and library design.

One thing we learned is that profiling in debug mode does not tell you much about how it behaves in release mode.
The performance can vary 3-6 times the optimized machine code.
If you benchmark in debug mode, you might draw wrong conclusions and end up optimizing the wrong thing.

Currently I am working on a reliable method that combines Piston's benchmark mode in the event loop,
with external measuring of how long the program takes to run.
The method is capable of estimating factors about O(N) algorithms from 10 numbers.

The point is to measure how performance vary with N overall,
even when there are lots of other things going on in the same program.
Initial overhead that scales with N gets filtered, together with background overhead.
The output is a number that tells how many microseconds is used to render and update and item,
and a percentage telling how close it is to O(N), where 100% is perfectly linear.

This means that one can modify the code in a few lines, run some commands and get a clue of how fast it runs.
It will be tested in Turbine, and if it works I will write a blog post about it.
