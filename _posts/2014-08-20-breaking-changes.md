---
layout: post
title: Breaking Changes
author: bvssvni
---

Recently we have started reexporting libraries like rust-image and rust-graphics under the piston crate. This means less work to configure the Cargo.toml when starting a new project.

This allows us to make the libraries smaller without sacrificing ergonomics. When you type `cargo doc` it will generate the documentation for all libraries.

The window back-ends `glfw_game_window` and `sdl2_game_window` are not reexported, so you need to include one of these when writing a game.

We will attempt to reexport gfx-rs as well. The motivation is to push Rust game development to use a safer interface against graphics drivers. gfx-rs is not pure Rust yet, because the OpenGL back-end is included in the crate, but it will be moved out when Rust gets associated types.

Libraries should not depend on Piston if they are intended to be reexported. Stuff will move out of the piston crate and into smaller libraries, so new libraries can depend directly on those.

We are developing a common structure for input events to make widget and controller programming easier. The API in Piston will be unstable as we iterate on the design. For more information see [input](https://github.com/pistondevelopers/input).

If you find your code breaking and don't know how to fix it, ask in the [#rust-gamedev IRC channel](http://chat.mibbit.com/?server=irc.mozilla.org&channel=%23rust-gamedev) or take a look at [piston-examples](https://github.com/pistondevelopers/piston-examples).
