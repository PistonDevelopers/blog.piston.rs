---
layout: post
title: "Benchmark Mode"
author: bvssvni
---

Piston is a user friendly game engine written in Rust
(see [tutorial](https://github.com/pistondevelopers/piston-tutorials) and [examples](https://github.com/pistondevelopers/piston-examples) to get started).

In benchmark mode, the game loop runs as fast as it can without sleeping.

To enable benchmark mode, add `.bench_mode(true)` to the event iterator.
User input events and idle events are ignored, in case the cat walks across the keyboard,
but you can still quit any time,
either by closing the window (not the window the cat came from, but the one on your screen)
of hitting Esc if you have `WindowSettings.exit_on_esc(true)`.
Where did that cat go? (searching under the sofa, lifting carpets, looking behind the door,
checking the mailbox, finally found it in under the kitchen sink ... and THERE YOU ARE).

I mean, here we are, because this blog post is about benchmarking.

### Preparing your game for benchmarking

An example of enabling benchmarking by changing the game loop code:

```Rust
let mut bench = true;
let mut iter = 0;
let iter_end = 10000;
if bench { println!("Benchmarking..."); }
for e in window.events().bench_mode(bench) {
    if bench {
        iter += 1;
        if iter >= iter_end { break; }
        
        // Play recorded actions on the player object
        if let Some(args) = e.update_args() {
            ...
        }
    }
    
    // Game logic
    ...
}
```

Assuming your game logic is deterministic, it should do the same thing each time it runs in benchmark mode.
The game loop pretends FPS and UPS are perfect, so they come in the same order.
In normal gameplay the frame rate might slip, while updates are fixed,
which gives less accurate results.

Use `args.dt` on the update event when updating game objects:

```Rust
for e in window.events() {
    if let Some(args) = e.update_args() {
        let dt = args.dt; // <-- this should be the only time game objects know
        // Update game objects
        ...
    }
}
```

For procedural generated content, make sure that the seeds are chosen deterministically and not by clock ticks.
If you are using an external timer, the game logic will not behave the same each time when benchmarking.
This includes libraries that uses external timers. It can work for minor things, but not impact the game logic or rendering in a significant way.

When watching a game running benchmark mode, the motion will jitter occationally.
It will speed up for parts that requires less capacity and slow down for the heavy parts.

In order to do a reliable benchmark in window mode, make sure to disable animated effects in the desktop.
For example, if your wallpaper is changing image every 5 min it might affect the results.

### Running a benchmark

Here is a suggestion for how to do a practical benchmark:

1. Make sure the computer does not run other heavy tasks
2. Make sure vsync is off (this is the default when using `WindowSettings`)
2. Compile with `cargo build --release`
3. Run `time ./target/release/<myapp>` 3 times and delete the slowest one

The reason you delete the slowest one is in case some background task uses the CPU heavily.
This introduces noise, so there is a greater chance that the 2 fastest gives a more accurate result.
You need at least 2 samples to know approximately how much the performance varies between each run.

After you have run you should have some results like these:

```
real    1m9.467s
user    0m56.241s
sys 0m1.935s

real    1m9.946s
user    0m56.620s
sys 0m1.976s
```

If there is a great difference between these 2 then something is wrong.

The `real` part tells the time from start to finish.
The `user` part tells how long time is spent in user-mode code.
The `sys` part tells how long time is spent in the kernel within the process.

### Other ways of benchmarking

Profiling helps finding out which part of your code spends most time.

On OSX you can use Instruments which follows Xcode.

You can also do hierarchial real-time profiling with the [hprof](https://github.com/cmr/hprof) library.

If you get stuck, then asking in #rust-gamedev IRC channel is good place to start.
There is a link to it on the Rust gamedev subreddit [/r/rust_gamedev](http://www.reddit.com/r/rust_gamedev/)
