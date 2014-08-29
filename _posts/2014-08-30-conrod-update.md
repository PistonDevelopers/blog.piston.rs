---
layout: post
title: Conrod - A 100% Rust GUI Library
author: mitchmindtree
---

[Conrod](https://github.com/PistonDevelopers/conrod) is a super-young, "immediate-mode", graphical user interface library written entirely in Rust! Before I bore you with the details, here's a demonstration of it in action.

<iframe width="420" height="315" src="//www.youtube.com/embed/n2UrjogA0j0" frameborder="0" allowfullscreen></iframe>

*click click clickity click...*

The Rust Story
--------------

About two or three months ago I landed on the [rust-lang.org](http://www.rust-lang.org/) home page after following a well-hyped HN link. Just so you know, I'm currently in the middle of developing a large real-time interactive generative music system and there's *no way in the world* I'd have time to go around willy-nilly checking out new programming languages! **No. Way.** But then I read the headline...

> "Rust is a systems programming language that runs blazingly fast, prevents almost all crashes*, and eliminates data races."

So here I am about two or three months later - 95% through porting all of my slave-laboured C++ code to [Rust](http://www.rust-lang.org/), and it remains one of the best decisions I can remember making! (it should be noted that none of my laundry has been eaten... yet).

The Conrod GUI
--------------

Conrod itself is only about two weeks old, so go easy! It still requires lots of refinement and optimisation, however I think it's showing a good amount of promise. This is also the first time I've had a go at "immediate-mode" UI - to be honest when I first heard of the idea I thought it sounded quite rubbish...

> "How on earth would you store complex widget state without having any objects!?"

and

> "Surely it would have to be so slow, having to create the widget every frame!?".

Only after I started to have a play with the idea did I realise how well the "immediate-mode" approach is suited to Rust's functional-esque style! The algebraic data types turned out to be perfect for both caching the necessary widget state and creating diverse yet concise `draw` signatures. Performance hasn't yet been an issue, and I no longer see any good reason why it should be!

API Code
--------

This is what drawing a Button looks like at the moment.

```Rust
// Inside our render loop...

button::draw(&render_args, // Screen width and height.
             &mut gl, // The OpenGL instance used to draw the GUI.
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

and this is what drawing a matrix of Toggles looks like.

```Rust
widget_matrix::draw(cols, // The number of columns.
                    rows, // The number of rows.
                    Point::new(x, y) // Screen location for the matrix.
                    mat_width, // Width of the matrix.
                    mat_height, // Height of the matrix.
                    |num, col, row, position, width, height| { // This is called once for each widget.

    // Now we draw the widgets with our callback params!
    toggle::draw(&render_args, &mut gl, &mut ui_context, ui_id + num,
                 position, width, height,
                 Frame(frame_width, frame_color), // Rectangle frame.
                 Color::new(r, g, b, a); // Rectangle color.
                 NoLabel, // We don't feel like having a label.
                 my_value_matrix[col][row], // Toggle value (true / false)
                 |new_val| { // Our callback with the new value!
        my_value_matrix[col][row] = new_val;
    });

});
```

There's still [heaps of room for improvement and lots to be done](https://github.com/PistonDevelopers/conrod/issues) however it's definitely approaching a usable state.

PistonWorks - Open Source Rust Libraries!
-----------------------------------------

It should be noted that there is no way Conrod would exist without the open source [PistonWorks](https://github.com/PistonDevelopers) collective! Conrod has several [Cargo](http://crates.io/) dependencies, and they are all Piston projects! Come and visit us / join in at [GitHub] (https://github.com/PistonDevelopers) or drop by on Mozilla's #rust-gamedev IRC channel where most of us are normally lurking.

https://github.com/PistonDevelopers

https://github.com/PistonDevelopers/conrod

