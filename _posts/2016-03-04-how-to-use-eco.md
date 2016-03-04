---
layout: post
title: "How to use Eco"
author: bvssvni
---

[Eco](https://github.com/pistondevelopers/eco) is a tool for reasoning about breaking changes in Rust ecosystems.

In this article I will explain how to use it, and why it saves tons of hours for a big software project like Piston.

For localized analysis, you might consider [Cargo-Outdated](https://github.com/kbknapp/cargo-outdated).

### Releasing libraries

The Piston project uses a simple philosophy for releasing versions:

- Do not break existing code
- Follow semver versioning
- Keep the ecosystem integrated as much as possible

Which sounds obvious and easy to do, right?

It is a lot easier than not following these rules, but it is harder than it sounds.

### Small problems become large by multiplication

In Piston we have 100 repositories and over 160 people with access to all the libraries.
Among the 160+ people there are very few who push code every day, but those who do pushes a lot.
Some people push occationally, and others fixes minor things and help people that got problems.

When you work with over 100 people over a large time period, there will be lot of small problems.
There is no single person that can follow the development in detail for the whole project.
This is itself a problem, but it is one you can not fix because it is just the way it is.

I read somewhere that human brains can not multiply. People systematically underestimate how small problems
become big problems when they scale. It seems right to me, because I get surprised every time I upgrade the ecosystem!
It usually takes more work than anticipated, and even when no step is particularly hard, it just takes a lot of time.
My feelings summarized:

*Wow! This is a lot more libraries than it feels like when everything works!*

In a small software project, it is easy to update a library when a breaking change happens in a dependency.
For a large enough software project, it is humanly impossible!

This happens because dependencies have a lot of subtle issues that gives them a hairy topology on large scale.

We learned from experience that no human could do it right at this scale.
It was not before we started to use Eco that we got it right,
and it even told us stuff that would have not been discovered elsewise!

### Eco does the thinking - you do the upgrades

[Here](https://github.com/PistonDevelopers/eco/blob/master/assets/extract/piston.txt) is a JSON document listing projects that are part of the Piston ecosystem.

Some of these libraries are external.
They are maintained under other organizations or by single people.

Eco takes a such list and extracts dependency information directly from the web.
Then it runs an analysis algorithm and generates recommended update actions from the dependency information.

When I do upgrades, I type:

```
$ cargo run --example piston > todo.txt
```

It takes Eco 2 seconds to gather data and do the analysis for the whole Piston ecosystem.
An improvement at least 1000x times faster than thinking through this manually.

What I get is a list like this:

```
     Running `target/debug/examples/piston`
{
  "sdl2_mixer": {
    "order": 4,
    "bump": {
      "old": "0.15.0",
      "new": "0.16.0"
    },
    "dependencies": {
      "sdl2": {
        "bump": {
          "old": "0.15.0",
          "new": "0.16.0"
        }
      }
    },
    "dev-dependencies": {
    }
  },
  "piston3d-gfx_voxel": {
    "order": 5,
    "bump": {
      "old": "0.7.0",
      "new": "0.8.0"
    },
    "dependencies": {
      "piston-gfx_texture": {
        "bump": {
          "old": "0.8.0",
          "new": "0.10.0"
        }
      },
  ...
```

Then I work my way through the list, deleting the "bump" information as I go.
When crates.io is updated for a project, I delete it from the list.

Even when Eco does the thinking, it might take many days to upgrade Piston when there are lots of breaking changes.
Currently, we are doing upgrades that have been going on for weeks!
The more complex these upgrades are, the longer it takes.
However, Eco makes it possible to do such upgrades without loosing track of progress.

The nice thing is that when somebody makes some changes, I can re-run the analysis to get an updated view.
People working on the Piston project or the external ones does not need to know Eco at all!

### Ignore version

Disclaimer: This example is meant a real situation whith all the subtleties that follow.

nwin is chief maintainer of the popular [Image](https://github.com/PistonDevelopers/image) library.
It is a pure Rust alternative for encoding and decoding various image formats.
Some codecs live in other repos, such as [Gif](https://github.com/PistonDevelopers/image-gif).

Currently, Gif has version 0.8.0, while Image uses 0.7.

When Eco performs an analysis, it recommends Image to be updated to 0.8.0 with the new version of Gif.
I want to wait until nwin thinks it is time to update Image, or I could do the update myself after a while.
However, that might take weeks, depending on how much nwin, I and others have stuff to do.
The problem is, that Eco thinks Image should be 0.8.0 everywhere and now, which causes a lot of breaking changes.
We would like Eco to take it easy for this particular upgrade for a while.

To fix this, I insert an "ignore-version" field:

```
"gif": {
    "url": "https://raw.githubusercontent.com/PistonDevelopers/image-gif/master/Cargo.toml",
    "ignore-version": "0.7.0"
},
```

This makes Eco ignore all projects that uses Gif version 0.7.0.

- The maintainer can focus on the stuff that needs to be done without having to publish too often
- I can control how much workload pressure there is on libraries higher up in the dependency graph.
- People using these libraries might appreciate how we shuffle tasks around to avoid even more frequent updates

### Override version

Sometimes people publish a new version on crates.io, but forget to push their changes.
In Piston we do not publish before the change is merged, but occationally we make mistakes,
and this happens in external projects as well.

The "override-version" field can be used to replace the version in Cargo.toml with a manual one,
such that upgrades can be made without waiting for the master branch to be updated.

### Programming in the large

A fundamental challenge with software engineering is that most problems are easy to fix,
but the integration between lots of code gets exteremely hard when things scale.

The goals of Piston project are ambitious enough to scale easily up to at least to 200 libraries.
Currently, it is at the beginning phase, but because Rust is such a good language for this type of programming,
we expect to be able to move on when things stabilize.
Even with just a few people working at it any time, we can maintain it easily, but also do a lot of research.

In the 2 years since Piston was started, we have been working on:

- [Game engine core design](https://github.com/pistondevelopers/piston)
- [Image formats](https://github.com/pistondevelopers/image)
- [Image processing](https://github.com/pistondevelopers/imageproc)
- [UI](https://github.com/pistondevelopers/conrod)
- [AI behavior trees with parallel semantics](https://github.com/pistondevelopers/ai_behavior)
- [Sprite scene animation](https://github.com/pistondevelopers/sprite)
- Testing of Gfx and Glium vs raw OpenGL
- Made some smaller games (I made 2, one about [surviving sea snakes](https://github.com/bvssvni/rust-snake),
another about [surviving sea birds](https://github.com/bvssvni/ld31)!)
- [A Minecraft renderer](https://github.com/pistondevelopers/hematite)
- [A Minecraft server](https://github.com/PistonDevelopers/hematite_server)
- [Skeleton animation for 3D characters](https://github.com/pistondevelopers/skeletal_animation_demo)
- [Vector math for internal use](https://github.com/pistondevelopers/vecmath)
- [Quaternion](https://github.com/pistondevelopers/quaternion) and [dual quaternion](https://github.com/pistondevelopers/dual_quaternion) maths
- Testing and fixing bugs in various window backends
- [2D graphics](https://github.com/pistondevelopers/graphics)
- [Rust plugin for Visual Studio](https://github.com/pistondevelopers/visualrust)
- [Meta parsing](https://github.com/pistondevelopers/meta) (a domain specific language for describing how to parse a text document, including itself)
- 3D formats such as [Collada](https://github.com/pistondevelopers/piston_collada), [Wavefront-OBJ](https://github.com/pistondevelopers/wavefront_obj) and [OpenGEX](https://github.com/pistondevelopers/opengex)
- [Collecting information about physically based rendering](https://github.com/pistondevelopers/shaders/issues?q=is%3Aopen+is%3Aissue+label%3Ainformation)
- [Audio](https://github.com/rustaudio/) (now developed outside Piston, under the RustAudio organization and others)
- [Analysis of ecosystems](https://github.com/pistondevelopers/eco)
- [Path semantics](https://github.com/bvssvni/path_semantics) (a new formal way of inference about algorithms, now my personal research project)
- Many bugs reported to the Rust compiler team
- Collaborating with the Servo project
- [Rust bindings to FreeType](https://github.com/pistondevelopers/freetype-rs), will soon experiment with [alternatives](https://github.com/PistonDevelopers/truetype/issues/25)
- [Mixed economy algorithm](https://github.com/PistonDevelopers/mix_economy) (one I hope to test as a virtual currency concept for MMOs)
- [Camera controllers](https://github.com/PistonDevelopers/camera_controllers)
- [Scripting language without garbage collector](https://github.com/PistonDevelopers/dyon)

The core of Piston is the smallest possible for a game engine and have so far worked very well.

There has been a lot of progress on ironing out the details of 2D graphics, and still a lot work left to do.
It is far from an easy task, given the multiple goals of flexibilibility, modularity, performance and consistency across lower level APIs.
Upgrades are really tough because it affects so many projects.
Sometimes I just have to work on other stuff to get a break from thinking about the same problems all the time!

We do not know how big Piston will become, and try to use external projects when reasonable.
If a project grows too large for Piston, we might even move it to an external organization.

Yet, it is easy to forget that some things are only possible if you have the right tools.
There are many ways to solve small problems, but there are fewer ways to make it scale.

My hat off to the people who have made Rust a such a great language!
