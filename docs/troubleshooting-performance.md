# Troubleshooting FeedBack performance on older devices

Use this guide to distinguish stuttering graphics, delayed instrument sound, audio crackling, slow menus, and late note judgments. Start with the short comparison below. You do not need to compile anything or install our fork: these checks are for the upstream FeedBack app you already use.

This is a community guide maintained in the stdio42 fork, checked against upstream source on 26 October 2026. Controls and diagnostic tools can differ between builds. If an option is missing, record your version and skip that step.

## Start with a repeatable comparison

Allow any first-run downloads or library processing to finish. Keep the app in the foreground. Choose a song and arrangement that reliably show the problem, then use the same 60-second section at 1.0× speed for each test.

1. **Record your setup.** Note the FeedBack build, desktop or browser mode, OS, computer model, RAM, GPU if known, audio interface, headphones or speakers, and enabled plugins. For a browser connected to another machine, record both machines.
2. **Enable the FPS counter for the guitar 3D highway.** Look under Settings → 3D Highway → Camera → **Show FPS counter**. It appears at the top right of the highway. Other renderers may not have this control. [Upstream control](https://github.com/got-feedBack/feedBack/blob/main/plugins/highway_3d/settings.html)
3. **Run your normal setup twice.** Write down the approximate FPS range, the song time where a hitch occurs, whether sound also breaks up, and whether the second run is better. Avoid screen recording during the initial test because it adds another workload.
4. **Change one thing and repeat twice.** Pick the relevant comparison in the table below. Restore the original setting before testing a different change. If an improvement repeats, turn the original setting back on once to see whether the problem returns.
5. **Export diagnostics after reproducing the problem.** In that same app session, open Settings → **Export Diagnostics**. Save the bundle with your notes. [Official export instructions](https://got-feedback.github.io/feedBack-user-guide/first-run/export-diagnostics/)

Keep the window size, display, power mode, song section, and audio devices consistent unless one of those is the variable you are testing. A laptop power comparison should be its own test. Do not update the app midway through a comparison; record a new baseline after an update.

## Match the symptom to a test

These results suggest where to investigate; they do not prove a single cause.

| What you notice | Change only this | What an improvement suggests |
| --- | --- | --- |
| Notes jump or the highway stutters while audio stays smooth | Switch from 3D to the basic 2D highway, if available | The 3D rendering path or its associated UI work deserves attention. This alone cannot distinguish CPU from GPU load. |
| Only 3D struggles | Use a smaller app window; in a separate test, disable an optional visual effect or animated background | Sensitivity to window size suggests rendering or compositing cost. Sensitivity to an effect narrows the expensive feature. |
| Stutter starts when detection is enabled | Temporarily turn off note detection, retaining playback and the same view | Detection or its integration with the UI contributes to the load. Scores are not comparable with detection off. |
| The whole app becomes sluggish with extra features | Disable one nonessential plugin and restart if needed; repeat for others | A reproducible improvement points toward that plugin or a plugin interaction. Keep required input and playback components enabled. |
| Crackles or gaps appear when amp simulation or effects are active | Bypass one audio effect at a time | The effect chain or its routing may be exceeding the available processing time. |
| You hear your instrument noticeably after playing, but graphics are smooth | Follow the audio checks below | Monitoring, output-device, or driver latency may be involved. |
| Sound and graphics seem smooth, but judgments arrive late or feel wrong | Follow the detection checks below | Input selection, tuning, chart alignment, or detection settings may be involved. |
| One song or arrangement behaves badly | Try a second song, then another arrangement where available | The original chart, media, or its density may be relevant. Record the exact arrangement and timestamp. |
| Menus or library loading are slow even before playback | Compare after library processing has finished, then with nonessential plugins disabled | Investigate library work, plugins, storage, or the backend before focusing on rendering. |

Trying another song and disabling nonessential plugins are also upstream's recommended starting checks. [Performance troubleshooting](https://got-feedback.github.io/feedBack-user-guide/troubleshooting/performance-crashes/) · [Playback troubleshooting](https://got-feedback.github.io/feedBack-user-guide/troubleshooting/playback/)

## Separate audio delay from audio dropouts

**Delay** means sound arrives late but remains continuous. **Dropouts** mean clicks, crackles, or missing sound. Write down whether the problem affects the backing track, your monitored instrument, or both.

- Confirm the selected input, output, and input channel. On Windows, check the driver label as well as the device name: the same interface can appear through several audio paths. Test the interface's low-latency driver if it is available. Record the old and new driver names. [Driver guide](https://got-feedback.github.io/feedBack-user-guide/desktop-audio/driver-types/)
- If using Bluetooth output, compare with wired headphones as a separate test. Keep the other settings unchanged and note that the output device changed.
- If the app or driver exposes buffer size, record it and the sample rate. For crackling, try one larger supported buffer. For continuous but delayed sound, try one smaller supported buffer and revert if crackling starts. This tests the latency-versus-stability tradeoff; it is not a universal fix.
- If your interface offers direct hardware monitoring, compare it with the app's processed monitoring, one at a time. Direct monitoring bypasses the software path and usually its effects. A responsive direct signal with delayed app monitoring narrows the issue to the software/output path; it does not identify the exact component.

Do not treat an FPS improvement as evidence that audio latency improved. Likewise, adjusting a scoring offset will not remove audible monitoring delay.

## Check note detection separately

First confirm the tuner responds to the intended instrument. Verify the input channel, tuning, instrument preset, and string count before changing calibration. [Upstream calibration checks](https://got-feedback.github.io/feedBack-user-guide/troubleshooting/calibration/)

If your Note Detection plugin provides the advanced tools:

1. In Settings → Note Detection, enable **Detection tuning (advanced)**.
2. Use **Download Diagnostic JSON** after a representative take.
3. For a replayable capture, arm **Reference Recording** from the player's Detect gear popover before starting, and play at **1.0× speed**. The documented recording auto-save occurs at song end.
4. Include the diagnostic JSON and whether judgments are mainly early, late, wrong-pitch, or missed entirely. Share a recording privately if it is needed for replay.

The developer replay harness can compare detector settings against the same recording. For calibration analysis, upstream identifies `timing_error_ms_hits` as the useful timing distribution; the all-matched `timing_error_ms` distribution can be misleading. These diagnostics measure detection and scoring behavior, not physical input-to-output latency. [Detection tools and instructions](https://github.com/got-feedBack/feedBack/blob/main/docs/note-detect-tuning.md)

## Optional detailed graphics capture

Use this section when the simple comparisons point toward visual stutter. It is easiest in the browser version. A browser reproduction is useful, but it does not reproduce the desktop app's native audio path.

### Built-in 3D profiler

Open your existing app URL with `?h3dbench=1` appended. If it already has query parameters, append `&h3dbench=1` instead. Put the parameter before any `#` fragment. This is the URL of your running app, not the got-feedback.org marketing site.

For example, if you already use `http://localhost:8000/`, open `http://localhost:8000/?h3dbench=1`. Use your actual address and port.

Open the browser's developer tools through its menu, select **Console**, and filter for `h3dbench`. Start the guitar 3D highway and play the problem section. Copy several consecutive reports, including the hitch and a smoother period.

The profiler reports approximately every five seconds:

- FPS and frame count.
- `frame`, `state`, `next`, `mat`, `noteDraw`, `chordDraw`, and `render` timings.
- `p50` (median), `p95` (95th percentile), and `max`, in milliseconds, for each segment.

Higher p95 or maximum values reveal spikes that a median can hide. These are JavaScript-side elapsed timings, not direct GPU execution measurements, and the total frame measurement overlaps its component segments. Do not add all the columns together. A smooth-looking FPS average also cannot rule out occasional hitches.

If the installed desktop app has no accessible URL or developer-tools control, skip this step and send the FPS observations and diagnostics bundle. Do not switch to a browser and label its results as a desktop capture. [Profiler implementation](https://github.com/got-feedBack/feedBack/blob/main/plugins/highway_3d/screen.js)

### Browser performance trace

If you can reproduce the issue in a Chromium-based browser, use Developer Tools → **Performance** to record a short trace covering the hitch, then stop and save it. A developer can inspect script execution, layout, painting, and garbage collection. Record this separately from the baseline: profiling itself adds overhead. [Chrome recording instructions](https://developer.chrome.com/docs/devtools/performance/overview)

## Optional automated benchmark for a developer

The upstream repository includes `scripts/perf-baseline.mjs`. With its Node dependencies and Playwright Chromium installed, and a running app containing the named song, run this from the repository root:

```bash
node scripts/perf-baseline.mjs \
  --base http://127.0.0.1:8000 \
  --n 60 --soak 30 \
  --song "Your exact song filename.feedpak" \
  --frames 30 --runs 3
```

Replace the URL and filename. The report includes API latency, page load to network idle, JavaScript heap usage, plugin-script count, and a song-mode measurement intended for the 2D highway's drawing cost. Use a basic 2D configuration for that comparison. Without `--song`, it does not measure playback drawing cost.

Run comparisons on the affected device with the same configuration. JavaScript heap is not total application memory; draw-callback duration is not a complete displayed-frame measurement. This browser harness does not measure native desktop audio latency or establish that a Rust rewrite would be faster. [Harness](https://github.com/got-feedBack/feedBack/blob/main/scripts/perf-baseline.mjs) · [Upstream baseline notes](https://github.com/got-feedBack/feedBack/blob/main/docs/perf-baseline.md)

## Send back a useful report

The most useful package is this completed template plus the diagnostics bundle from the failing session. Include profiler output or a trace only if you captured one. Before sharing logs or recordings publicly, check them for personal paths, device names, or audio you did not intend to publish.

```text
FeedBack version/build:
Desktop app or browser (browser name/version if applicable):
OS and computer model:
CPU / RAM / GPU, if known:
Display resolution / refresh rate; app window size:
Laptop plugged in or on battery; power mode:
Audio interface / input channel / output device:
Audio driver / sample rate / buffer size, if visible:
Song / arrangement / problem timestamp:
Playback speed:
Enabled plugins and audio effects:

What happens: visual stutter / delayed sound / crackles / bad judgments / slow UI
Backing audio affected, instrument monitoring affected, or both:
Happens every time, only first play, or after several minutes:

Baseline, two runs: FPS range and symptoms
Change tested, one variable only:
Changed setup, two runs: FPS range and symptoms
Original setting restored: did the problem return?
Second song comparison:

Diagnostics bundle filename:
Optional h3dbench output / performance trace / detection JSON:
```

Start by sending the baseline and the single comparison that made the biggest difference. That gives the next investigation a specific target instead of just “the app lags.”
