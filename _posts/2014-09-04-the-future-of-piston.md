---
layout: post
title: Rust Gamedev News
author: bvssvni
---

With Piston and Gfx you can make 2D and 3D games in Rust, but what is the next step?

Let's talk business.

One thing for sure, the Rust gamedev community is not sitting and waiting
for AAA game studios to "consider" a safer language with equal performance as C++.

The PistonWorks initiative and its ecosystem is currently pioneering a foundation for a whole new industry.

* First, we are not going to limit ourselves to games, we are talking apps in general, on all platforms
* We are going to integrate the ecosystem in the open, on the web, for everyone and everybody

### Why Rust?

Rust allows you to create better quality software in shorter time
with far lower maintenance costs through cleaner abstractions
that will run on every thinkable platform in the future.

Not only this, but Rust's package manager Cargo makes it easy to download and use open source applications.

Cargo is almost at the level where a non-technical person can use it, but I think it will become even better.
I believe Rust will go mainstream in record time!

### Rust is easy to read

I did an experiment, by showing Rust to somebody I knew, that had relatively little programming experience.
It seems that things like types interference, traits and generics: People get it!

While Rust is a hard language to learn because of its size and complexity, it is fairly easy to read.
It does not depend on a nice IDE and it does not try to hide what is going on.

Hmm... which area do you want source to be readable? Ah! Free Software! Open source!
Which leads to my next thought: Rust is going to be a great language for open source.

OK, so *where* will it be a great language?

### Rust is going to be a great language for open distribution

Why? Because:

* Rust has not a reflective type system, which basically "means what you see in the source is what you get"
* Rust is less vulnerable against reverse engineering the source than languages running on VMs
* Rust is developed completely in the open
* The package manager is developed completely in the open

Which means if Rust applications are developed in the open, you have better full stack security.

* On one side, you want the source to be readable to verify that the computer will behave as promised
* On the other side, you want the binaries to not be reversible to source, to protect your system

Do you get this? It is *system* that is running you want to protect, not the source.

### Rust will make the future more secure

Compiling to JavaScript will be OK in some cases, but not for scope where system programming lies in general.
I don't think compiling to the browser will solve the system security problem, because:

* You can't verify that the source is actually doing as promised
* It only emulates a sandboxed cross platform distribution

There are cases you want to be permissible to what the software can do, as long as it is understood.
For example, if you are feeding an application in the browser your personal information,
and you can't see what the application does from the source, then there is no guarantee it does as it promises.
Also, the browser itself might be comprimised and modify the source without your knowledge.

On the other hand, if I compile a native binary from source,
I feel more secure, because usually I am not that worried about my computer, but about my personal information!

As people get more technical insight, they want to trust the software is doing as promised.
They don't want to be limited technically in what the software can do for them.

### How will indie developers make money on Rust?

First, it is going to be hard to make money directly on libraries,
because people will expect the source to be open.
The way to make money on the distribution part is through services and downloadable content.

Secondly, sharing libraries only mean less work for the developer and less maintenance cost.
It also means better control over dependencies. There is also a psychological benefit of not working in isolation.

The whole Piston project is developed with the MIT license, so you are in full right to
use close source for platforms or ecosystems where you find it necessary.

### Will Piston be there in the future?

I believe that people should not be forced to pick one solution that is a substandard of what they need.
Therefore, we are designing Piston to be modular in every possible way,
such that you can take out the piece you want.

For example, we are not building abstractions on top of dependencies.
We are building the back-end on top of the abstractions,
such that the software stack becomes a "root" instead of a "tree".

Most code depend on pure Rust abstractions, which you can plug in custom hardware and custom platforms.
This means no vendor lock-in or paying for a developer license, ever.

The only way Piston can continue to flourish is by being the best designed ecosystem for applications.
This is a high constraint to put on yourself, but I think its for best of end users.
Besides, the people developing it are also the ones benefiting from how it is used.
If it is not usable, then nobody wants it to survive.

We are also being modular in the way we deal with human resources.
For example, we give every collaborator full write access to all repositories.
There are assigned maintainers to each project, but if something goes wrong, anyone can step in.

I think Piston is pretty well prepared for the future.

### Piston is shifting to second gear

I believe Piston will bring safer systems applications not just to programmers,
but to all people, from those who want to tinker with computers up to professionals,
without going through all the hazzards and fallpits of low level programming.

We will push the direction toward general application programming:

* We are designing a [cross platform API for user input](https://github.com/pistondevelopers/input)
* We are collaborating closely with [Gfx](https://github.com/gfx-rs/gfx-rs), the safe low level graphics API
* We are developing a [cross platform, hardware accelerated user interface](https://github.com/pistondevelopers/conrod)
* We are doing research on the Window back-end side

Some latest features include:

* Unicode input
* Anti aliasing
* Lots of widgets, as demonstrated [here](http://blog.piston.rs/2014/08/30/conrod-update/)

### Where are we going?

I dream of Piston changing how we write applications, in particular for new hardware.
There are plenty of people that have ideas and dreams of their own,
which can become a reality through the tools we are developing.
The adaption of Piston in the established industry would be just a bonus.

Happy coding!
