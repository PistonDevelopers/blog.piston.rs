---
layout: post
title: Another Year With Conrod
author: mitchmindtree
---

[Conrod](https://github.com/PistonDevelopers/conrod) has been getting some painstaking love and attention over the past few months, and considering it has been about a year since [the last post](http://blog.piston.rs/2014/09/14/conrod-api-overhaul/) I thought it was about time for a quick update!

In summary, the main changes are:

- Working with stable rust and on crates.io
- Conrod now supports custom widgets!
- We landed a `widget_ids!` macro for automatically generating unique `WidgetId`s.
- A [massive list](https://github.com/PistonDevelopers/conrod/blob/master/src/position.rs#L105) of new positioning methods.
- Tabbed canvasses, draggable canvasses, auto-layout canvas splits (see the [canvas example](https://github.com/PistonDevelopers/conrod/blob/master/examples/canvas.rs)).
- Themes.
- Generalised opt-in scrolling (have a widget automatically become scrollable if its children occupy a greater area than its scrollable area).
- Changed our internal cache datastructure from a Vec to a Graph.
- Much faster performance (still a lot of room for improvement).


Custom Widgets
--------------

Conrod now properly supports custom widgets via [the `Widget` trait](https://github.com/PistonDevelopers/conrod/blob/master/src/widget/mod.rs#L59). This means it should be easy to build and share third party widgets and have them all work together within the same conrod `Ui` instance.

All of the widgets that come with Conrod are implemented using this trait so they should make for decent examples. jarret was also kind enough to contribute a heavily annotated example implementing a custom circular button widget (see it [here](https://github.com/PistonDevelopers/conrod/blob/master/examples/custom_widget.rs)).


`widget_ids!`
-------------

As conrod is an immediate mode GUI framework, it requires that each widget instantiation has its own unique ID. this ID is used to look up cached state within the `Ui` between updates.

In the past, users have been required to do something like this:

```Rust
const SLIDER: WidgetId = 0;
const DROP_DOWN_LIST: WidgetId = SLIDER + 1;
const TOGGLE_MATRIX: WidgetId = DROP_DOWN_LIST + 1;
const LABEL: WidgetId = TOGGLE_MATRIX + ROWS * COLS + 1;
```

which was a pain. Now with the `widget_ids!` macro the above example would look like this:

```Rust
widget_ids! {
    SLIDER,
    DROP_DOWN_LIST,
    TOGGLE_MATRIX with ROWS * COLS,
    LABEL,
}
```

Not only is this much easier and more concise, it is also a lot safer when refactoring or adding/removing widgets.


Positioning Methods
-------------------

The idea here is to have an extensive enough list so that a user should never need to place a widget using absolute coordinates. Although we still might have a way to go, I think we are getting close!

You can peruse the whole list [here](https://github.com/PistonDevelopers/conrod/blob/master/src/position.rs#L105).

There are a few different "categories" of positioning methods here:

- Directional (`down`, `right_from`, etc). These describe positioning a widget in some direction either relative to the last set widget or some given specific widget.
- Placement (`middle_of`, `top_left_of`, etc). These are for placing a widget at some place on top of another widget. These are often useful when placing a widget on top of some Canvas widget for example.
- Alignment (`align_top`, `align_left_of`, etc). These allow you to align your widget with either the last set widget or some given specific widget.
- Absolute (`xy`, `point`). Position a widget with absolute coordinates.
- Relative (`relative_xy`, `relative_to`, etc). Set the position of a widget with absolute coordinates relative to some other widget.


Canvasses
---------

These are widgets whose primary purpose revolves around assisting in the positioning of other widgets.

`Split` is a wrapper around the `Canvas` widget which automatically calculates their layout by segmenting the window into a tree of splits. This was heavily inspired by eddyb who did something similar with his IDE project.

`Tabs` is a widget that internally generates a `Canvas` for each identifier given, and only displays the last that was selected.

Widgets also offer a `floating` method for becoming "detached" from the gui graph and making them float above other widgets in a draggable manner (useful as a pop-up or alert window). This is particularly useful on the `Canvas` widget.

Demonstrations of all of these can be found within the [`canvas.rs` example](https://github.com/PistonDevelopers/conrod/blob/master/examples/canvas.rs).


Themes
------

Writing your own `Theme` allows you to specify all your default styling and layout behaviour, saving you from calling the same styling methods on widgets again and again. We don't have any proper examples for writing custom themes yet, but you can see the `theme` module [here](https://github.com/PistonDevelopers/conrod/blob/master/src/theme.rs#L16).


Scrolling
---------

```Rust
.scrolling(true)
.horizontal_scrolling(true)
.vertical_scrolling(true)
```

These methods allow for opt-in scrolling on widgets. For example, if you were to call `.vertical_scrolling(true)` on a `Canvas` widget, it would automatically calculate the height occuppied by the `Canvas`' children widgets, compare it to the height of the scrollable area and determine whether or not the area should become scrollable in order to fit all child widgets.

These methods are particularly useful to call on `Canvas` widgets, or any widget that is designed to be a parent widget of many other child widgets.


Performance
-----------

We've added a `.draw_if_changed` method which only re-draws all widgets if there has been some visual change to one or more of the widgets (or if `needs_redraw` has been called manually). This took the `all_widgets` example from about 25% cpu down to ~7% on my mbp in --release mode. We have some ideas for [optimising this further](https://github.com/PistonDevelopers/conrod/issues/537) but haven't had time yet. There are also still a few unnecessary allocations within the Graph's `Element` construction that should be easily removed once time permits.

We've also included `.element` and `.element_if_changed` methods for producing the entire renderable GUI as a single `Element`. This allows for easy interop with the [elmesque](https://github.com/mitchmindtree/elmesque) graphics layout crate and in general a little more control over the rendering of your GUI.


TODO
----

Some things that come to mind are:
- Write a Guide! (issue [here](https://github.com/PistonDevelopers/conrod/issues/505))
- Support `set`ting widgets in either update *or* render stages, rather than forcing the user to do so in the render stage (issue [here](https://github.com/PistonDevelopers/conrod/issues/509)).
- Multi-line text widgets (for both viewing and editing) (issue [here](https://github.com/PistonDevelopers/conrod/issues/466), [here](https://github.com/PistonDevelopers/conrod/issues/342), [here](https://github.com/PistonDevelopers/conrod/issues/71) and [here](https://github.com/PistonDevelopers/conrod/issues/62)).
- Context Menu (aka Clickdown menu) (issue [here](https://github.com/PistonDevelopers/conrod/issues/394)).
- Have custom widgets be able to add child widgets without occupying the public WidgetId space. The main step in this was implementing the new internal graph cache datastructure, now we just need a method.
- Support for images in general (on buttons, toggles, viewers, etc). The main problem is that I am currently unsure the best way to generically support this.
- [Lots more stuff](https://github.com/PistonDevelopers/conrod/issues).


We also have a milestone for [what we'd like to see in a 1.0 conrod release](https://github.com/PistonDevelopers/conrod/milestones/1.0.0). If you're interested in contributing please drop by - any help is more than welcome!

Come and visit [conrod](https://github.com/PistonDevelopers/conrod).

