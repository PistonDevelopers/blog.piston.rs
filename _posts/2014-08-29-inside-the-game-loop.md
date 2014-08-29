---
layout: post
title: Inside the Game Loop
author: bvssvni
---

```Rust
/// Returns the next game event.
fn next(&mut self) -> Option<GameEvent> {
    loop {
        match self.state {
            RenderState => {
                if self.game_window.should_close() { return None; }

                let start_render = time::precise_time_ns();
                self.last_frame = start_render;

                let (w, h) = self.game_window.get_size();
                if w != 0 && h != 0 {
                    // Swap buffers next time.
                    self.state = SwapBuffersState;
                    return Some(Render(RenderArgs {
                            // Extrapolate time forward to allow smooth motion.
                            ext_dt: (start_render - self.last_update) as f64 / billion as f64,
                            width: w,
                            height: h,
                        }
                    ));
                }

                self.state = UpdateLoopState;
            },
            SwapBuffersState => {
                self.game_window.swap_buffers();
                self.state = UpdateLoopState;
            },
            UpdateLoopState => {
                let current_time = time::precise_time_ns();
                let next_frame = self.last_frame + self.dt_frame_in_ns;
                let next_update = self.last_update + self.dt_update_in_ns;
                let next_event = cmp::min(next_frame, next_update);
                if next_event > current_time {
                    sleep( Duration::nanoseconds((next_event - current_time) as i32) );
                } else if next_event == next_frame {
                    self.state = RenderState;
                } else {
                    self.state = HandleEventsState;
                }
            },
            HandleEventsState => {
                // Handle all events before updating.
                return match self.game_window.poll_event() {
                    None => {
                        self.state = UpdateState;
                        // Explicitly continue because otherwise the result
                        // of this match is immediately returned.
                        continue;
                    },
                    Some(x) => Some(Input(x)),
                }
            },
            UpdateState => {
                self.state = UpdateLoopState;
                self.last_update += self.dt_update_in_ns;
                return Some(Update(UpdateArgs{
                    dt: self.dt,
                }));
            },
        };
    }
}
```
