# `devtools-perf-tricks`

Chrome DevTools have a bunch of advanced undocumented settings hidden in Settings → Experiments. Some of them are tremendously useful during performance profiling.

### Timeline: event initiators

The _Settings → Experiments → Timeline: Event initiators_ setting enables little arrows between the “Timer Fired”, “Animation Frame Fired”, etc. frames – and the code that called `setTimeout()`, `requestAnimationFrame()`, etc.:

<img width="1312" alt="CleanShot 2023-02-15 at 00 05 52@2x" src="https://user-images.githubusercontent.com/2953267/218883252-0a65290f-7cc0-471c-b6c5-0b28984ea6d0.png">

Works for:

- Timers
- Animation frames
- Style and layout recalculations

**How to enable:** open DevTools settings (press F1) → Experiments → check “Timeline: Event initiators”.
