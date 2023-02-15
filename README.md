# `devtools-perf-tricks`

Chrome DevTools have a bunch of advanced undocumented settings hidden in Settings → Experiments. Some of them are tremendously useful during performance profiling.

Contents:

- [Timeline: event initiators](#timeline-event-initiators)
- [Timeline: show all events](#timeline-show-all-events)
- [Timeline: invalidation tracking](#timeline-invalidation-tracking)

## Timeline: event initiators

The _Settings → Experiments → Timeline: event initiators_ setting enables little arrows between the “Timer Fired”, “Animation Frame Fired”, etc. frames – and the code that called `setTimeout()`, `requestAnimationFrame()`, etc.:

<img width="1312" alt="CleanShot 2023-02-15 at 00 05 52@2x" src="https://user-images.githubusercontent.com/2953267/218883252-0a65290f-7cc0-471c-b6c5-0b28984ea6d0.png">

Works for:

- Timers
- Animation frames
- Style and layout recalculations

**How to enable:** open DevTools settings (press F1) → Experiments → check “Timeline: event initiators”.

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

This is useful to debug layout thrashing in more details.

Here’s how the behavior differs:

- Without the setting, DevTools only show the first place that invalidated styles or layout. This means that if you have code like this:

  ```js
  document.querySelector('.header').classList.add('header_dark')
  // 500 lines lower
  document.querySelector('.sidebar').classList.add('sidebar_dark')
  ```

  then DevTools will show “First Invalidated: <link to file and line>” – but only link to the first line.

- With the setting, DevTools track every change that invalidates cached styles or layout – and prints links to all of them.

Note that enabling this setting makes all layout operations more expensive.

More watching by Nolan Lawson: https://www.youtube.com/watch?v=nWcexTnvIKI

**How to enable:** open DevTools settings (press F1) → Experiments → check “Timeline: invalidation tracking”.
