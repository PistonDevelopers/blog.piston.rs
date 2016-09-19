---
layout: post
title: "What is happening 3"
author: bvssvni
---

Another update on the Piston project!

Follow [@PistonDevelopers at Twitter](https://twitter.com/PistonDeveloper)!

This year we have focused on upgrading, improving docs and stabilizing libraries.
The overall architecture of the core is finished, and minor features are being added to flesh out the API.
A lot of libraries that can be used independently also saw improvements and are getting closer to stable.

The most complex upgrade in the history of Piston is done!
This was due to Gfx getting an overhaul to support next-gen APIs.
It took up most of my time this year, and the rest I spent developing a new scripting language (more about this later!).
This is why there have not been that many blog posts about the Piston project in general.

### [Image](https://github.com/pistondevelopers/image)

A library for encoding and decoding images.

In-tree JPEG decoder was replaced with [jpeg-decoder](https://crates.io/crates/jpeg-decoder).
This adds support for progressive JPEGs.

The image library has now Radiance HDR and ICO support.

Docs were improved, performance was improved, and lot of bugs was fixed!

[List of contributors (78)](https://github.com/PistonDevelopers/image/graphs/contributors)

### [Conrod](https://github.com/pistondevelopers/conrod)

A backend agnostic UI framework.

Widgets are now composed from a small set of primitive widgets.
This makes it easier to add custom backends for Conrod.

A lot of new widgets were added, a lot of bugs were fixed, and performance was improved!

Conrod has now a [guide](http://docs.piston.rs/conrod/conrod/guide/index.html) to help you get started.

For more information, see the [previous blog post](http://blog.piston.rs/2016/09/13/this-year-in-conrod/).

[List of contributors (51)](https://github.com/PistonDevelopers/conrod/graphs/contributors)

### [Imageproc](https://github.com/PistonDevelopers/imageproc)

A library for image processing.

Features for contrast, canny edge and other algorithms were added.

[List of contributors (8)](https://github.com/PistonDevelopers/imageproc/graphs/contributors)

### [Piston-Graphics](https://github.com/PistonDevelopers/graphics)

Performance was improved significantly (6x) in the Gfx and Glium backends.

RustType is now replacing FreeType as the library for font rendering.

Colors are now standardized to use sRGB, to get same behavior across platforms.

You can now override triangulation in the graphic backend for faster or high quality rendering.

[List of contributors (25)](https://github.com/PistonDevelopers/graphics/graphs/contributors)

### [Dyon](https://github.com/PistonDevelopers/dyon)

One unexpected surprise this year was the creation of a whole new dynamically typed language,
which like Rust provides safety without a garbage collector!

It takes the object model from Javascript, some nice syntax from Rust and Go,
replaces `null` with `opt` and `res`, and adds optional type system with support for ad-hoc types.
A new concept for dynamical scope called "current objects" does the job of globals, but better.
4D vectors, vector algebra and swizzling, HTML hex colors are built-in features.
For template programming it uses an optimized link structure (faster than arrays, uses less memory)
which is useful when generating web pages or code.
It has `?` like Rust, with tracing error messages on unwrap.
Syntax is standardized through Piston-Meta's meta language.

It is a language that focuses on "scripting experience" and borrows ideas from mathematics.
Indexed, packed loops, inference of range from body,
and something called "secrets" helps solving problems with less typing.
These features were tested for game prototyping, machine learning and linear algebra.

Current performance (interpreted) is in the same ballpark as Python, sometimes faster, sometimes slower.
Loading a "hello world" program takes 0.3 milliseconds (type checking and AST construction happens in parallel),
and dynamical modules makes interactive programming environments easy to set up.

The 0.8 release was epic, and the 0.9 release adds some important polish.
Like a baby, it took 9 months to get out of the womb, and it is now usable!
We will try to keep breaking changes minimal from now.

[List of contributors (3)](https://github.com/PistonDevelopers/dyon/graphs/contributors)

### Other projects

Some libraries that are not stable yet:

- [hematite](https://github.com/pistondevelopers/hematite_server) - Upgrades
- [hematite_server](https://github.com/PistonDevelopers/hematite_server) - Upgrades
- [mix_economy](https://github.com/pistondevelopers/mix_economy) - Added measurement of Gini coefficient
