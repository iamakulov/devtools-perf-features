# `devtools-perf-features`

Chrome DevTools have a bunch of advanced undocumented flags and features. Some of them are tremendously useful during performance profiling. This repo attempts to document them.

Contents:

- [Timeline: event initiators](#timeline-event-initiators)
- [Timeline: show all events](#timeline-show-all-events)
- [Timeline: invalidation tracking](#timeline-invalidation-tracking)
- [Measuring a part of the recording](#measuring-a-part-of-the-recording)

## Timeline: event initiators

The _Settings → Experiments → Timeline: event initiators_ setting draws little arrows from code that calls `setTimeout()`, `requestAnimationFrame()`, etc. – to code that fires as a result:

<img width="1312" alt="" src="https://user-images.githubusercontent.com/2953267/218883252-0a65290f-7cc0-471c-b6c5-0b28984ea6d0.png">

Works for:

- Timers
- Animation frames
- Style and layout recalculations

**How to enable:** open DevTools settings (press F1) → Experiments → check “Timeline: event initiators”. Then, select any timer callback in the performance trace.

## Timeline: show all events

The _Settings → Experiments → Timeline: show all events_ setting makes DevTools track and show calls to native Chromium functions (rendered in gray):

| Without the setting | With the setting |
| --- | --- |
| <img width="1177" alt="CleanShot 2023-02-15 at 00 45 29@2x" src="https://user-images.githubusercontent.com/2953267/218888346-0df21cd7-7f31-46cb-b414-b95448f99829.png"> | <img width="1177" alt="CleanShot 2023-02-15 at 00 17 13@2x" src="https://user-images.githubusercontent.com/2953267/218886573-98b55344-2a64-4422-b8d7-0eb029d45555.png"> |

(Under the hood, this [enables more tracing categories](https://github.com/ChromeDevTools/devtools-frontend/blob/6948720964e8555a915a5142016fa956943a8ceb/front_end/panels/timeline/TimelineController.ts#L93) and disables filtering [in](https://github.com/ChromeDevTools/devtools-frontend/blob/6948720964e8555a915a5142016fa956943a8ceb/front_end/models/timeline_model/TimelineModel.ts#L728-L730) a [bunch](https://github.com/ChromeDevTools/devtools-frontend/blob/6948720964e8555a915a5142016fa956943a8ceb/front_end/models/timeline_model/TimelineJSProfile.ts#L219-L221) of [places](https://github.com/ChromeDevTools/devtools-frontend/blob/6948720964e8555a915a5142016fa956943a8ceb/front_end/panels/timeline/TimelinePanel.ts#L1110-L1112).)

Use this to:
- see more details about what exactly the browser is doing and when
- quickly jump to relevant places in the Chromium codebase: e.g., to find the where exactly `HTMLDocumentParser::PumpTokenizer` is called, copy that string and paste it [into Chromium Code Search](https://source.chromium.org/search?q=HTMLDocumentParser::PumpTokenizer&sq=&ss=chromium)

**How to enable:** open DevTools settings (press F1) → Experiments → check “Timeline: show all events”.

## Timeline: invalidation tracking

The _Settings → Experiments → Timeline: invalidation tracking_ setting makes DevTools capture what exactly triggered “Recalculate style” and “Layout” operations:

| Without the setting | With the setting |
| --- | --- |
| <img width="1012" alt="CleanShot 2023-02-15 at 00 50 39@2x" src="https://user-images.githubusercontent.com/2953267/218889227-0d076d6f-a32f-4304-9828-e7995fa88e8a.png"> | <img width="1012" alt="CleanShot 2023-02-15 at 00 52 20@2x" src="https://user-images.githubusercontent.com/2953267/218889274-0d4a112c-1ad1-4350-a71e-8401b6c330bd.png"> |
| <img width="1012" alt="CleanShot 2023-02-15 at 00 50 43@2x" src="https://user-images.githubusercontent.com/2953267/218889252-77f446c7-cb8f-4a34-b0cd-06b9c4718cbf.png"> | <img width="1012" alt="CleanShot 2023-02-15 at 00 52 29@2x" src="https://user-images.githubusercontent.com/2953267/218889295-091691a7-536e-435a-971e-2447c3fa2c56.png">

This is useful to debug [layout trashing](https://gist.github.com/paulirish/5d52fb081b3570c81e3a).

Here’s how the setting changes the DevTools’ behavior:

- Without the setting, DevTools only show the first place that invalidated styles or layout. This means that if you have code like this:

  ```js
  document.querySelector('.header').classList.add('header_dark')
  // 500 lines lower
  document.querySelector('.sidebar').classList.add('sidebar_dark')
  ```

  then DevTools will only link to the first line.

- With the setting, DevTools track every change that invalidates cached styles or layout – and links to all of them.

Note that enabling this setting makes all layout operations more expensive.

More watching by Nolan Lawson: https://www.youtube.com/watch?v=nWcexTnvIKI

**How to enable:** open DevTools settings (press F1) → Experiments → check “Timeline: invalidation tracking”.

## Measuring a part of the recording

To quickly measure a part of the recording, hold Shift and select that part of the trace:

https://user-images.githubusercontent.com/2953267/218892333-53a812ff-9eea-4ad2-9116-2e7876051562.mov

<br>

In the bottom part of the selection, you’ll see exactly how long it is:

![](https://user-images.githubusercontent.com/2953267/218892846-01a5654d-7a2c-4d14-b844-21b8cff51598.jpg)

**How to enable:** this is enabled by default but is surprisingly hard to discover.
