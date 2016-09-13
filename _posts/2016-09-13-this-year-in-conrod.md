---
layout: post
title: "This Year In Conrod"
author: mitchmindtree
---

[Conrod](https://github.com/PistonDevelopers/conrod) has landed some
long-desired features and overhauls over the past year. I've been excited about
writing this post for the past few months now, though I have managed to
continuously put it off until we "just land that one extra special feature!".

I realise that there's probably always going to be one more exciting feature, so
I think it is about time for what is turning into the annual Conrod update!


[![imgur](http://i.imgur.com/6KTixap.png)](http://i.imgur.com/6KTixap.png)

*a WIP of a personal project that uses Conrod for a non-trivial GUI*


Highlights
----------

Here are some of the highlights before we dive into the details:

- Dropped from 11 crate dependencies to 4.
- Greatly improved the rendering API (now renders as a list of
  `render::Primitives` making it possible to batch drawing).
- Introduced a new event system which interprets high-level UI events from raw
  window events.
- Simplified the back-end agnostic story.
- Changed from FreeType to [RustType](https://github.com/dylanede/rusttype).
- Added lots of new useful widgets (multi-line `TextEdit`, `List`, `Scrollbar`
  and more).
- Simplified and improved many of the existing widgets.
- Removed the `.react` closure convention in favour of having widgets return a
  `Widget::Event` associated type, greatly improving control-flow.
- Simplified implementation of custom widgets.
- Began work on The Guide.

I'll address each of these in roughly the same order in more detail below.


Simplifying "Backend Agnostic"
------------------------------

The changes that I am most excited about are those that relate to simplification
of conrod's backend agnostic story.

To clarify, when I say "backend agnostic" I'm talking about enabling people to
use Conrod with different window+graphics combinations - a commonly requested
feature due to the variety of growing window and graphics libraries throughout
the ecosystem. For example, some users wish to use sdl2 for the window context
and gfx for the graphics, while others want to use glutin for the window context
and glium for the graphics, etc.

### **The Old Problem**

Implementing a back-end for conrod used to require numerous traits from various
crates around the Piston ecosystem. This would frequently cause problems with
version conflicts, strange error messages and highly complex generic types.

The window and event traits required wrapping the entire event loop in a new
type, making it very intrusive to introduce into existing projects.

The graphics trait bound required that every primitive widget (lines, shapes,
text, images) needed a unique draw call, which quickly became impractical for
use with conrod's modular, granular approach to widgets (the GUI above is
composed of over 3000 unique widgets). Even after applying some culling
optimisations (only drawing visible primitives), the number of opengl draw calls
was clearly showing itself as a severe bottleneck when profiling.

The primary reason that conrod originally took this approach is that there were
no other abstractions for drawing simple 2D graphics at the time. As a newbie to
graphics programming, these piston traits at least made it possible for me to
get started. However, in the following two years I have learned a lot and
finally managed to find the time and solution to address these issues.

### **The New Solution**

The key to simplifying all of this was realising that there are two primary ways
in which the UI interacts with an application loop.

- Receive window events as input.
- Produce graphics as output.

I decided to abandon the wrapper-like, trait-heavy approach in favour of
treating conrod as a pipeline that takes a new `event::Raw` type as input and
renders to a list of depth-sorted `render::Primitives` as output.

**`event::Raw` as input -> `Ui` -> `render::Primitives` as output**

The `event::Raw` type describes a set of basic events (window resize, mouse
press, key release, etc) that conrod can use to interpret higher level events
(widget A was double clicked, widget B has captured the keyboard, widget C was
dragged, etc) which may then be delivered to widgets. When a widget requests for
events, it is only delivered those that apply to it unless it specifically
requests for global events.

A
[**render::Primitive**](http://docs.piston.rs/conrod/conrod/render/struct.Primitive.html)
describes a set of [**6 graphics
primitives**](http://docs.piston.rs/conrod/conrod/render/enum.PrimitiveKind.html),
from which all conrod GUIs can be drawn: `Rectangle`, `Lines`, `Polygon`,
`Text`, `Image` and `Other`. The first 5 are self-explanatory. `Other` allows
for custom user primitives which conrod does not yet support natively (i.e.
video playback). Calling `ui.draw()` produces a depth-sorted list of
`render::Primitive`s, allowing the user to batch draw calls or render to some
other format that I'm unaware of. Calling `render::Primitives::owned` creates an
owned instance of the primitive list which can be sent across threads or stored
for rendering at some other time.

This means that there are now only two steps a user must do to integrate conrod
into their project:

1. Convert their window events into `conrod::event::Raw`s.
2. Draw the `conrod::render::Primitives` using their graphics back-end of choice.

In order to reduce some of the boilerplate involved for users, I've added cargo
features that provide implementations of these two steps for a handful of
popular back-ends (piston, glutin, glium (WIP)) and hope to add more in the
future (winit, glfw, sdl2, gfx).


From FreeType to RustType
-------------------------

For many folks, Conrod was a non-starter due to it's dependency on FreeType - a
font rendering library implemented in C. It particularly caused [a lot of
issues](https://github.com/PistonDevelopers/conrod/issues/546) for Windows
users. *Cue regular thanks to windowsbunny for the help!*

By switching to [RustType](https://github.com/dylanede/rusttype), we were able
to complete [The Road to Pure
Rust](https://github.com/PistonDevelopers/conrod/issues/321), add support for
kerning and finally require nothing but `cargo build` to build conrod from
scratch.

RustType has been a real pleasure to work with and IMO is a great example of a
backend agnostic rendering API. It was the RustType rendering API that really
inspired conrod's pipeline-esque rendering solution described in the previous
section. A massive thanks to [@dylanede](https://github.com/dylanede) for both
the lib and the inspiration! I believe dylanede also has a GUI lib in the
works, so I recommend keeping an eye out! I sure will be.


More Widgets!
-------------

New widgets include:

- [**primitive** widgets](http://docs.piston.rs/conrod/conrod/widget/primitive/index.html):
  A variety of primitive graphical elements which can be used as building blocks
  for more complex widgets. These include:
  - [**Image**](http://docs.piston.rs/conrod/conrod/widget/primitive/image/struct.Image.html)
  - [**Text**](http://docs.piston.rs/conrod/conrod/widget/primitive/text/struct.Text.html)
  - [**Rectangle**](http://docs.piston.rs/conrod/conrod/widget/primitive/shape/rectangle/struct.Rectangle.html)
  - [**Polygon**](http://docs.piston.rs/conrod/conrod/widget/primitive/shape/polygon/struct.Polygon.html)
  - [**Oval**](http://docs.piston.rs/conrod/conrod/widget/primitive/shape/oval/struct.Oval.html)
  - [**Circle**](http://docs.piston.rs/conrod/conrod/widget/primitive/shape/circle/struct.Circle.html)
  - [**Line**](http://docs.piston.rs/conrod/conrod/widget/primitive/line/struct.Line.html)
  - [**PointPath**](http://docs.piston.rs/conrod/conrod/widget/primitive/point_path/struct.PointPath.html)

  All provided conrod widgets are now built from these primitives.

  [![image.rs](http://i.imgur.com/a310A9h.png)](http://i.imgur.com/a310A9h.png)
  [![text.rs](http://i.imgur.com/Vp4poXS.png)](http://i.imgur.com/Vp4poXS.png)
  [![primitives.rs](http://i.imgur.com/zPQV4ck.png)](http://i.imgur.com/zPQV4ck.png)

- [**TextEdit**](http://docs.piston.rs/conrod/conrod/widget/text_edit/struct.TextEdit.html):
  A multi-line text editing widget that allows for:
  - Auto-wrapping via either character or whitespace.
  - Left, center and right justification.
  - Control over line-spacing.
  - Block selection.

  [![text_edit.rs](http://i.imgur.com/IVO5ORT.png)](http://i.imgur.com/IVO5ORT.png)

  The **TextBox** widget is now a thin wrapper around the **TextEdit** widget.
- [**List**](http://docs.piston.rs/conrod/conrod/widget/list/struct.List.html):
  A widget that abstracts common logic between all list-like widgets including:
  - Generating a dynamic number of `widget::Id`s.
  - Automatic positioning and sizing of items.
  - Scrollability.
  - Optimised item instantiation i.e. only instantiating visible items. Very
    useful for massive lists (100+ items).
- [**ListSelect**](http://docs.piston.rs/conrod/conrod/widget/list_select/struct.ListSelect.html):
  Extends the `List` widget providing an abstraction over item selection.
  Includes support for both single and multiple item selection. Used internally
  within the **DropDownList** widget and the new **FileNavigator** widget.

  [![list_select.rs](http://i.imgur.com/gJGx6Ds.png)](http://i.imgur.com/gJGx6Ds.png)

- [**FileNavigator**](http://docs.piston.rs/conrod/conrod/widget/file_navigator/struct.FileNavigator.html):
  An OS X Finder inspired file navigator with adjustable columns,
  auto-scrolling, multiple selection and a variety of useful events.

  [![file_navigator.rs](http://i.imgur.com/DLreIXl.png)](http://i.imgur.com/DLreIXl.png)

- [**Scrollbar**](http://docs.piston.rs/conrod/conrod/widget/scrollbar/struct.Scrollbar.html):
  Used for manually scrolling some scrollable widget along the *x* or *y* axes.
  Supports auto-hiding.
- [**PlotPath**](http://docs.piston.rs/conrod/conrod/widget/plot_path/struct.PlotPath.html):
  Allows generating a **PointPath** from some given function *X -> Y*, mapped to
  its dimensions (This is how the waveform is drawn in the top image).

  [![plot_path.rs](http://i.imgur.com/uf0edt3.png)](http://i.imgur.com/uf0edt3.png)


Removing `.react` closures - Adding `Widget::Event`
---------------------------------------------------

Previously, the conventional way for a Widget to support reacting to certain
interactions was by implementing a `.react` closure. Something like this:

```rust
Button::new()
    .react(|event| match event {
        button::Event::Pressed(mouse_button) => /* react to press, may get called multiple times */,
        button::Event::Released(mouse_button) => /* react to release, may get called multiple times */,
        button::Event::Clicked(mouse_button) => /* react to click, may get called multiple times */,
    })
    .set(ID, &mut ui);
```

There are three primary issues that arose from this convention:

- **Difficulty with control flow**:
  - No easy way to return data from the closure.
  - Awkward Result handling (can't use `try!`).
- **Ownership issues**: especially when using `FnMut` closures.
- **Awkward builder method**: We can't write custom `Fn`/`FnMut`/`FnOnce` types
  to use as default `.react` functions for widgets, meaning the user call the
  builder method with an empty closure even if they don't care about its
  interactions.

We were able to solve all of these issues by introducing a `Widget::Event` type,
returned via the `Widget::update` method. The above example now looks more like
this:

```rust
for event in Button::new().set(ID, &mut ui) {
    match event {
        button::Event::Pressed(mouse_button) => /* react to press */,
        button::Event::Released(mouse_button) => /* react to release */,
        button::Event::Clicked(mouse_button) => if mouse_button.is_left() {
            try!(load_file()); // Using `try!` like this would not have been possible using the `react` convention.
        },
    }
}
```

For more details and reasoning behind this change, see [this
issue](https://github.com/PistonDevelopers/conrod/issues/741).


Simplifying Custom `Widget` impls
---------------------------------

Over the past year, the process of implementing custom
[**Widget**](http://docs.piston.rs/conrod/conrod/widget/trait.Widget.html)s has
been simplified significantly:

- You can now build new widgets by instantiating other widgets. All
  non-primitive widgets in conrod now do this, so you can see [their
  implementations](https://github.com/PistonDevelopers/conrod/tree/master/src/widget)
  for examples of this.
- The
  [**widget_ids!**](http://docs.piston.rs/conrod/conrod/macro.widget_ids.html)
  macro simplifies the generation of identifiers for child widgets.
- The
  [**widget_style!**](http://docs.piston.rs/conrod/conrod/macro.widget_style.html)
  macro allows for generating a collection of optional styling parameters for a
  widget, with automatic fallback to parameters specified within the `Ui`'s
  `Theme` or some other custom expression.
- Several unnecessary `Widget` trait methods [have been
  removed](https://github.com/PistonDevelopers/conrod/pull/760).
- Rather than having to manually track raw input, widgets can now receive
  auto-generated high level widget events:

  ```rust
  for event in ui.widget_input(id).events() {
      match event {
          event::Widget::Click(click) => { ... },
          event::Widget::DoubleClick(click) => { ... },
          event::Widget::Drag(drag) => { ... },
          event::Widget::Scroll(scroll) => { ... },
          // etc
      }
  }
  ```

  See an enumeration of the available events
  [here](http://docs.piston.rs/conrod/conrod/event/enum.Widget.html). This
  method of listening for events has been especially nice as it also simplifies
  the process of listening to relative (child, parent, grandchild, etc) widget
  events, i.e. `ui.widget_input(child_id).clicks().left()`.
  
  Widgets can also listen for globally occurring events if necessary.

  ```rust
  for event in ui.global_input().events() {
      match event {
          event::Ui::Press(widget_id, press) => { ... },
          event::Ui::WidgetCapturesKeyboard(widget_id) => { ... },
          // etc
      }
  }
  ```

  See an enumeration of these global events
  [here](http://docs.piston.rs/conrod/conrod/event/enum.Ui.html).


The Guide
---------


The first couple of chapters [have been
written](http://docs.piston.rs/conrod/conrod/guide/index.html), though are
already getting a little stale following all of the advancements mentioned
above. The Guide is one of my top priorities for the near future now that conrod
is less likely to go through any more major API overhauls.

If you are new to rust and are interested in contributing to conrod, this is
probably the most valuable area in which one could do so! I'd be more than happy
to mentor, so feel free to leave an issue or catch me on #rust if you are
interested.


TODO
----

Here are some of the issues that are closest on my radar.

- Complete the glutin_glium.rs example as an example of batched rendering. Once
  this is done, abstract the boilerplate into the `glium` feature and change the
  rest of the examples to use this instead of piston_window in order to
  demonstrate greater efficiency.
- Add a gfx-rs example.
- Consider the trade-offs of moving these backend cargo features into unique
  crates within the same repo. This may help to stabilise the core of conrod
  sooner as progress on the stability of the compatible back-ends cannot be
  guaranteed.
- Remove the num crate dependency.
- *Finish The Guide*.
- Address all existing bugs.
- Start thinking about how to extend conrod to support multiple native windows.
- Investigate how much work would be involved in providing a set of
  "native look and feel" widgets.

Otherwise, you can get an idea of what remains by checking out the [**1.0.0
milestone**](https://github.com/PistonDevelopers/conrod/milestone/2).


Contributors
------------

A lot of the above would not have happened without help from the following:

- @christolliday for his work on `TextEdit`, fixing loads of bugs, adding
  new-line insertion and a bunch of other useful key commands.
- @Boscop for also adding some handy `TextEdit` key commands.
- @tmerr for some `TextEdit` fixes and for inspiring simplification of the
  widget identifier system.
- @Hyperchaotic for kicking off the `ListSelect` widget and helping to hide
  hidden files in the `FileNavigator`.
- @pierrechevalier83 for adding support for `Image`s on `Button`s.
- @|||Shaman||| for moving `Ui` construction over to the builder method
  convention.
- @psFried for their 2kloc PR that kicked off the new event system.
- Many other broken-link-fixers, spell-checkers and find-and-replacers :)


*Written as of Conrod version 0.44.0*
