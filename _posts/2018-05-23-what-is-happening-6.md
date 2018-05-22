---
layout: post
title: What is Happening 6
author: bvssvni
---

Follow [@PistonDeveloper at Twitter](https://twitter.com/PistonDeveloper)!

This blog post is a brief summary of what happened the past 7 months in the Piston project.

### [Piston (Core)](https://github.com/pistondevelopers/piston)

- [Added support for Xbox One controller D-pads](https://github.com/PistonDevelopers/piston/pull/1227)

[List of contributors (69)](https://github.com/PistonDevelopers/piston/graphs/contributors)

### [Piston-Examples](https://github.com/pistondevelopers/piston-examples)

I forgot to mention in last blog post that there is a [new example using the rs-tiled and piston together](https://github.com/PistonDevelopers/piston-examples/tree/master/tiled).

[List of contributors (34)](https://github.com/PistonDevelopers/piston-examples/graphs/contributors)

### [Conrod](https://github.com/PistonDevelopers/conrod/)

Conrod is an UI framework that makes it easy to program UIs in Rust.

- [New graph widget](https://github.com/PistonDevelopers/conrod/pull/1104)
- [Directly-only file navigator](https://github.com/PistonDevelopers/conrod/pull/1072)
- Improved tutorial
- Various fixes

[List of contributors (84)](https://github.com/PistonDevelopers/conrod/graphs/contributors)

### [Image](https://github.com/pistondevelopers/image)

Image is a very popular image library with pure-Rust encoders and decoders.

- [DXT encoding support](https://github.com/PistonDevelopers/image/pull/700)
- PNM support ([695](https://github.com/PistonDevelopers/image/pull/695) and [730](https://github.com/PistonDevelopers/image/pull/730))
- [Better TGA support](https://github.com/PistonDevelopers/image/pull/739)
- [Made Rayon optional dependency](https://github.com/PistonDevelopers/image/pull/724) (for environments without threads)
- Lots of API improvements
- Various bug fixes

Recently we have made a collective effort to reduce the burden of maintenance and got rid of the PR queue.
Thanks to people making PRs helping each other with reviews!
We started an image working group [where you can chat with people working on Rust image libraries](https://gitter.im/PistonDevelopers/rust-image-group).

[List of contributors (118)](https://github.com/PistonDevelopers/image/graphs/contributors)

### [Imageproc](https://github.com/PistonDevelopers/imageproc)

Imageproc is a library for image processing.

- [Support for integral images computed from multi-channel inputs](https://github.com/PistonDevelopers/imageproc/pull/260)
- [Optional normalization of SSE template matching](https://github.com/PistonDevelopers/imageproc/pull/259)
- [L2 distance transform](https://github.com/PistonDevelopers/imageproc/pull/247)
- Improved error messages

[List of contributors (21)](https://github.com/PistonDevelopers/imageproc/graphs/contributors)

### [Dyon](https://github.com/pistondevelopers/dyon)

Dyon is a scripting language with lifetime checker instead of garbage collection,
a similar object model to Javascript and lots of other features useful for gamedev.

[Dyon-Interactive](https://github.com/PistonDevelopers/dyon/tree/master/interactive) is now upgraded with many new features.

You can now install `dyongame` on your computer:

```
cargo install piston-dyon_interactive --example dyongame
```

To run, type `dyongame <file.dyon>`

Other news:

- New subreddit: [/r/dyon](https://www.reddit.com/r/dyon/)
- [Vim editor plugin for Dyon](https://github.com/thyrgle/vim-dyon)
- [Precompute data at top level with grab](https://github.com/PistonDevelopers/dyon/pull/511)
- [`Call` object for calling Dyon from Rust](https://github.com/PistonDevelopers/dyon/pull/501)
- [`args_os` intrinsic](https://github.com/PistonDevelopers/dyon/pull/500)
- [Support for `none` and `some` in Dyon data format](https://github.com/PistonDevelopers/dyon/pull/499)
- [In-types, a way to subscribe on inputs to loaded functions](https://github.com/PistonDevelopers/dyon/pull/494)
- [Insert and remove intrinsics for arrays](https://github.com/PistonDevelopers/dyon/pull/493)
- [`file` feature to turn off file access](https://github.com/PistonDevelopers/dyon/pull/491)

[List of contributors (7)](https://github.com/PistonDevelopers/dyon/graphs/contributors)

### [Turbine](https://github.com/PistonDevelopers/turbine)

Turbine is a long term project to develop a game engine with built-in editor.

Currently some components are developed separately and tested, for later be used in the game engine.

- [3D scene rendering](https://github.com/PistonDevelopers/turbine/tree/master/scene3d)
- [Reactive design framework](https://github.com/PistonDevelopers/turbine/tree/master/reactive)

### [AdvancedResearch](https://advancedresearch.github.io/)

A part of Piston project is research, which moved to its own organization when unrelated to game development.

I have been busy the past half year working on the control problem of artificial super-intelligent agents (ASI).
Basically, no one knows how yet how to solve this important research problem, but we believe we are getting closer.

There will only be a few bullet points here,
the rest you have to start exploring [here](https://github.com/advancedresearch/advancedresearch.github.io).

- A "Polite Zen Robot" (PZR) might be made safely extensible by using neutral judgements ([Link to paper](https://github.com/advancedresearch/path_semantics/blob/master/papers-wip/the-polite-zen-robot.pdf))
- Granular judgments indicates rational agents might cooperate if their future identity is uncertain ([Link to paper](https://github.com/advancedresearch/path_semantics/blob/master/papers-wip/ethical-reasoning-about-altruistic-competitions.pdf))
- [@forefinger](https://twitter.com/PistonDeveloper) suggested a mechanism for bounded utility functions in the human brain linked to the role of serotonin ([Link to repository](https://github.com/advancedresearch/ethicophysics/tree/master/askesis))
- [@forefinger](https://twitter.com/PistonDeveloper) has made a mathematical model of the cerebellum (!) ([Link to paper](https://github.com/advancedresearch/ethicophysics/blob/master/antipsychotic/serotonin.pdf))
- We will start combining Piston and AdvancedResearch to create simulated environments for agents (and use other Rust projects as well!) ([Link to repository](https://github.com/advancedresearch/challenges))
