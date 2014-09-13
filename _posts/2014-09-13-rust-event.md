---
layout: post
title: Rust-Event: The new Piston library for event logic
author: bvssvni
---

https://github.com/PistonDevelopers/rust-event

Piston is a Rust game engine developed by the open source organization PistonDevelopers.

http://www.piston.rs/

Some profiled projects are:

* [rust-image](https://github.com/pistondevelopers/rust-image) - a pure Rust image library
* [conrod](https://github.com/pistondevelopers/conrod) - an immediate mode GUI
* [rust-graphics](https://github.com/pistondevelopers/rust-graphics) - a 2D graphics library
* [hematite](https://github.com/pistondevelopers/hematite) - a simple Minecraft clone written in Rust
* [VisualRust](https://github.com/pistondevelopers/visualrust) - Visual Studio extension for Rust

We are now working on another project that we call "Rust-Event".
Rust-Event has been in experimental phase for several months, but we are now thinking we should test out the ideas for real.

Traditionally event programming has been dealt with using the observer pattern, signal/slot systems or event handlers. A weakness in these designs is the accidental occurrence of recursive event loops, that become invisible and hard to debug (Event A triggers B which triggers C which triggers A). These designs also do not scale to anything more complex than traditional GUI programming.

# Event threading

To illustrate how event threading differs from traditional event programming, here is some pseudo code:

Event handler

     fn button_click(app, args) {
         // click
     }

     event_center.add_handler(button.click(), button_click);
     event_center.update(app);

Event threading

    for e in event_iter {
        if button::draw(..., || { // click });
    }

Event threading has the down side that all interaction between widgets, that are not window events, must be performed manually. You can not "connect" them with a signal/slot system, even though you might design a system for a such purpose. In an immediate mode GUI the widget exists as function calls, so you have to solve this problem anyway. By combining event threading and immediate mode you get a more specialized structure for events instead of a general pattern.

On the upside you don't loose the stack context at each event, which means you can shift from one stack environment to another without having to build a new application structure. Experienced GUI programmers notice that event handlers are very buggy when you try to change the overall behavior of the application from one state to another. This is unfortunate because many application require special modes, such as when synchronizing, updating or loading in new data.

Another benefit with event threading is that you don't need to connect each signal to each slot. If there is a new type of event, for example multitouch, you only need to update the widget library. In Conrod, all the input received from the window is passed down to widgets that decide what input they responds to. This means you write 1 or 2 lines of code to create a new widget. You can also write simple controllers with this technique, for example a first person 3D camera.

# Behavior trees

While event threading is good enough for widgets and simple controllers, it is not good enough to express more complex things. For example, if you have an animated character, it needs to respond differently to events depending on the state of the character, such as jumping, sliding or fighting. It is even harder for game AI, where each character might do planning in parallel to achieve different goals. Still, you are interacting with these objects in the fundamental same way as for widgets, through keyboard press, mouse button clicks etc.

In a component/entity system you create one system for each behavior and turn on/off the components of an entity when want that specific behavior. One problem with this design is that you still have to express the logic of turning on/off components, which is usually done through a scripting language. The result is often not reusable across projects. While a component/entity system is very powerful, it only solves the problem half ways.

A behavior tree is a data structure that describes a behavior. For example, if you are programming a robot, then you could use the `Sequence` node to describe commands such as "move forward 3 m", "turn right", "wait 3 seconds" etc. You could use another `If` node to pick actions depending on some condition. Very soon you discover this looks like normal programming. So why not use a normal programming language? The problem is that a robot might run out of batteries, it might face an obstacle or somebody wants to turn it off safely. All these actions are "behaviors" in the sense they describe something in isolation, but put together they influence each other. Therefore, all behaviors must deal with failure and delay as an integrated part of the logic.

A problem with behavior trees is that you can't describe parallel events with side effects perfectly, because when you look at two behavior in isolation they might not behave exactly how you expected when putting them together. For example, if an event A succeeds and B succeeds, the composition of those two might succeed at time of A even if A logically happens after B, only because A was evaluated before B. This is because you don't know who succeeds first before they get evaluated! The way we solve this in Rust-Event is that behavior is expected to be correct down to the delta time update. It is basically the same problem as for physics collision where character A gets to the door before B because A was evaluated first.

Behavior trees are not only useful to describe AI logic, but also for simpler things. For example, if you want a behavior that succeeds if the user is pressing A, holding for 1 second and then releasing A, it will look like this (a bit simplified for reading purposes):

`Sequence(Pressed(A), After(Wait(1.0), Released(A)))`

If you wrote it another way that looks almost the same, you would get a different behavior:

`Sequence(Pressed(A), Wait(1.0), Released(A))`

This is different because while you wait for 1 second, the user might release the A button and the last part in the sequence never terminates.

# Actions

When you describe a behavior, you can define an action which when encountered, calls a closure that you pass to the update function.
The action is generic, so you can define as complex actions as you like as long it implements the `Clone` trait.

Here is an example that increases or decreases a counter:

    let mut counter: u32 = 0;
    let seq = Sequence(vec![Wait(1.0), Action(Inc)]);
    let mut state = State::new(seq);
    state.update(&e, |dt, action| {
        match *action {
            Inc => { counter += 1; (event::Success, dt) },
            Dec => { counter -= 1; (event::Success, dt) },
        }
    });

