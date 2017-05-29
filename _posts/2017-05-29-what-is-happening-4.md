---
layout: post
title: "What is Happening 4"
author: bvssvni
---

Follow [@PistonDeveloper at Twitter](https://twitter.com/PistonDeveloper)!

Here are some of the things that happened since [the last post](http://blog.piston.rs/2016/09/19/what-is-happening-3/):

### [AdvancedResearch](https://advancedresearch.github.io/)

Advanced research focuses on making conceptual breakthroughs in system thinking.
The previous advanced research under the Piston project is moving to this new open source organization.
You can read about the rationale of separating this research from the Piston project [here](https://github.com/PistonDevelopers/piston/issues/1177).

Some bullet points of the overall progress:

- New approach to mathematical foundation of friendly artificial intelligence nicknamed "golden rationality"
- Some progress on path semantics and generalizations to probability theory
- Gini solvers could balance economic inequality (can be tested in MMOs)
- Established contact with scientists and engineers about mitigating climate change

Golden rationality is a very recent topic, where I try avoiding the [scary first order approximation to friendly superintelligence](http://blog.piston.rs/2017/04/15/no-superintelligence/) and
instead focus on human value approximation concerning group strategies.
This means it is not about superintelligence "in a box", but about beating the efficiency of individual rational agents
in a world where AI is standardized and utilized by many people.
For discussion and questions, open up an issue [here](https://github.com/advancedresearch/advancedresearch.github.io/issues).

Normal research related to gamedev will continue under the Piston project.

### [Conrod](https://github.com/PistonDevelopers/conrod/)

- New `CollapsibleArea` widget
- Improved performance
- Added crates.io categories
- Various bug fixes and improvements

[List of contributors (59)](https://github.com/PistonDevelopers/conrod/graphs/contributors)

### [Image](https://github.com/pistondevelopers/image)

There is an [internal compiler bug](https://github.com/PistonDevelopers/image/issues/645) on Rust 18 Beta,
so if you have troubles compiling your project, you might want to use Right stable or nightly instead.

- Simple BMP support
- PPM support
- New constructor for adjusting quality of JPEG encoding

[List of contributors (89)](https://github.com/PistonDevelopers/image/graphs/contributors)

### [Imageproc](https://github.com/PistonDevelopers/imageproc)

- Improved performance, incluing better parallization using Rayon
- Added more benchmarks
- Ellipse and bezier drawing
- Various bug fixes and improvements

[List of contributors (14)](https://github.com/PistonDevelopers/imageproc/graphs/contributors)

### [Dyon](https://github.com/PistonDevelopers/dyon)

- New intrinsics 
- Optional namespaces with shared aliases
- Link loop (a loop that concatenates body expressions using Dyon's `link` data structure)
- Try expressions (turning errors into result)
- Various bug fixed and improvements

[List of contributors (4)](https://github.com/PistonDevelopers/dyon/graphs/contributors)

### [VisualRust](https://github.com/PistonDevelopers/visualrust)

- Cargo support
- Support for both Visual Studio 2015 and 2017
- Various bug fixes and improvements

[List of contributors (14)](https://github.com/PistonDevelopers/VisualRust/graphs/contributors)

### Other projects

- New versions of Gfx has been updated through the ecosystem
- [resize](https://github.com/PistonDevelopers/resize) has gotten nice upgrades and performance improvements
- [music](https://github.com/PistonDevelopers/music) now supports sound clips
