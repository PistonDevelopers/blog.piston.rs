---
layout: post
title: New Meta Language
author: bvssvni
---

Meta parsing is a technique for generating parsing rules at runtime, using a meta language.
It was popular in the 60's and early 70's, but was mostly forgotten by mainstream programming until recent years.
See [meta compiler](https://en.wikipedia.org/wiki/Metacompiler) for more information.
Also, check out [OMeta](https://en.wikipedia.org/wiki/OMeta) which is the project that inspired me to learn more about it.

Previous blog posts:

- [Meta Language](http://blog.piston.rs/2015/06/18/meta-language/)
- [Piston-Meta 0.1](http://blog.piston.rs/2015/05/29/piston-meta/)

Disclaimer: If you do not understand this blog post, DON'T PANIC. Meta parsing is mental acrobatic.

[Piston-Meta](https://github.com/PistonDevelopers/meta) has upgraded the meta language to a nicer, more polished version!
It is still not perfect, but more readable than the old syntax.
Improving readability is quite a challenge since the meta language is very information dense.

Built-in rules now starts with `.` and uses `:` for the name of generated meta data.

The ugly `@` in front of custom rules is gone.

String starts with underscore to separate them from other rules.

Separated-by rule takes two arguments to avoid confusion with select rule.

### 3D coordinates example

Assume we want to parse a document containing a list of 3D coordinates, separated by commas.

```
0.3, 0.4, 0.7
0.4, 0.8, 0.9
0.5, 0.2, 0.7
```

Syntax:

```
// A rule for separating numbers with optional whitespace before and after.
0 , = [.w? "," .w?]
1 pos = [.w? .$:"x" , .$:"y" , .$:"z" .w?]
// The last rule is used for the entire document.
2 document = .l(pos:"pos")
```

`.$` is used to parse numbers as `f64`.
The "x", "y" and "z" tells what name to give the number.

The generated meta data is stored in a flat `Vec<Range<MetaData>>`, but we can print it out as JSON using `json::print`.
This is often used for debugging the syntax.
Extra zeros are caused by numbers that do not have an accurate floating number representation.

```json
"pos":{
 "x":0.30000000000000004,
 "y":0.4,
 "z":0.7000000000000001
},
"pos":{
 "x":0.4,
 "y":0.8,
 "z":0.9
},
"pos":{
 "x":0.5,
 "y":0.2,
 "z":0.7000000000000001
}
```

### Why Piston-Meta?

I game development, there are often many different text formats.
Normally this requires a dependency on a library per format.
With meta parsing, one can use a single library and change the syntax without recompiling the program.

Debugging parsers is often difficult but necessary when designing custom domain specific languages or text formats.
Piston-Meta has nice error reporting, where the deepest error is picked and with a reference to the rule generating the error.
It also gives document structure validation where a custom format is a subset of another, for example JSON.

When performance matters, it is common to use a parser generator or a custom binary format.
However, when prototyping or developing a game, readability, editing and version control matters.
Piston-Meta fills a niche where you do not want a whole programming language to test an idea,
or need more flexibility than just serializing Rust structures.

The library is still very experimental and redesigns can be expected.

### How the new syntax was developed

Meta parsing has the ability to parse its own rules in its own grammar.
This is the part where most people (me too sometimes!) gets confused.
However, you can exploit this to do faster self-improving.

I used this when developing the new syntax:

1. Write the new syntax in the old syntax.
2. Write the new syntax in itself.
3. Write the old syntax in the new syntax.
4. Replace the rules of the old syntax with the rules of the new syntax.

This happened gradually over several weeks, where the new syntax was tested in various projects.

At any given moment, the library has a built-in AST of the current rules it uses for parsing grammar.
When it reads its own syntax, it generates a parse tree called "meta data".

Meta data can be converted into rules of the same kind of AST that is used for parsing grammar.
Therefore, it can replace its own rules with new ones,
allowing custom formats to be described in a slightly different meta language.

### Meta math & mental acrobatics

When you have a document X the meta syntax Y of X is X's meta language.
When Y has the meta syntax Z, then X's meta meta language is Z.
Continuing this sequence for N steps and you get meta^N.
When a such sequence ends up in the first meta language after N steps, you can set `meta^N = meta^0`.
X's meta^0 language is X itself.

It does not matter how the language looks like.
For example, you can use a text language to develop a graphical language that describes the text language. 
When describing other languages that can not express their own grammar, the sequence terminates.

A language does not have to be Turing complete to describe itself.
Piston-Meta is not Turing complete because it looses all state when rolling back.
However, you can use it to implement a language that is Turing complete, just not run programs in itself.

More advanced languages like OMeta can play this game with programming languages.
OMeta has been implemented in Javascript to host itself plus a limited subset of Javascript.
This means you get a Javascript that can extend its own syntax with new languages!

### Meta parsing in historic perspective

Meta parsing has played an important role in computer history.
One famous example is [Tree Meta](https://en.wikipedia.org/wiki/TREE-META) that generated machine instructions directly.
It was a very central technology at the [Augmented Research Center](https://en.wikipedia.org/wiki/Augmentation_Research_Center) in the 60's.
To put it in perspective of how innovative this was: Tree Meta was developed before C.

Alan Kay has advocated this type of programming several times in his career.
Yet, it took a long time before meta parsing regained popularity.
One of the reasons might be that the whole idea is very hard to grasp.
It was only when I had a difficult problem that made me start reading about it.
Despite programming for 15 years, I was shocked to find out what our computer pioneers did.
What they produced in such short amount of time is impressive even by todays standards.

I do not think that meta parsing is a silver bullet for solving every problem.
However, it gives a different viewpoint on programming that is valuable.
Makes me think that history could easily turned out differently,
and learning to understanding this better is something I am grateful for.
