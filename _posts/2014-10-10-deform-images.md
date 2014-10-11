---
layout: post
title: Deform Images
author: bvssvni
---

Recently I added a `DeformGrid` object to [piston-graphics](https://github.com/PistonDevelopers/graphics)

Here is a demo of it in action:

<iframe width="420" height="315" src="//www.youtube.com/embed/qW4OlBWH5Gs" frameborder="0" allowfullscreen></iframe>

Example code is in the "deform" app under [piston-examples](https://github.com/pistondevelopers/piston-examples).

I have been using this algorithm for years in a 2D animation software.
The algorithm is called "Rigid Moving Least Squares Deformation".

The algorithm takes 42 lines of code:

```Rust
for m in range(0, nx) {
    for n in range(0, ny) {
        let ip = m + n * nx;
        let vx = m as f64 * units_h + x;
        let vy = n as f64 * units_v + y;
        let mut sum_wi = 0.0;
        let mut p_star_x = 0.0; let mut p_star_y = 0.0;
        let mut q_star_x = 0.0; let mut q_star_y = 0.0;
        for i in range(0, num) {
            let pix = ps[i][0]; let piy = ps[i][1];
            let qix = qs[i][0]; let qiy = qs[i][1];
            let vl = (pix - vx) * (pix - vx) + (piy - vy) * (piy - vy);
       
            let w = if vl < eps && vl > -eps { 1.0 / eps } else { 1.0 / vl };
            sum_wi += w;
            p_star_x += w * pix; p_star_y += w * piy;
            q_star_x += w * qix; q_star_y += w * qiy;
            wis[i] = w;
        }

        p_star_x /= sum_wi; p_star_y /= sum_wi;
        q_star_x /= sum_wi; q_star_y /= sum_wi;
        let mut fr_x = 0.0; let mut fr_y = 0.0;
        let vpx = -(vy - p_star_y); let vpy = vx - p_star_x;
        for i in range(0, num) {
            let pix = ps[i][0]; let piy = ps[i][1];
            let qix = qs[i][0]; let qiy = qs[i][1];
            let pi_hat_x = pix - p_star_x; let pi_hat_y = piy - p_star_y;
            let qi_hat_x = qix - q_star_x; let qi_hat_y = qiy - q_star_y;
            let ai11 = pix * vpy - piy * vpx;
            let ai21 = pi_hat_y * vpy + pi_hat_x * vpx;
            let ai12 = pix * (-vpx) - piy * vpy;
            let ai22 = pi_hat_y * (-vpx) + pi_hat_x * vpy;
            fr_x += wis[i] * (qi_hat_x * ai11 + qi_hat_y * ai21);
            fr_y += wis[i] * (qi_hat_x * ai12 + qi_hat_y * ai22);
        }

        let vl = vpy * vpy + vpx * vpx;
        let fl = fr_x * fr_x + fr_y * fr_y;
        let vl = if fl == 0.0 { 0.0 } else { (vl / fl).sqrt() };
        fr[ip][0] = fr_x * vl + q_star_x;
        fr[ip][1] = fr_y * vl + q_star_y;
    }
}
```

When you work on a complicated math algorithm like this,
the best thing is often to type out every addition and multiplication.
This makes it far easier to translate to another programming language,
because you don't have to depend on a math library (there are exceptions of course, like determinants).
There are some other reasons we will examine later in this post.

The variable names are chosen to reflect [the original paper](https://github.com/PistonDevelopers/graphics/issues/625).
One difference is that is uses a fixed "alpha" for performance reasons.

I wish computer scientists provided such example code with research papers!

### Usage

Deformed images are common on objects that require organic animation, such as characters or plants.
[Here](https://www.youtube.com/watch?v=8w11OsruvZw) is an example where it used to create a game
with the look of the old He-Man cartoon movies.
It was animated and exported to sprites, so the deforming does not happen in real time.

### How does it work?

Perhaps you are familiar with matrix transformations?
A matrix transformation is a general linear map,
which means it can transform while keeping lines straight.
The algebra on matrices is called linear algebra and is one of the central topics in mathematics.

While it seems difficult on the surface, it all boils down to: `+ - * / .sqrt()` and some loops.
These are very simple operations, which are put together in patterns, and then comes out as a deformed image!

Instead of transforming every pixel, we transform a grid and render the image on the GPU.

### Algorithm walkthrough

Let's look at the first part:

```Rust
// Get the coordinate in the grid.
let vx = m as f64 * units_h + x;
let vy = n as f64 * units_v + y;
// Initialize variables to sum something.
let mut sum_wi = 0.0;
let mut p_star_x = 0.0; let mut p_star_y = 0.0;
let mut q_star_x = 0.0; let mut q_star_y = 0.0;
// For each control point.
for i in range(0, num) {
    // Get original control point P.
    let pix = ps[i][0]; let piy = ps[i][1];
    // Get deformed control point Q.
    let qix = qs[i][0]; let qiy = qs[i][1];
    // Get distance^2 from original control point.
    let vl = (pix - vx) * (pix - vx) + (piy - vy) * (piy - vy);

    // Get 1 / distance^2, with some trick to prevent division by zero.
    // This is a "weight" of the current point.
    let w = if vl < eps && vl > -eps { 1.0 / eps } else { 1.0 / vl };
    // Sum up weights.
    sum_wi += w;
    // Multiply weight with the point, will use this value later.
    p_star_x += w * pix; p_star_y += w * piy;
    q_star_x += w * qix; q_star_y += w * qiy;
    // Remember the weight for later.
    wis[i] = w;
}

// Divide by sum of weights to get something called "weighted average".
p_star_x /= sum_wi; p_star_y /= sum_wi;
q_star_x /= sum_wi; q_star_y /= sum_wi;
```

Notice that the weight is `1 / distance^2`.
This means that the closer you are to a control point,
the more influenced you are by that point.

"Weighted average" tells us how much a point is influenced by all control points.
It gives an ideal position where the 1 / distance^2 weights are balanced.
We need two weighted averages, one for the original and one for the deformed grid.

Since the second part is most complex, let's wait with it until the end.
Here is the last part:

```Rust
// Normalization trick to get length of vector.
let vl = vpy * vpy + vpx * vpx;
let fl = fr_x * fr_x + fr_y * fr_y;
let vl = if fl == 0.0 { 0.0 } else { (vl / fl).sqrt() };
// Add vector to weighted average in deformed coordinates.
fr[ip][0] = fr_x * vl + q_star_x;
fr[ip][1] = fr_y * vl + q_star_y;
```

This gives us the final deformed point.

Notice the second step computes something to add to the weighted average!
How can this be?

You can [override dependencies](https://github.com/PistonDevelopers/piston/issues/482)
with Cargo to experiment by putting some `0.0` in front of `fr_x` and `fr_y`.
If you run this, the image will become a shrinked shape with spikes attached to the control points.

![spiked image](http://i.imgur.com/JR1JAwl.png)

This tells us that the effect of the second part
of the algorithm wears off when we get close to a control point!
The second part of the algorithm is just to straighten out rest of the warped image.

Notice that `(vl / fl).sqrt()` saves us one square root operation.
This is because to get a length of a vector, you take the square root
of the dot product with itself.
Since we are dividing one length of a vector with another,
we can move the square root operation to after the division,
while getting the same result.
Typing out every operation explicitly makes such optimization easier.
This why it is sometimes better to type out every operation.

Now to the second part:

```Rust
// The new deformed position is given by a sum.
let mut fr_x = 0.0; let mut fr_y = 0.0;
// Get normal vector from original weighted average to position.
let vpx = -(vy - p_star_y); let vpy = vx - p_star_x;
for i in range(0, num) {
    // Get original control point P.
    let pix = ps[i][0]; let piy = ps[i][1];
    // Get deformed control point Q.
    let qix = qs[i][0]; let qiy = qs[i][1];
    // Vector from original weighted average to original control point.
    let pi_hat_x = pix - p_star_x; let pi_hat_y = piy - p_star_y;
    // Vector from deformed weighted average to deformed control point.
    let qi_hat_x = qix - q_star_x; let qi_hat_y = qiy - q_star_y;
    // Compute a 2x2 matrix using the normal vector,
    // and the vector from original weighted average to original control point.
    let ai11 = pix * vpy - piy * vpx;
    let ai21 = pi_hat_y * vpy + pi_hat_x * vpx;
    let ai12 = pix * (-vpx) - piy * vpy;
    let ai22 = pi_hat_y * (-vpx) + pi_hat_x * vpy;
    // Sum transformed vectors in deformed coordinates by weights.
    fr_x += wis[i] * (qi_hat_x * ai11 + qi_hat_y * ai21);
    fr_y += wis[i] * (qi_hat_x * ai12 + qi_hat_y * ai22);
}
```

Notice that we don't use the deformed control points before the very last step.
We could precompute the matrix, but that would require one matrix for every point.
Storing that much memory might even be slower!
The algorithm is `O(N * M * P)` where `N` is grid width, `M` is grid height and `P` is number of control points.
This remains unchanged even if you precompute the matrices.

Enjoy!
