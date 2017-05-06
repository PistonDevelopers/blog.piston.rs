---
layout: post
title: "Announcing Binpool"
author: bvssvni
---

This blog post is about a new experimental uniform binary format for particle physics: [Piston-Binpool](https://github.com/PistonDevelopers/binpool)

### Background

Currently I am diving deep into [marine cloud brightening](http://blog.piston.rs/2017/02/02/cloud-albido-control-is-the-way-to-go/)
to understand how humanity can avoid some severe effects of climate change by using
some form of clean and reversible geoengineering.
If you did not know already, there have been some very worrisome observations in 2016-2017,
so I thought it was a good idea to educate myself a bit and perhaps get some input on what technological challenges there are.
The scientists and engineers working on this have been quite open and even sent me papers to read! Thanks guys.

While reading about this, which seems to require understanding of climate modelling (it is understandable if you take your time),
I figured doing some basic virtual physics experiments could help my brain along the way.
Nothing advanced yet, just freshing up some of the things I played around with over the years.

One thing I discovered, that it is not obvious what kind of analysis you want to do with a virtual experiment.
Instead of running simulations over and over, I would like to collect the raw data
and later run algorithms that computes some functional relationship.

The advantage of this method is that you can exploit some physical similarites between systems.
For example, it is commonly known that certain physical behaviors can transfer knowledge between a model and the full size system.
So, collecting data about some few particles colliding in a slow motion simulation,
can give you knowledge about particles colliding in high velocities and over a huge area.
This way you can save both time and energy, because you do not have to simulate every detail of a full system.

This is also interesting because I came across similar ideas when researching path semantics.
Studying physics could help design algorithms that predict approximate behaviors of other algorithms.
One long term goal is to develop a constraint solver that can assist with engineering tasks.

### Motivation and design

I could not find a standard format for storing and reading particle data in a such way
that you can reuse algorithms between projects, so I made my own experimental one.

Here is the format:

```
type format == 0 => end of stream
type format: u16, property id: u16
|- bytes == 0 => no more data, next property
|- bytes: u64, offset instance id: u64, data: [u8; bytes]
```

The first flag is a type format, which supports 10 common Rust numerical primitives,
with vector and matrix dimensions up to 80x80.

The second flag is a unique property id that identifies the property of the particle system.
This is also used to define time, e.g. an f32 scalar.

Next the format stores the number of bytes in the data block and an offset instance id,
such that you can read and write parts of the particle system (e.g. only the part that is dynamic or needed for analysis).

The number of particles in the block is derived from the number of bytes and the type format.
You can also specify a custom binary format, in which case a general purpose analysis tool can just skip the block.

For each frame in a simulation, you simply write out a set of properties,
and when reading back a frame you use a loop that breaks when all properties are set.
More information about the usage is in the [README](https://github.com/PistonDevelopers/binpool/blob/master/README.md) file.

### Future work

My hope is that I am able to reuse realgorithms across projects,
for example a command line tool that computes the energy or the number of collisions.
