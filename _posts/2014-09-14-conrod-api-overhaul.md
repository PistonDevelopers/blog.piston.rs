---
layout: post
title: Conrod - New and Improved API
author: mitchmindtree
---

Following last month's [introductory post to Conrod](http://blog.piston.rs/2014/08/30/conrod-update/) there has been some excellent feedback via HN, reddit and IRC!

One of the most promising and repeatedly occurring suggestions was to switch the API over to a ["builder pattern"](http://en.wikipedia.org/wiki/Builder_pattern). So we did!

Old Style
---------

```Rust
button::draw(&mut gl, // The OpenGL instance used to draw the GUI.
             &mut ui_context, // A user interface context keeps track of state.
             unique_id, // Each widget needs it's own UI_ID.
             Point::new(x, y), // Screen location.
             width, // Rectangle width f64.
             height, // Rectangle height f64.
             Frame(frame_width, frame_color), // Or perhaps `NoFrame` if you don't want one.
             Color::new(r, g, b, a), // The color of the rectangle.
             Label("PRESS", font_size, font_color), // Here you can pass Label(...) or NoLabel.
             || {
    // Callback closure - Do your things here!
});
```

New Style with "Builder Pattern"
--------------------------------

```Rust
uic.button(0u64)
    .dimensions(90.0, 60.0)
    .position(50.0, 115.0)
    .rgba(0.4, 0.75, 0.6, 1.0)
    .frame(demo.frame_width, Color::black())
    .label("PRESS", 24u32, Color::black())
    .callback(|| demo.bg_color = Color::random())
    .draw(gl);
```

Benefits of the builder pattern include:

- All methods are optional apart from `.draw(..)`. The old style was much more verbose due to the lack of default arguments.
- All methods apart from `.draw(..)` can be called in any order which saves trying to remember arg ordering.
- Once themes and positioning-helper-methods are implemented, you could do something like this `uic.set_theme("awesome_theme")` in your load/setup, and then widgets could look more like this:
```Rust
uic.button(uiid)
    .down(padding)
    .callback(|| do_stuff)
    .draw(gl);
```
- Removes the need for the old enums that were necessary to handle defaults, etc.
- the Callable trait (which offers the callback method) is generic. If my impression of associated types is correct, this means that handling different kinds of callbacks could be done statically rather than doing them dynamically with a Callback enum which we were previously considering.
- this pattern is much more consistent with the rust-graphics api (on which Conrod depends), meaning that users hanging around the Piston ecosystem will move between APIs more easily.

Still to come
-------------

- Themes!
- Positioning methods i.e. `.down(padding)`, `right_from(other_uiid, padding)`, etc.
- More widgets.
- Optimisation and faster performance.
- [lots more](https://github.com/PistonDevelopers/conrod/issues)

We could use help on all of these things! Feel free to drop by the [github issues](https://github.com/PistonDevelopers/conrod/issues) and pitch in or add your own ideas :)

[https://github.com/PistonDevelopers/conrod](https://github.com/PistonDevelopers/conrod)

[mitchmindtree](https://github.com/mitchmindtree)

