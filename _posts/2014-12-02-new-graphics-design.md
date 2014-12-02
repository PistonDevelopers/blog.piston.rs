---
layout: post
title: "New graphics library design"
author: bvssvni
---

[Piston-Graphics](https://github.com/pistondevelopers/graphics) got a new design.
The new design is based on a principle "objects as functions".
Let's take a look at it!

This is how you clear the screen and draw an image:

```Rust
graphics::clear([1.0, ..4], gl);
graphics::image(&image, &c, gl);
```

The `Context` object now only contains view and transform information,
so the actual render logic must be provided through other parts of the library.

When looking up the source code for the `image` function, you see this:

```Rust
/// Draws image.
pub fn image<B: BackEnd<I>, I: ImageSize>(
    image: &I, c: &Context, back_end: &mut B
) {
    Image::new().draw(image, c, back_end);
}
```

The `image` function is a convenience function for a more powerful way of rendering images.
The `Image` type is declared as following:

```Rust
/// An image
#[deriving(Copy)]
pub struct Image {
    /// The color
    pub color: Option<internal::Color>,
    /// The rectangle to draw image inside
    pub rectangle: Option<internal::Rectangle>,
    /// The image source rectangle
    pub source_rectangle: Option<internal::SourceRectangle>,
}
```

If you want a colored image, you can use `Image::colored([r, g, b, a]).draw(&image, &c, gl)`.

With [Piston-Current](https://github.com/pistondevelopers/current) you can also use builder methods:

```Rust
use current::Set;

Image::new().set(Color([r, g, b, a])).set(SrcRect([x, y, w, h])).draw(&image, &c, gl);
```

This pattern is a modified version of [Rust-Modifier](https://github.com/reem/rust-modifier),
tuned to work better with generics.

So why is the `Image` separated from the image data?

The principle is called "objects as functions" and defines a type of semantics
where you do not want to deal with pure states or pure functions.
First you think of what the main purpose of using the object is,
and use as argument the part that changes most frequently.
In the case of rendering an image, the actual image object changes most frequently.
We could call it `DrawImage` instead of `Image` but because the library is fully generic
over image objects, it makes sense to just use `Image` or `graphics::Image` if you have an image object.

Initially this started out as an experiment, but after seeing how nice this
played out in practice, I decided to go with it.

For example, it makes it easier to separate out stuff and name things from loops.
Here is an old piece of code that renders a snake's tail from [Sea Snake Escape](https://github.com/bvssvni/rust-snake):

```Rust
let n = snake.tail.len() / 2;
for i in range(0, n) {
    let x = snake.tail[i * 2];
    let y = snake.tail[i * 2 + 1];
    if (i / 8) % 2 == 1 {
        cam.circle(x, y, rad).color(colors::BLACK).draw(gl);
    } else {
        cam.circle(x, y, rad).color(settings::SNAKE_TAIL_COLOR).draw(gl);
    }
}
```

If you look briefly over the source, it is not easy to see immediate what going on.
Here is the new design:

```Rust
let black = graphics::Ellipse::new(colors::BLACK);
let tail = graphics::Ellipse::new(settings::SNAKE_TAIL_COLOR);
for i in range(0, n) {
    let x = snake.tail[i * 2];
    let y = snake.tail[i * 2 + 1];
    if (i / 8) % 2 == 1 {
        black.draw(graphics::ellipse::circle(x, y, rad), cam, gl);
    } else {
        tail.draw(graphics::ellipse::circle(x, y, rad), cam, gl);
    }
}
```

`black` and `tail` easier to spot because they appear first in the line.
Not a major improvement over the old design, but it is nicer.
I like how it refactors in ways that makes it easier to improve code incrementally.

The library is a lot easier to understand now, with -1000 loc (lines of code),
and it is easier to add new features in a modular way.

While the old design had the benefit of having no run time state,
such as whether to render a border or not,
the potential performance benefit was traded against the readability.
For example, `Rectangle` is also used for round and bevel shapes.
On the up side, you can draw a filled rectangle with a border through a single call.

The old design is pushed to branches "olddesign".
