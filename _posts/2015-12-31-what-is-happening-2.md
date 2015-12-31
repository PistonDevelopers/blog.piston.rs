---
layout: post
title: "What is happening 2"
author: bvssvni
---

Here is a new update on what is happening in the Piston project!

### [Image](https://github.com/PistonDevelopers/image)

This is a very popular library to read and write image formats.
All code is Rust, except the flate2 dependency.
It was started by ccgn and continued by nwin.

nwin cleaned up the API in the codec libraries.
Some bugs were fixed, thanks to bfops, lgvz and froydnj.

### [Conrod](https://github.com/PistonDevelopers/conrod/)

mitchmindtree landed new design. This was [an epic PR](https://github.com/PistonDevelopers/conrod/pull/626).

You can now update the UI outside the render event.
This means you can change the updates-per-frame to a lower rate for reduced CPU usage:

```Rust
// Poll events from the window.
for event in window.ups(60) {
    ui.handle_event(&event);
    event.update(|_| ui.set_widgets(set_widgets));
    event.draw_2d(|c, g| ui.draw_if_changed(c, g));
}
```

The `draw_if_changed` method renders the UI to the frame buffers.
By default this is 3 times since triple buffers are common, but you can change this with a setting.
If the UI is not animated, this saves energy usage significantly.

The benchmarks we have done so far indicates that the performance is approximately the same as previous design.
This could indicate that more use of hash functions is a tradeoff with widgets as primitives.
It could also mean that we are running into the limit of immediate design of piston-graphics.

We also discussed making Conrod backend agnostic on a higher level of graphics API, but have not come up with a good design yet.
One important thing is to not put severe restrictions on the type of widgets that can be made.

### [Piston-Graphics](https://github.com/pistondevelopers/graphics)

Piston-Graphics is a backend agnostic library for 2D graphics, using immediate design.
It was one of the first libraries in the Piston project and have been redesigned multiple times over the past 2 years.

Some default trait method were added to make it possible to do further optimization in the backend.
The docs were improved to make it easier to learn how to write a new backend.

One benefit of immedate design is the flexibility and easy to make new backends.
A downside is performance since it needs to move data to the GPU.

tomaka has suggested that a graphics API could be built on top of texture arrays and persistent mapped buffers.
This technique has the advantage of reducing draw calls, boosting performance on systems with high driver overhead.
Think of it as closer to O(1) compared to O(N) when rendering is limited by CPU usage.
[Here](http://www.gamedev.net/page/resources/_/technical/opengl/persistent-mapped-buffers-in-opengl-r3979) is an article if you are interested.

If somebody wants to start a new library to test a such design, open an issue [here](https://github.com/pistondevelopers/piston/issues).
This could be done over Glium, raw OpenGL or possibly Gfx.

### [Imageproc](https://github.com/pistondevelopers/imageproc)

This project started by theotherphil to separate image processing from image formats in the image library.

More people are now getting interested, and there has been some discussion about the overlap with computer vision.
Since this is a large field, should we start a new organization, like we did with [RustAudio](https://github.com/rustaudio/)?

If you are new to Rust but interested in image processing and computer vision, this is a project for you!

### [Turbine](https://github.com/PistonDevelopers/turbine)

This is a project with plans to make a 3D game engine with built-in editor.
It uses simple Entity-Component-System and one file for every entity for easier version control.
The goal is to connect together some of the tech we already have developed.

mitchmindtree and I tested and fixed some bugs when displaying Conrod UI on top of 3D.

### [Hematite](https://github.com/PistonDevelopers/hematite)

The project goal is to develop a Minecraft clone for client and server.

One the client side:

- limeburst donated the "hematite" name on crates.io :)
- Some bugs were fixed, and toqueteos landed world auto-finding, an epic PR that was broken for months.

On the server side:

- Progress on configuration
- Getting close to [single player support](https://github.com/PistonDevelopers/hematite_server/pull/106)!

### [TrueType](https://github.com/PistonDevelopers/truetype)

This is an attempt to get a pure Rust alternative to read and rasterize TrueType fonts.

I got it compiling by porting it from C, but expected it to never actually work.
To my delightful surprise PeterReid [made it](https://github.com/PistonDevelopers/truetype/pull/9)!
zummenix continued cleaning it up and improving it.

### [Mix-Economy](https://github.com/PistonDevelopers/mix_economy)

This is a new library to regulate virtual economies based on an experimental idea.

I got this idea from Alan Watts actually, that money could be thought of as a pure abstraction.
After all, the goal of money is to make people collaborate toward complex goals which the monetary system is blind to.
This means when trying to achieve new goals, for example sustainable development under near full automation,
the monetary system might need to be redesigned.
We live in a new time where the old ideas of money are challenged, which is exciting.

One big problem with conventional economics is the difficulty with reducing economic inequality.
Putting the individual interests aside, severe economic inequality reduces economic growth.
It can be thought of as less ability to reach the complex goals the system is supposed to do.

This is not just a problem with real economies, but also in virtual economies, such as MMOs.
Actually, virtual economies seems to have worse [Gini coefficient](https://en.wikipedia.org/wiki/Gini_coefficient) than any market in the real world!
Some companies have people working full time to balance economies so the gameplay attracts both hardcore and causual players.

I am not an economist, so I think the experts should deal with real world problems.
However, for virtual economies which are just for fun, there is no harm in testing out new ideas.

Every economy needs an outlet, which is either the poor, the rich, or a mix of both.
Some players have less time to play than others,
in the same way some people have less resources to contribute to an economy in the real world.
A game is often about something more than maximizing money,
so there is a complex, unknown goal that the algorithm is blind to.

One way to reduce inequality, is to offer everybody a basic income or a negative income tax.
However, in an MMO world where the algorithms have perfect information,
I figured out it would be easier to use a negative fortune tax charged by lack of money under a normalized limit.
This means, rich players are rewarded more than poor players, up to a soft limit where it starts to "burn in the pocket".

Normalizing the algorithm makes it easier to mix with an existing economy, to find a sweet spot that is fun to play.
An idea is to test this for [Project R](https://github.com/PistonDevelopers/project_r), possibly built on top of Turbine.
However, that's long down the road and very uncertain.

### [Pluto](https://github.com/PistonDevelopers/pluto)

The goal is to make a game competiton website using pure Rust server stack.

I experimented with a droplet on [pluto.rs](http://pluto.rs/). Seems broken at the moment :P

### Detecting O(N) performance behavior

Working on Turbine and testing the performance of Conrod, I thought of a way to detecting O(N) behavior without internal profiling.

For now, it is just a [spreadsheet](https://github.com/PistonDevelopers/turbine/blob/master/external-overhead-per-item.ods).

The method is simple: You run the program in benchmark mode with different settings,
and the formulas filter out initial overhead, initial overhead per item, and predicts linear accuracy.

It might be useful for cases when you do not have time to test for lots of different settings.

Happy new year!
