---
layout: post
title: "Event loop now uses piston-current"
author: bvssvni
---

You can now decide between the following ways to write the event loop:

```Rust
// Move window by value (this prevents you from using the window elsewhere).
for e in Events::new(window) {
    ...
}

// Use shared reference (this allows you to use the window elsewhere).
let window = RefCell::new(window);
for e in Events::new(&window) {
    ...
}

// Use current window (the window must be set as current object).
for e in Events::new(current::UseCurrent::<Window>) {
    ...
}

// Specify usage.
let window = RefCell::new(window);
let usage = current::Use(&window);
for e in Events::new(usage) {
    ...
}
```

This is powered by the new [Piston-Current](https://github.com/pistondevelopers/current) library.

To learn more, read [Best coding practices with current objects](https://github.com/PistonDevelopers/current/issues/15)
