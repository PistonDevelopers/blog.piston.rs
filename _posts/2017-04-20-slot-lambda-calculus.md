---
layout: post
title: "Slot Lambda Calculus"
author: bvssvni
---

In this post I will describe a useful technique for formal study of language semantics.

[Link to paper](https://github.com/bvssvni/path_semantics/blob/master/papers-wip/slot-lambda-calculus.pdf) (work in progress)

Usually when somebody mentions the words "language semantics", people run and hide under their beds,
hoping that the person uttering these words will not find them there any time soon.

Yet, I want to show that given the right tools, semantics does not need to be that hard to wrap your head around.

After reading this post, you should be able to claim something as powerful as:

*There are certain rules that are valid for meaning in all languages, and I know what they are.*

One reason semantics seems hard, is because we think of the world as this place of infinite possibilities,
and since language is used to describe what happens in the world,
then the meaning of language must consist of infinite possibilities.

Some of these infinite possibilites are:

- When programming, people can type in a text editor, save to a file and use a compiler (complex)
- When talking, people can use different sounds to change the meaning of words (complex)
- When expressing themselves in art, people can shape an experience that points to something "elsewhere"
(if this is not complex, I have no idea what complex is)

All these activities are related to the meaning of language, and this makes the world a complex place.

Slot lambda calculus is a tool that takes those "infinite possibilities" and makes them very, very simple.

First, instead of an infinite number of ways of constructing expressions in a language,
in slot lambda calculus there is only one way. For example, if you want to express `1 + 1 = 2`:

```
  (_ = _)(_ + _)(1)(1)(2)
```

This gets turned into:

```
    ((1 + 1) = 2)
```

So, where did `(_ = _)` and `(_ + _)` come from? They were constructed the same way:

```
    (_ _)(?)(_ _)("=")(\)
    (_ _)(?)(_ _)("+")(\)
```

The `?` symbol inserts a placeholder, and `\` removes all `?`s and turns all strings into syntax.

This gets turned into:

```
    (_ (= _))
    (_ (+ _))
```

When you have defined a pattern like this, you can write equations to make them nicer:

```
    (_ (= _)) = (_ = _)
    (_ (+ _)) = (_ + _)
```

Slot lambda calculus behaves similarly to normal lambda calculus, but uses a different application rule.
Normal lambda calculus is even simpler, but more suitable for computation than reading new patterns created from existing patterns.
However, there is a way to hack lambda calculus so you get slot lambda calculus! Mwuhahaha!

This is how you do it [Dyon](https://github.com/pistondevelopers/dyon) (without syntax extensions):

```rust
fn pass(f: \(any) -> any, val: any) -> any {
    return if typeof(val) == "closure" {
        \(x) = {
            val := grab val
            new_val := \val(x)
            f := grab f
            pass(f, new_val)
        }
    } else {
        \f(val)
    }
}

fn main() {
    add := \(x) = \(y) = (grab x) + y
    mul := \(x) = \(y) = (grab x) * y

    a := pass(add, mul) // (_ * _) + _
    b := pass(a, 2) // (2 * _) + _
    c := pass(b, 3) // (2 * 3) + _
    d := pass(c, 1) // (2 * 3) + 1
    println(d) // prints `7`
}
```

Notice that this is the exact same behavior as a stack machine, which is common in the runtime for most programming languages.
The use of single argument closures has some nice mathematical properties, e.g. to [infer equations from data](http://blog.piston.rs/2017/02/20/new-algorithm-for-inferring-equations/).

Now, when you are looking at `(_ + _)`, you might wonder: What does it mean?
When designing meaning for a new language, you might not have an interpreter,
or even know in which context the expression `(_ + _)` is valid.

How do you figure out what `(_ + _)` means when all you have is `(_ + _)`?

Again, back to slot lamdba calculus:

1. For an expression to be interpreted, it must be *constructed* in its context.
2. Any construction is reducible to a calculation with slot lambda calculus.
3. Meaning is *the rules we use* when constructing a particular expression.

These are the rules that are valid for all languages, despite infinite possibilities.

You can study the context by using the construction in slot lambda calculus!

For example:

```
    (_ = _)(_ + _) = (_ + _ = _)
```

This new construction has 3 empty slots.

You now have two choices:

1. `(_ + _)` shall have a unique meaning for `(_ + _ = _)`.
2. `(_ + _)` shall have same meaning wherever it is used.

The 2nd option is called "context free". It means whenever you see the pattern `(_ + _)` you can reuse the same rules.

Since you can construct any pattern you like, you can create one pattern for each meaning you assign to your language.
When you have done this, there is a proven technique in slot lambda calculus that shows how you can be sure that
all edge cases are checked, such that all expressions are either invalid or assigned to a rule.

A "meaning" in this sense corresponds to a set of rules that decides what you should do when encountering a specific pattern.
The semantics of a pattern is only well defined when the rule is deterministic,
such that the same action is repeated when the pattern occurs over and over.

OK, now let us have some fun: We want to create a tiny language for describing what happens in a romantic novel.

```
    (_ loves _)(<name>)(<name>)
    (_ hates _)(<name>)(<name>)
    (_ ignores _)(<name>)(<name>)
    (_ see _)(<name>)(_)
        (_ see _)(<name>)(<name>)
		    (_ see _ loves _)(<name>)(<name>)(<name>)
		    (_ see _ hates _)(<name>)(<name>)(<name>)
		    (_ see _ ignores _)(<name>)(<name>)(<name>)
		    (_ see _ see _ loves _)(<name>)(<name>)(<name>)(<name>)
		    (_ see _ see _ hates _)(<name>)(<name>)(<name>)(<name>)
		    (_ see _ see _ ignores _)(<name>)(<name>)(<name>)(<name>)
```

This language can describe relationships, e.g. `Bob loves Alice`,
but can also describe events that affect relationships e.g. `Bob see Alice loves Carl`.

With this tiny language, we can write some "code":

```
Bob loves Alice.
Alice loves Carl.
Bob see Alice loves Carl.
Bob hates Carl.
Carl see Bob hates Carl.
Carl ignores Bob.
Carl see Alice.
Carl loves Alice.
Bob see Alice see Carl loves Alice.
...
```

Can you visualize a story from something like this? I can, I wrote it down, and it was fun. :P (no, I will not share it)

*The big idea of formal languages is that you can give your own thoughts some new structure.
You don't have to think "with" the structure like a computer, but think of the world inside it.*

Just create a tiny language, write something random, and watch your thoughts become alive!

Slot lambda calculus is the latest technique I use to study [path semantics](https://github.com/bvssvni/path_semantics).
Path semantics grounds meaning using functions, in a powerful way that allows us to reason about a mathematics and programs.
In the attempt to formalize path semantics, I built a lot of other tools to make it easier for me to wrap my head around things.
I am getting closer, slowly, but it goes forward!

This is why it has been so hard to study path semantics: There are hundreds of edge cases to check!
If you have `N` patterns, then you get at least `N^2` cases you need to give an interpretation,
so see if it makes sense to use the pattern in that context.
For example, with 6 patterns you get 36 cases, but increase to 12 and you get 144!
I am not sure how many patterns that are required in a practical and convenient notation. 20? 30?

The reason we know path semantics is hard to formalize is because of the simple properties of slot lambda calculus.
Until recently it was a complete mystery for me how to check all the cases.
With slot lambda calculus it is possible to create map and measure progress.
