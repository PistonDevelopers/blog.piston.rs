---
layout: post
title: "Piston-Meta 0.1"
author: bvssvni
---

Piston is an open source game engine written in Rust ([http://www.piston.rs](http://www.piston.rs)).

[Piston-Meta](https://github.com/PistonDevelopers/meta) is 0.1!

Piston-Meta is a research project under the Piston project to explore the use of meta parsing
and composing for data.
Meta parsing is a technique where you use a rules to describe how to transform a text into a tree structure,
in a similar way that a parser generator can generate code for reading a text by a grammar,
except that the rules in meta parsing can be changed at run time,
or even read from a text described by the same rules.
Piston-Meta is inspired by [OMeta](https://en.wikipedia.org/wiki/OMeta) developed at
Viewpoints Research Institute in 2007.

### Why Piston-Meta?

Assuming you have a text document that you want to parse and convert into an application structure.
The normal way is to use a library written specifically for that format, for example JSON.
This has drawbacks for several reasons:

1. Changes in the document structure are entangled with the application code
2. It requires one dependency for each document format
3. Data validation is limited and hard
4. Writing parsing logic manually is error prone and composing is duplicated work
5. Error messages are bad

[Piston-Meta](https://github.com/PistonDevelopers/meta) solves these problems by using meta rules
that fits for working with data, where good error messages are important,
and where the abstraction level is the same as JSON building blocks.
For more information see https://github.com/PistonDevelopers/meta/issues/1.

### "Hello world" in Piston-Meta

Piston-Meta allows parsing into any structure implementing `MetaReader`, for example `Tokenizer`.
`Tokenizer` stores the tree structure in a flat `Vec` with "start node" and "end node" items.

```Rust
extern crate piston_meta;
extern crate range;

use std::rc::Rc;
use piston_meta::*;

fn main() {
    let text = "foo \"Hello world!\"";
    let chars: Vec<char> = text.chars().collect();
    let mut tokenizer = Tokenizer::new();
    let s = TokenizerState::new();
    let foo: Rc<String> = Rc::new("foo".into());
    let text = Text {
        allow_empty: true,
        property: Some(foo.clone())
    };
    let _ = text.parse(&mut tokenizer, &s, &chars[4..], 4);
    assert_eq!(tokenizer.tokens.len(), 1);
    if let &MetaData::String(_, ref hello) = &tokenizer.tokens[0].0 {
        println!("{}", hello);
    }
}
```

### New features

Parsing is now working! This library contains the rules and parse algorithms. For examples, see the unit tests. 

The deepest error is picked to make better error messages. When a text is parsed successfully, the result contains an optional error which can be used for additional success checks, for example whether it reached the end of a file.

- `Node` (allows reuse a rule and reference itself)
- `Number` (reads f64)
- `Optional` (a sequence of rules that can fail without failing parent rule)
- `Rule` (an enum for all the rules)
- `Select` (tries one sub rule and continues if it fails)
- `SeparatedBy` (separates a rule by another rule)
- `Sequence` (rules in sequence)
- `Text` (a JSON string)
- `Token` (expects a sequence of characters)
- `UntilAnyOrWhitespace` (reads until it hits any of the specified characters or whitespace)
- `Whitespace` (reads whitespace)

### Referencing other rules or itself

The `Node` struct is the basis for creating rules that can be referenced by other rules.
When writing the rules, use the `NodeRef::Name` and then call `Rule::update_refs` to replace them with references.
`update_refs` walks over the rules with a list of rules you want to put in, and replaces `NodeRef::Name` with `NodeRef::Ref`.

### How to use the library for game development

Just combine rules that describe how a text is interpreted. For example:

```
weapons: fork, sword, gun, lazer_beam
```

Assume we wanted to read this into an array of weapons, we could make up a simple pseudo language for helping us figuring out the rules:

```
array: ["weapons:", w!, sep(until(",") -> item, [",", w?])]
```

`w!` means required `Whitespace` and `w?` means optional `Whitespace`. The allow `->` tells what field to store that data, which is the `property` of that rule. "sep" is an abbreviation for `SeparatedBy`, "until" is for `UntilWhitespaceOrAny`. The stuff in quotes are `Token`.

Now you have 2 options:

1. Write the rules in Rust directly, which requires recompiling each time you change the data format.
2. Write the rules for parsing your pseudo language, then read the rules to read the data from a text document. This allows you to change the data format without recompiling.

The second option is useful combined with an Entity Component System (ECS) because it allows you to expand both the document format and your application with new features without recompiling. The technique is called "bootstrapping" because you are writing rules in Rust that describes how to read rules from a text document which is parsed into rules in Rust. This idea takes some time to get used to, so stick to the first option until you get familiar with the library and the meta thought process.

### Bootstrapping

It is not as hard as it sounds, because you can use the pseudo language to read data to help you write the rules in Rust for it. Just look at the rule and write down a list of what you need to describe it.

```
array: ["weapons:", w!, sep(until(",") -> item, [",", w?])]
```

1. We need a rule for reading stuff like "array:".
2. We need a rule for reading a text string "weapons:" and ",".
3. We need a rule for reading "w!" and "w?".
4. We need a rule for reading "until(<any_characters>)".
5. We need a rule for reading stuff like "[...]" that can contain 2, 3, 4 and itself 5.
6. We need a rule for the whole document that repeats 1 followed by 5, separated by new lines

Write this down in your pseudo language:

```
1. label: [text -> name, ":"]
2. text: ["text -> ", text -> name]
3. whitespace: ["w", sel("!" -> required, "?" -> optional)]
4. until_any_or_whitespace: ["until(", text -> any_characters, ")"]
5. list: ["[", sep([w?, sel(text, whitespace, until_any_or_whitespace, list)], ",")]
6. document: [sep([label, w!, list], "\n"]
```

Now, replace these rules with Piston-Meta rules (pseudo code):

```Rust
let label = Node {
    name: "label",
    body: Sequence {
        args: vec![Text { property: "name" }, Token { text: ":" }]
    }
};
let text = Node {
    name: "text",
    body: Sequence {
        args: vec![
            Token { text: "text -> " },
            Text { property: "name" }
        ]
    }
};
...
```

#### Confused?

Here is an overview of the process:

First you need some rules to parse the meta language, which you could read with `Tokenizer` and then convert to rules in Rust. These rules are then used to read the data, which then is converted to the application structure in Rust.

```
rules (for meta language) -> document (meta language describing rules for data) ->
meta data -> rules (for data) -> document (data) -> meta data -> data
```

In general, meta parsing is about this kind of transformation:

```
rules (for X) -> document (X describing Y) -> meta data -> Y
```

This transformation is composable, so you can build an infrastructure around it.

### How Piston-Meta started

When I first came on the idea of Piston-Meta, I had a problem:
I wanted to parse something, but I did not quite understand what I should parse it into.
Sometimes it gets very hard to see the whole picture of all the things that need to work together.
So I just started to write down something explaining how to parse the things I was thinking on,
and realized that this could be a self describing language.
Within an hour, I had a first draft for how to parse the rules of the language.

I remembered OMeta that did something similar, and I started reading about it to see if I could use it.
One of the key points of what meta parsing lets you do, is "bootstrapping" the rules.
A language like OMeta can parse itself and other languages and translate from one language to another.
This can be done by any general purpose programming language,
but the difference is that OMeta is specifically designed for this task.
You can quickly prototype a programming language and use OMeta to test it.
Unfortunately it is not particularly good at error messages,
which is a common reason people write parsers by hand.

There are many tools for parsing, for example parsing generators, which outputs the code
you need to parse a text for a specific grammar, for example [ANTLR](https://en.wikipedia.org/wiki/ANTLR).
Unfortunately you can not change the rules on the fly,
which is something you might want to do when working on a game.

Since I had a draft of the rules I wanted, I just started working on it.

### How does the parsing algorithm work?

There is a lot of information on the web about parsers,
but I am not an expert on these issues.
So, I will describe this in my own words.

Piston-Meta uses look-ahead rules from bottom up that might fail,
but instead of storing the data in temporarily,
it sends it to the implementor of `MetaReader`.
`MetaReader` has an associate type `State` which tells it when to roll back changes.
There is only one method, so `MetaReader` needs to check the state
and do roll back when receiving new data.

This means that if all rules starts with an expected token unique to that rule and parsing succeeds,
it will not allocate data unecessary.
A `Tokenizer` might contain extra data that needs to be truncated away with `TokenizerState` on success.

All rules can fail, so if you are putting a weird token at the end of a huge rule inside `Select`,
it will roll back all the changes and try the next rule.

Error messages can be optional and actual.
Optional error messages are useful when a rule might not fail when encountering an error.
The parent rule compares all errors, optional or not, and picks the deepest one in the document.
This is the most likely error message to be useful without any pre-knowledge.
The error message might not match with the data read by `MetaReader`.

### Performance

Piston-Meta uses `Rc<String>` in the rules, such that when parsing a document,
it only allocates for the actual data and not for strings used as keys etc.
It could in theory use `&[char]` for zero allocations, but this requires a lifetime constraint.
The performance has not been tested yet.

### Further work

Composing is the reverse operation of parsing, where you go from data to text.
This would work similar to serializing, except you can change the format it serializes at run time.
You could do this directly by printing out the text manually,
but it might not be valid for the rules you choose.
Ideally, you only need to edit the rules and both parsing and composing will work accordingly.

Making debugging easier, when there is something wrong with the rules, is also planned.
This can be solved by adding a `debug_id` to each rule so you know which rule generated an error.

Originally I thought of making a meta language syntax,
but this falls out of scope of the project for now because there are different tastes and usages.
This could be done in another library, or we could build libraries that helps with bootstrapping.
