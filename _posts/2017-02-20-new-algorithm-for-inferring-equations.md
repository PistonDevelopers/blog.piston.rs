---
layout: post
title: "New Algorithm for Inferring Equations"
author: bvssvni
---

In this post, I introduce a new practical algorithm for inferring equations from data.
It is inspired by category theory.

The algorithm is a recent result from the ongoing research on [path semantics](https://github.com/bvssvni/path_semantics),
and the first one that can infer asymmetric paths efficiently (equivalent to a subset of equations)!!!
However, we do not have Artificial Superintelligence yet, because the algorithm currently requires human input. :P

The algorithm has these properties:

1. All equations up to a chosen complexity gets inferred.
3. No need to assign a prior hypothesis.
4. Adaptive filter suitable for human assisted exploration.

Here it is (Dyon script):

```rust
/// Gets the shared equivalences.
fn infer(as: [[[str]]]) -> [[str]] {
    a := as[0]
    return sift k {
        eq := a[k]
        if !all i [1, len(as)) {
            b := as[i]
            any k {(eq == b[k]) || (eq[0] == b[k][1]) && (eq[1] == b[k][0])}
        } {continue}
        clone(eq)
    }
}

/// Prints out data.
fn print_data(a: {}) {
    println("===== Finished! =====")
    keys := keys(a.data)
    for i {println(link {keys[i]": "str(a.data[keys[i]])})}
    println("")
    println("Equivalents:")
    for i {println(link {a.equivs[i][0]" == "a.equivs[i][1]})}
}


fn wrap_fill__data_explore_filter(mut data: {}, explore: [[str]], filter: [[str]]) -> res[{}] {
    equivs := []
    gen := fill(data: mut data, explore: explore, filter: filter, equivs: mut equivs)
    if len(gen) != 0 {
        return err(link i {
            println(link {
                str(gen[i])": "
                str(\data[gen[i][0]](data[gen[i][1]]))
            })
        })
    }
    return ok({
        data: data,
        equivs: equivs
    })
}

fill__data_explore_filter_equivs(mut data: {}, explore: [[str]], filter: [[str]], mut equivs: [[str]]) = {
    explore(mut data, explore, mut equivs)
    gen(data, filter, mut equivs)
}

key(f, x) = str(link {f"("x")"})

/// Adds objects to category to explore.
fn explore(mut data: {}, explore: [[str]], mut equivs: [[str]]) {
    keys := keys(data)
    for i {
        key := key(explore[i][0], explore[i][1])
        if has(data, key) {continue}
        if !has(data, explore[i][0]) {continue}
        if !has(data, explore[i][1]) {continue}
        data[key] := \data[explore[i][0]](data[explore[i][1]])
    }
}

/// Generates alternatives and looks for equivalences.
fn gen(data: {}, filter: [[str]], mut equivs: [[str]]) -> [[str]] {
    ret := []
    keys := keys(data)
    for i {
        for j {
            if any k {(len(filter[k]) == 2) && (filter[k] == [keys[i], keys[j]])} {continue}

            if i != j {
                same := try data[keys[i]] == data[keys[j]]
                if is_ok(same) {
                    if unwrap(same) {
                        if any k {
                            (equivs[k] == [keys[i], keys[j]]) || (equivs[k] == [keys[j], keys[i]])
                        } {continue}
                        push(mut equivs, [keys[i], keys[j]])
                    }
                }
            }

            r := try \data[keys[i]](data[keys[j]])
            if is_ok(r) {
                key := key(keys[i], keys[j])

                if any k {(len(filter[k]) == 1) && (filter[k][0] == keys[i])} {continue}

                if has(data, key) {continue}
                push(mut ret, [keys[i], keys[j]])
            }
        }
    }
    return clone(ret)
}
```

### Example: Even and zero under addition and multiplication

```rust
fn test(x, y) -> res[{}] {
    data := {
        add: \(a) = \(b) = (grab a) + b,
        mul: \(a) = \(b) = (grab a) * b,
        even: \(a) = (a % 2) == 0,
        and: \(a) = \(b) = (grab a) && b,
        or: \(a) = \(b) = (grab a) || b,
        is_zero: \(a) = a == 0,
        eq: \(a) = \(b) = (grab a) == b,
        xy: [x, y],
        fst: \(xy) = clone(xy[0]),
        snd: \(xy) = clone(xy[1]),
    }
    filter := [
        ["mul"],
        ["or"],
        ["add"],
        ["and"],
        ["eq"],
        ["eq(even(fst(xy)))"],
        ["add(fst(xy))"],
        ["mul(fst(xy))"],
        ["or(even(fst(xy)))"],
        ["and(is_zero(fst(xy)))"],
        ["or(is_zero(fst(xy)))"],
    ]
    explore := [
        ["fst", "xy"],
        ["snd", "xy"],
        ["even", "fst(xy)"],
        ["even", "snd(xy)"],
        ["is_zero", "fst(xy)"],
        ["is_zero", "snd(xy)"],
        ["eq", "even(fst(xy))"],
        ["eq(even(fst(xy)))", "even(snd(xy))"],
        ["add", "fst(xy)"],
        ["add(fst(xy))", "snd(xy)"],
        ["even", "add(fst(xy))(snd(xy))"],
        ["is_zero", "add(fst(xy))(snd(xy))"],
        ["mul", "fst(xy)"],
        ["mul(fst(xy))", "snd(xy)"],
        ["is_zero", "mul(fst(xy))(snd(xy))"],
        ["even", "mul(fst(xy))(snd(xy))"],
        ["or", "even(fst(xy))"],
        ["or(even(fst(xy)))", "even(snd(xy))"],
        ["and", "is_zero(fst(xy))"],
        ["and(is_zero(fst(xy)))", "is_zero(snd(xy))"],
        ["or", "is_zero(fst(xy))"],
        ["or(is_zero(fst(xy)))", "is_zero(snd(xy))"],
    ]

    return wrap_fill(data: mut data, explore: explore, filter: filter)
}
```

Program:

```rust
fn main() {
    a := unwrap(test(2, 3))
    b := unwrap(test(3, 3))
    c := unwrap(test(2, 2))
    d := unwrap(test(3, 2))
    e := unwrap(test(0, 1))
    f := unwrap(test(1, 0))
    c := infer([a.equivs, b.equivs, c.equivs, d.equivs, e.equivs, f.equivs])
    
    if len(c) > 0 {
        println("========== Found equivalences!!! ==========\n")
        for i {println(link {str(c[i][0])" == "str(c[i][1])})}
        println("\n===========================================")
    } else {
        println("(No equivalences found)")
    }
}
```

Output:

```
========== Found equivalences!!! ==========

or(is_zero(fst(xy)))(is_zero(snd(xy))) == is_zero(mul(fst(xy))(snd(xy)))
even(add(fst(xy))(snd(xy))) == eq(even(fst(xy)))(even(snd(xy)))
or(even(fst(xy)))(even(snd(xy))) == even(mul(fst(xy))(snd(xy)))
is_zero(add(fst(xy))(snd(xy))) == and(is_zero(fst(xy)))(is_zero(snd(xy)))

===========================================
```

First you start with empty lists, and the algorithm give you choices of how to expand the category.
The value of the new objects are printed (Dyon can print closures).
Choices are either added to filter or for exploration, until there are no new objects.
Finally, the program prints out the inferred equivalences.

The filter has two settings, one that disables choices for a whole function, e.g. `["mul"]`,
and another that disables a choice for a specific input to a function, e.g. `["mul", "x"]`.

This gives all equations found among the objects, but can yield equations that does not hold in a wider context.
If the context is too narrow, add more test data.

You can also infer implications by adding a `true` object and the implication rule:

```
imp: \(a) = \(b) = if grab a {clone(b)} else {true},
t: true,
```

The same trick can be used infer approximate equality.

### How it works

A category is a set of objects which has a set of morphisms (arrows) between them that compose, plus identity.
The algorithm treats composition as an unknown quantity over all input data.

Therefore, by constructing a finite category for each input, the equivalences can be found by taking
the intersection of found equivalences for all inputs afterwards.

In order to construct a category of mathematical expressions, the functions must be encoded as single-argument closures.
This is because an arrow can only point from one object in a category.

For example:

```
or(is_zero(fst(xy)))(is_zero(snd(xy)))
```

is equivalent to:

```
is_zero(fst(xy)) || is_zero(snd(xy))
```

A filter is necessary because of the combinatorial explosion.
With some training, this becomes easy to use.

Exploration step in this algorithm adds new objects to the category prior to generating new alternatives.
The name of these objects corresponds to the mathematical expressions.
Therefore, an equation is the same as comparing two objects in the category for equivalence.
You do not have to compare the paths, because the variance in input propagates with the construction of the explored objects.
