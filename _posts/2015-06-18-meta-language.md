---
layout: post
title: "Meta Language"
author: PistonDevelopers
---

"How do you feel today Piston-Meta 0.6.6?"

"Good! Like I can parse everything!"

[Piston-Meta](https://github.com/PistonDevelopers/meta) is new Domain Specific Language (DSL) parsing library (written in Rust) for human readable formats.
It gives nice error messages, more higher level than regular expressions, easy to use, and reads JSON strings.
When combining complicated rules containing new lines, but separated by new lines, it "just works".
It is not indention sensitive, as it is designed for data and configuration files typical in game development.
Transformation happens to a flat tree structure stored in a `Vec`, and uses reference counting on the strings that are copied from the rules.
You get fewer allocations with better data validation.
Rules are dynamic and can be changed at runtime, designed for systems that never need to shut down for maintenance.

Since an example can say more than thousand words, here is "hello world" in Piston-Meta:

```Rust
extern crate piston_meta;

use piston_meta::*;

fn main() {
    let text = "say \"Hello world!\"";
    let rules = "1 \"rule\" [\"say\" w! t?\"foo\"]";
    // Parse rules with meta language and convert to rules for parsing text.
    let rules = bootstrap::convert(
        &parse(&bootstrap::rules(), rules).unwrap(),
        &mut vec![] // stores ignored meta data
    ).unwrap();
    let data = parse(&rules, text);
    match data {
        Ok(data) => {
            assert_eq!(data.len(), 1);
            if let &MetaData::String(_, ref hello) = &data[0].1 {
                println!("{}", hello);
            }
        }
        Err((range, err)) => {
            // Report the error to standard error output.
            ParseStdErr::new(&text).error(range, err);
        }
    }
}
```

When you run this, you should get the following text on standard output:

```
Hello world!
```

Try changing the text to `don't say "Hello world!"`, and you get an error:

```
Error #1001, Expected: `say`
1: don't say "Hello world!"
1: ^
```

Try changing the text to `say quickly "Hello world!"`, and you get an error:

```
Error #1003, Expected text
1: say quickly "Hello world!"
1:     ^
```

Try changing the text to `say "Hello world!" 3 times`, and you get an error:

```
Error Expected end
1: say "Hello world!" 3 times
1:                   ^
```

The rule we used to parse `say "Hello world!"` in the "hello world" example consisted of one line:

```
1 "rule" ["say" w! t?"foo"]
```

`1` is a number that gets multiplied with 1000 and used as seed for `debug_id` in the rule.
For example `Error #1003, Expected text` tells you that the rule causing parsing to fail is located in `1`.
This is useful when you are developing rules for a new text format.

After the number `1`, we have `"rule"` which is used internally to name rules.
Rules can reference other rules.
If we wanted to reference `"rule"` from another rule, we could use `@"rule"` to use it.

The square brackets `[]` is a sequence rule, which processes each sub rule from left to right.

1. A token rule `"say"`, failing if there is no `say` in the text
2. A required whitespace rule `w!`
3. A text rule `t?"foo"`, which allow an empty string and gives it a name `"foo"`

### Some tiny examples

`1 "numbers" l($"num")`:

```
1001
2002
3003
```

`1 "number" l($_"num")`:

```
1_001
2_002
3_003
```

`1 "laughter" r?("ha")`:

```
hahahahahaha
```

`1 "forced laughter" r!("ha")`:

```
Error #1001, Expected: `ha`
1: (:-|)
1: ^
```

`1 "evil laughter" ["mua" r!("ha")]`:

```
muahahahaha
```

`1 "lines of two kinds of laughter" l({r?("ha") r?("hi")})`:

```
hihihihihihihihihi
hahahaha
hihihi
hi
hahahahahahahahahaha


hi
ha
```

```
1 "farm animals separated by commas" s?(["," w?]){
  {"cow" "pig" "hen" "sheep"}
}
```

```
sheep, cow, pig, hen,hen,hen, cow, cow, pig
```

The `s?(..){..}` rule means separat `{..}` by `(..)`.
`?` means it can occur zero times.

`1 "numbers separated by commas, allow trailing comma but no whitespace" [s?.(","){$}]`:

```
1,2,3,4,
```

If you write `s?.` or `s!.` the separation rule is allowed at the end.

`1 "can't we all be friends?" l(["can't we all be friends?" w! {"yes""said_yes" "no"!"said_yes"}])`:

```
can't we all be friends? no
can't we all be friends? yes
```

The first line sets `said_yes` to `false` and the second line sets `said_yes` to `true`.

`1 "you can say you love me, but you don't have to" ?"I love you"`:

``

A rule that starts with `?` is optional.

### How does it work?

The rules are used to parse a text document, and you get a list of tokens or an error.
Here is the signature of the `parse` function:

```Rust
pub fn parse(
    rules: &[(Rc<String>, Rule)],
    text: &str
) -> Result<Vec<(Range, MetaData)>, (Range, ParseError)>
```

The `Range` tells which characters the data was read from.
`MetaData` is an enum defined as:

```Rust
pub enum MetaData {
    /// Starts node.
    StartNode(Rc<String>),
    /// Ends node.
    EndNode(Rc<String>),
    /// Sets bool property.
    Bool(Rc<String>, bool),
    /// Sets f64 property.
    F64(Rc<String>, f64),
    /// Sets string property.
    String(Rc<String>, Rc<String>),
}
```

All you have to do in order to use it with your own library/application,
is to convert from `MetaData` to the structure you want in Rust.
Look in [convert.rs](https://github.com/PistonDevelopers/meta/blob/master/src/bootstrap/convert.rs) for example code.

(We probably will make this easier at some point)

### Self syntax

The meta language for Piston-Meta can be bootstrapped using its own rules.

```
opt: "optional"
inv: "inverted"
prop: "property"
any: "any_characters"
seps: "[]{}():.!?\""
1 "string" [..seps!"name" ":" w? t?"text"]
2 "node" [$"id" w! t!"name" w! @"rule""rule"]
3 "set" {t!"value" ..seps!"ref"}
4 "opt" {"?"opt "!"!opt}
5 "number" ["$" ?"_""underscore" ?@"set"prop]
6 "text" ["t" {"?""allow_empty" "!"!"allow_empty"} ?@"set"prop]
7 "reference" ["@" t!"name" ?@"set"prop]
8 "sequence" ["[" w? s!.(w!) {@"rule""rule"} "]"]
9 "select" ["{" w? s!.(w!) {@"rule""rule"} "}"]
10 "separated_by" ["s" @"opt" ?".""allow_trail"
  "(" w? @"rule""by" w? ")" w? "{" w? @"rule""rule" w? "}"]
11 "token" [@"set""text" ?[?"!"inv @"set"prop]]
12 "optional" ["?" @"rule""rule"]
13 "whitespace" ["w" @"opt"]
14 "until_any_or_whitespace" [".." @"set"any @"opt" ?@"set"prop]
15 "until_any" ["..." @"set"any @"opt" ?@"set"prop]
16 "repeat" ["r" @"opt" "(" @"rule""rule" ")"]
17 "lines" ["l(" w? @"rule""rule" w? ")"]
18 "rule" {
  @"whitespace""whitespace"
  @"until_any_or_whitespace""until_any_or_whitespace"
  @"until_any""until_any"
  @"lines""lines"
  @"repeat""repeat"
  @"number""number"
  @"text""text"
  @"reference""reference"
  @"sequence""sequence"
  @"select""select"
  @"separated_by""separated_by"
  @"token""token"
  @"optional""optional"
}
19 "document" [l(@"string""string") l(@"node""node") w?]
```

This is how you check rules when by bootstrapping them twice and compare them:

```Rust
extern crate piston_meta;

use piston_meta::parse;
use piston_meta::bootstrap;
use std::fs::File;
use std::io::Read;
use std::path::PathBuf;

fn main() {
    let rules = bootstrap::rules();
    let self_syntax: PathBuf = "assets/self-syntax.txt".into();
    let mut file_h = File::open(self_syntax).unwrap();
    let mut source = String::new();
    file_h.read_to_string(&mut source).unwrap();
    let res = parse(&rules, &source).unwrap();
    let mut ignored1 = vec![];
    let rules1 = bootstrap::convert(&res, &mut ignored1).unwrap();
    println!("ignored1 {:?}", ignored1.len());
    let res = parse(&rules1, &source).unwrap();
    let mut ignored2 = vec![];
    let rules2 = bootstrap::convert(&res, &mut ignored2).unwrap();
    println!("ignored2 {:?}", ignored2.len());
    let _ = parse(&rules2, &source).unwrap();
    assert_eq!(rules1, rules2);
    println!("Bootstrapping succeeded!");
}
```

If don't like the meta language, you can change the self syntax,
print out the rules with `println!("{:?}", rules)`, fix it so it compiles in Rust,
and then use the bootstrap test on your new meta language.
