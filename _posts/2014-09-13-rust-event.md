---
layout: post
title: "Rust-Event: The new Piston library for event logic"
author: bvssvni
---

Piston is a Rust game engine developed by the open source organization PistonDevelopers.

http://www.piston.rs/

Some profiled projects are:

* [conrod](https://github.com/pistondevelopers/conrod) - an immediate mode GUI
* [hematite](https://github.com/pistondevelopers/hematite) - a simple Minecraft clone written in Rust
* [piston](https://github.com/pistondevelopers/piston) - Piston, the game engine
* [rust-graphics](https://github.com/pistondevelopers/rust-graphics) - a 2D graphics library
* [rust-image](https://github.com/pistondevelopers/rust-image) - a pure Rust image library
* [VisualRust](https://github.com/pistondevelopers/visualrust) - Visual Studio extension for Rust

We are now working on another project that we call "Rust-Event".

https://github.com/PistonDevelopers/rust-event

Rust-Event has been in experimental phase for several months, now it is time to test out the ideas for real.
It is still early to say if this is the right direction, so you should consider it very unstable.

### Event Threading

Traditionally, event programming has been dealt with using event handlers (similar to observer pattern or signal/slot systems).

To illustrate how event threading differs from traditional event programming, here is some pseudo code:

Event handler

```Rust
fn button_click(app, args) {
    // click
}

event_center.add_handler(button.click(), button_click);
event_center.update(app);
```

Event threading

```Rust
for e in event_iter {
   if button::draw(..., || { // click });
}
```

Notice that event threading looks more low level, because it handles all the events through the same loop. This is how all low level window APIs deal with events.

Event threading has the down side that all interaction between widgets, that are not window events, must be performed manually. You can not "connect" them with a signal/slot system, even though you might design a system for a such purpose. However, in an immediate mode GUI the widget exists as function calls, so you have to solve this problem anyway. By combining event threading and immediate mode you get a more specialized structure for events instead of a general pattern.

On the upside you don't loose the stack context at each event, which means you can shift from one stack environment to another without having to build a new application structure. For example, if you have two different scenes, you can write them as two functions, instead of writing an application structure that supports "scenes" in general. Experienced GUI programmers notice that event handlers are very buggy when you try to change the overall behavior of the application from one state to another. This is unfortunate because many applications require special modes, such as when connecting, rendering or loading in new data.

Another benefit with event threading is that you don't need to connect each signal to each slot. If there is a new type of event, for example multitouch, you only need to update the widget library. In Conrod, all the input received from the window is passed down to widgets that decide what input they responds to. This means you write 1 or 2 lines of code to create a new widget and not hooking up 3-4 different event in a GUI editor. You can also write simple controllers with this technique, for example a [first person 3D camera](https://github.com/pistondevelopers/cam).

### AI Behavior Trees

While event threading is good enough for widgets and simple controllers, it is not good enough to express more complex behavior. For example, if you have an animated character, it needs to respond differently to events depending on the state of the character, such as jumping, sliding or fighting. What if you want do tune the behavior a little for particular scene? Then you need to go back and extend the character code to support the new behavior. It is even harder for game AI, where each character might do planning in parallel, with different sets of goals. Still, you are interacting with these objects in the same way as for widgets, through keyboard press, mouse button clicks etc.

In a component/entity system you create one system for each behavior and turn on/off the components of an entity when want that specific behavior. One problem with this design is that you still have to express the logic of turning on/off components. The result is often not reusable across projects. While a component/entity system is very powerful to organize application data, it does not solve the problem of how to express behavior.

An AI behavior tree is a data structure that describes a behavior, often through a high level language that makes it understandable, because it hides the details of how it is executed. For example, if you are programming a robot, then you could use the `Sequence` node to describe commands such as "move forward 3 m", "turn right", "wait 3 seconds" etc. You could use another `If` node to pick actions depending on some condition. Very soon you discover this looks like normal programming. So why not use a normal programming language? The problem is that a robot might run out of batteries, it might face an obstacle, or somebody wants to turn it off safely. All these actions are "behaviors" in the sense they describe something in isolation, but put together they influence each other. Therefore, all behaviors must deal with failure and delay as an integrated part of the logic.

A problem with behavior trees is that you can not describe parallel events with side effects perfectly, because when you look at two behaviors in isolation they might not behave exactly how you expected when putting them together. For example, if an event A fails after B fails in `WhenAll(A, B)`, the behavior fails at time A even if A logically happens after B, but because A was evaluated before B. This is because you don't know who succeeds first before they get evaluated, at which point it no longer matters because the side effects already happen! The way we solve this in Rust-Event is that behavior is expected to be correct down to the delta time update, but order of evaluation might determine behavior for shorter intervals. It is basically the same problem as for physics collision where character A gets to the door before B because A was evaluated first. For applications where you just want to describe the overall behavior and got a way to recover from failures, this is good enough.

Behavior trees are not only useful to describe AI logic, but also for simpler situations. For example, if you want a behavior that succeeds if the user is pressing A, holding for 1 second and then releasing A, it will look like this (a bit simplified for reading purposes):

`Sequence(Pressed(A), After(Wait(1.0), Released(A)))`

If you wrote it another way, that looks almost the same, you would get a different behavior:

`Sequence(Pressed(A), Wait(1.0), Released(A))`

This is different because while you wait for 1 second, the user might release the A button and the last part in the sequence never terminates.

Programming with behavior trees is almost like learning a new programming language.
It requires you to think differently from normal programming.
In particular, the part where failure and delay is baked into the logic, will take some time to grasp.

### Actions

When you describe a behavior, you can define an action which when encountered, calls a closure.
The action is generic, so you can define as complex actions as you like as long it implements the `Clone` trait.

Here is an example that increases or decreases a counter:

```Rust
let mut counter: u32 = 0;
let seq = Sequence(vec![Wait(1.0), Action(Inc), Wait(1.0), Action(Dec)]);
let mut state = State::new(seq);
for e in event_iter {
     state.update(&e, |dt, action| {
          match *action {
               Inc => { counter += 1; (event::Success, dt) },
               Dec => { counter -= 1; (event::Success, dt) },
          }
     });
}
```

If an action takes time, it can return `(event::Running, 0.0)` to tell that it will run for the whole update.
No matter how complex behavior you define, you only deal with the actions.
You can use the same update logic on all behaviors that have actions of same type.

With this design we hope to achieve more reuse of code across projects, independently of the environment it is executed.
