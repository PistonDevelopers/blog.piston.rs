---
layout: post
title: "The image library is now pure Rust"
author: bvssvni
---

Yesterday, oyvindln made a [PR to image-png](https://github.com/PistonDevelopers/image-png/pull/46) to replace `flate2` with `deflate`,
a DEFLATE and zlib encoder written in safe rust.
The PR was merged, and currently I am [ecoing](https://github.com/pistondevelopers/eco) the new version in the Piston ecosystem.

This means that the image library is now pure Rust! C is gone!

Just think of it: An image library supporting several popular image formats, written from scratch in Rust, over a period of 3 years!

To celebrate this moment, here is an overview of the people who made this possible (listing major components):

- [image](https://crates.io/crates/image), [83 contributors](https://github.com/PistonDevelopers/image/graphs/contributors)
- [png](https://crates.io/crates/png), [13 contributors](https://github.com/PistonDevelopers/image-png/graphs/contributors)
- [jpeg-decoder](https://crates.io/crates/jpeg-decoder), [7 contributors](https://github.com/kaksmet/jpeg-decoder/graphs/contributors)
- [gif](https://crates.io/crates/gif) [7 contributors](https://github.com/PistonDevelopers/image-gif/graphs/contributors)
- [lzw](https://crates.io/crates/lzw), [1 contributor](https://github.com/nwin/lzw/graphs/contributors)
- [deflate](https://crates.io/crates/deflate), [1 contributor](https://github.com/oyvindln/deflate-rs/graphs/contributors)
- [inflate](https://crates.io/crates/inflate), [2 contributors](https://github.com/PistonDevelopers/inflate/graphs/contributors)

*Notice! When library was moved, some people were left out, but hopefully they are in the list of authors.
There are also lot of people who contributed indirectly by testing and feedback. Thanks to you all!*

A special credit goes to these 2 fine people:

- ccgn, who started the project in 2014
- nwin, who has been the top maintainer since then

During the early start of the Piston project, in a storm of rustc breaking changes in 2014,
these two stood together as pillars, keeping the project afloat on top of a build script hacked together in Make and Bash,
before Cargo came and saved us all from drowning.

OK, perhaps not *that* dramatic, but those breaking changes were *intense*.

Btw, we welcome new contributors!
