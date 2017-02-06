---
layout: post
title: "The Mathematics of Infinite-Things-Space"
author: bvssvni
---

I am currently doing some research on procedurally generated 3D by inflating structures.

This blog post is about how the mathematics work.

### Visualizing Infinite-Things-Space

Vertex data in a 3D structure is often stored in an array.  
Ignoring the position and other attributes of vertices,
each vertex can be represented as a natural number which refers to the index in the array of vertices.

To connect two vertices, I use a math formula that takes an index and outputs a new one.  
The nice thing about this method is that you can compactly represent an infinite number of structures.

![step1](http://i.imgur.com/bWpZ6vk.png)

When the function returns the same index, I visualize it as a loop for each natural number from 0 to infinity.  

By adding by one, `f(x) = x + 1`, the loops are disconnect from themselves and attached to each other.  

To create circles with 4 vertices, use the formula `f(x) = x - x % 4 + (x+1) % 4`.

Mathematical operations on the natural numbers are equivalent to operations on these loops.  
For any mathematical expression of a single natural number, there is an equivalent shape in Infinite-Things-Space.  

In group theory, these functions are called "generators".

### The Super Function

The Super function is a higher order function that takes a generator and produces a new generator.
It preserves the structure of the input generator, but now "scaled up" with a factor of `n`.

```
super(n, f: N -> N) = \(x: N) = f(x / n) * n + x % n
```

The Super function is the key to combine multiple generators.

![step2](http://i.imgur.com/XkecKf7.png)

```
f_0(x) = x - x % 4 + (x + 1) % 4
f_1(x) = super(4, \(x) = x + 1)
```

You can use the Super function with any generator, for example circles to create a donut topology (torus):

![step3](http://i.imgur.com/LWCJbhE.png)

```
f_0(x) = x - x % n + (x + 1) % n
f_1 = super(n, \(x) = x - x % m + (x + 1) % m)
```

When `n` and `m` in the example above goes to infinity, you can start at 0,  
add infinity to take an infinitesimal step in the rotation direction of the torus.  
Take an Aleph-2 step (a higher infinity), and you jump from the 0-point at the first torus  
to the 0-point (ignoring Aleph-2) to the second torus.

### Inflating 3D structures

To inflate a 3D structure created by generators, I use an algorithm that takes a list of generators  
and walks along all edges for a fixed range, generating a list of spring constraints.  
If a generator leads back to the same vertex, it gets ignored.

The spring constraints are passed to a simulator that inserts random positions for vertices.   
An extra spring constraint drives inflation by being connected temporarily
between two random vertices `A` and `B` with a target distance `10 * (A - B).length().sqrt()`. 

Inflation causes an internal pressure outward on the whole shape, so it unfolds into its "natural form".  

The square root operation is to avoid too much acceleration between distant points,  
which might lead to stretching the shape like spaghetti.  

Here is a shape inflated by connecting cubes with some internal support to edges of a pentagon,  
which in turn is connected by a super hundredgon (a circle shape with 100 vertices).  

![torus](https://pbs.twimg.com/media/C3cjdmAWIAAL9vn.jpg)

As you see in the picture above, the shapes do not become perfect.  
This is because the springs are not possible to satisfy unless the differential topology is complete.  
Because the inner circle is the same size as the outer one, the shape above twists and stretches the pentagons.

Finding a complete differential topology could be done by "tuning" the constraints while simulating,  
or perhaps relaxing the target distances based on a function of the tension.  
Alternatively, some new constraints could be added to keep edges at a fixed angle relative to each other.  

The twisting effect can be used on purpose, e.g. on a cylinder with sinus modified distances:

![twist](https://pbs.twimg.com/media/C3bCcZJWEAExIXw.jpg)

Have you seen how the sweets with double twisted paper get wrinkles? This seems to be the same effect!
