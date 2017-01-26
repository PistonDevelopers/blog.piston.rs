---
layout: post
title: "A brief teaser about Composable Parallel Stream Processing with Rayon"
author: bvssvni
---

A few days ago, I was about to write a blog post about a programming pattern in Rust,
using the [Rayon](https://crates.io/crates/rayon) library.
Then, I discovered there are some things to learn about this pattern that could lead to
other variations of it, in some ways that is more closer to the original inspiration:
[The Nile programming language](https://github.com/damelang/nile) that I mentioned in the [Happy New Year blog post](http://blog.piston.rs/2016/12/31/happy-new-year/).
I figured out this requires some more time to study, but by request, here is a teaser blog post about it.

This is one of those ideas that you can start from various places and arrive at the same design.
Sooner or later, I would be surprised if people have not already,
somebody else will get the exact same idea as described here.
The design is so simple, so there is not much originality to it.

To explain how it works, I will choose a matematical background first and then work from there.

In mathematics, the most important abstraction is a function.

The way we are used to think about functions is by taking some arguments and returning values.

For example:

```rust
fn f(f32) -> f32 { ... }
fn g(u32) -> f32 { ... }
```

It is obvious that `f` and `g` are composable, but they are only composable in a specific way (because of types):

```rust
f(g(x))
```

So, when I describe the pattern as "Composable Parallel Stream Processing" you now understand what the first
word means: "Composable" just means combining some building blocks like we combine functions in mathematics.

Another pattern in Rust that is composable: Iterators.
The difference between a mathematical function and an iterator is that iterators have an internal state they use as input.
Rayon is designed to turn sequential iterator pattern into parallel code.
Instead of `.iter()` you type `.par_iter()`, instead of `.iter_mut()` you type `.par_iter_mut()` etc.

Rayon also supports `.par_chunks` and `.par_chunks_mut`, which will be used extensively in our pattern.

We start with ordinary mathematical functions and do two transformations.

The first transformation is, instead of calling the function for each item in a list,
we pass an immutable slice and write the output to mutable slice:

```rust
fn f(&[f32], &mut [f32]) { ... }
fn g(&[u32], &mut [f32]) { ... }
```

This is still composable, but now for slices instead of single values:

```rust
let a = vec![0, 1, 2];
let mut b = vec![0.0; a.len()];
let mut c = vec![0.0; a.len()];
g(&a, &mut b);
f(&b, &mut c);
```

In stream processing this is common because algorithms operate on chunks of data instead of single values.

The second transformation is, instead of calling the functions directly,
we create the composition using higher order functions (functions that take functions and return functions):

```rust
type Func<I, O> = Box<Fn(&[I], &mut [O]) + Sync>

fn f(g: Func<u32, f32>) -> Func<u32, f32> { ... }
fn g() -> Func<u32, f32> { ... }
```

This is also composable, but easier to use than those functions taking slices:

```rust
let fx = f(g());
let input = vec![0, 1, 2];
let output = vec![0.0; data.len()];
fx(&input, &mut output);
```

Now, why would anyone do that? Here are some reasons:

1. When processing a lot of data, deep compositions are faster when operating over slices, even in sequential code
2. Boxed closures are first class citizens, which makes it easier to reuse algorithms
3. Rayon makes it easy to parallelize the tasks done over such slices
4. Optimizations that take advantage of data locality patterns (e.g. chunk branching instead of branching per item)

Not only is this pattern faster in sequential code and easier to reuse, but it is also easier to parallelize in Rust!

OK, so this stuff sounds nice, but does it work in practice?

What about image processing?

Taking an idea from Nile, instead of thinking of textures as an object you keep around,
you can think of them as a parallelizable function `position >> color`.
If you translate a texture, you could think of it as a higher order function operating
on `position >> color` that returns a new parallelizable function of type `position >> color`.

Like this (NB! Not a complete example, but you get the idea):

```rust
extern crate image;
// Piston's float abstraction has recently been updated for this kind of generic code!
// https://crates.io/crates/piston-float
extern crate float;
extern crate rayon;

use std::sync::Arc;
use image::RgbaImage;
use float::Float;
use rayon::prelude::*;

const CHUNK_SIZE: usize = 1024;

/// Stores color information.
pub type C4<T> = [T; 4];

/// Stores coordinates for 2D vectors.
pub type V2<T> = [T; 2];

/// Samples from an image.
pub fn texture(img: Arc<RgbaImage>) -> Func<V2<f32>, C4<f32>> {
    let (width, height) = img.dimensions();
    let width = width as f32;
    let height = height as f32;
    Box::new(move |input, output| {
        for (it, ot) in input.iter().zip(output.iter_mut()) {
            *ot = if it.x() < 0.0 || it.x() >= width ||
                     it.y() < 0.0 || it.y() >= height
                {
                    // Use transparent color outside the image.
                    [0.0; 4]
                } else {
                    let c = img.get_pixel(it.x() as u32, it.y() as u32).data;
                    [
                        c[0] as f32 / 255.0,
                        c[1] as f32 / 255.0,
                        c[2] as f32 / 255.0,
                        c[3] as f32 / 255.0
                    ]
                };
        }
    })
}

/// Translates in 2D.
pub fn trans2<T: Float, U: Send + 'static>(
    pos: V2<T>,
    f: Func<V2<T>, U>
) -> Func<V2<T>, U> {
    Box::new(move |input, output| {
        input.par_chunks(CHUNK_SIZE)
             .zip(output.par_chunks_mut(CHUNK_SIZE))
             .for_each(|(i, o): (&[V2<T>], &mut [U])| {
                 /// Use stack memory to avoid allocations.
                 let mut i2: [V2<T>; CHUNK_SIZE] = [[T::zero(), T::zero()]; CHUNK_SIZE];
                 for (p, pi) in i2[..i.len()].iter_mut().zip(i.iter()) {
                     *p = pi.sub(pos);
                 }
                 f(&i2[..i.len()], o)
             })
    })
}
```

Notice that since we know the size of the chunk that gets parallelized,
we can use stack allocated memory without checking for overflow!

Here is an example of how it looks like when using it:

```rust
fn main() {
    let width: u32 = 300;
    let height: u32 = 300;

    let wood = Arc::new(image::open("assets/materials/wood.png")
        .unwrap().to_rgba());

    let f = front_to_back(vec![
            trans2([10.0; 2], texture(wood)),
            color(blue())
        ]);

    let mut image = RgbaImage::new(width, height);
    // Render with 4x4 super sampling.
    render(4, &f, &mut image);
    image.save("assets/test.png").unwrap();
}
```

Again, this is a very simple pattern, so while it is not so interesting in itself,
there are some nice optimization techniques that are possible, and interesting things you can do with it.
This might come in later blog posts.

This is just a teaser, but you can try out this pattern for yourself and compare it to other techniques!
