# CLICK — live metronome (PWA)

A setlist metronome for live music. Build a list of songs (name + BPM), tap one to
load its tempo, hit **Start**. Switch songs and the click follows — even mid-play.

> **Status:** working prototype, PWA-first. No build step, no dependencies — plain
> HTML/CSS/vanilla JS in a single `index.html`. Open it, host it, install it.

---

## Core spec (the must-haves)

1. A setlist where each entry has a **name** and a **BPM** (and an optional time signature).
2. Tapping a song **loads its tempo** into the metronome instantly.
3. A click engine that plays at the current tempo without drifting over a long set.
4. The setlist **persists** on the device.

Everything else (tap tempo, time signatures, wake lock, keyboard shortcuts) is
additive and can be removed without touching the core.

---

## Run it locally

A service worker needs HTTPS or `localhost`, so **don't** just double-click
`index.html` (install + offline won't work from `file://`). Serve the folder:

```bash
# from inside click-pwa/
python3 -m http.server 8000
# or:  npx serve .
```

Then open http://localhost:8000

## Deploy (real PWA)

- **Fastest:** drag the whole `click-pwa` folder onto https://app.netlify.com/drop
- **Also fine:** GitHub Pages, Vercel, Cloudflare Pages — any static host.

Keep every file in the **same folder**; all paths are relative.

## Install to home screen

- **Android/Chrome:** menu → "Install app" / "Add to Home screen".
- **iPhone/Safari:** Share → "Add to Home Screen", then launch from that icon so it
  runs full-screen and the screen-lock toggle works.

---

## Project structure

```
click-pwa/
├── index.html            # the entire app: markup, styles, logic
├── manifest.webmanifest  # installability (name, icons, theme)
├── sw.js                 # service worker — cache-first app shell (offline)
├── icon-192.png          # home-screen icon
├── icon-512.png          # home-screen icon
├── icon-maskable.png     # Android maskable icon
├── apple-touch-icon.png  # iOS icon
└── README.md
```

## Architecture notes

- **Timing — look-ahead scheduler.** `setTimeout` runs every 25 ms and schedules any
  clicks landing within the next 100 ms using the Web Audio clock
  (`audioContext.currentTime`). This decouples imprecise JS timers from sample-accurate
  audio, so the click never drifts. See `scheduler()` / `scheduleClick()`.
- **Click sound.** Triangle oscillator with a fast exponential envelope (~35 ms).
  Downbeat = 1760 Hz and louder; other beats = 1108 Hz. Tweak in `scheduleClick()`.
- **Visual sync.** A `requestAnimationFrame` loop reads a queue of scheduled beats and
  lights the LED row / pulses the readout exactly when each click sounds.
- **Storage abstraction.** `store.get/set` prefers `window.storage` (Anthropic artifact
  sandbox), falls back to `localStorage` (hosted), then in-memory. Persisted under key
  `clicklist_v1` as `{ songs: [{id,name,bpm,sig}], currentId }`.
- **Wake Lock.** `navigator.wakeLock` keeps the screen on while playing; re-acquired on
  tab refocus. Toggle in the top-right.
- **Keyboard.** `space` start/stop · `←/→` change song · `↑/↓` tempo · `T` tap tempo.

## Conventions (please keep)

- **No framework, no bundler.** Single self-contained file is the design. Don't add
  React/Vite unless we deliberately decide to.
- **Stage-first visual language.** Dark background, amber LED-style readout, **red
  downbeat** (matches real hardware metronomes). Fonts: Oswald (display) + JetBrains
  Mono (numbers/data). Bump `CACHE` in `sw.js` whenever a cached file changes.

---

## Roadmap / next tasks

- [ ] Count-in (1 bar of clicks before the "song" starts)
- [ ] Sub-divisions (8th / 16th note clicks) and custom accent patterns
- [ ] Per-song notes field (key, cues, lyrics snippet)
- [ ] Multiple named setlists; export/import as JSON
- [ ] Reorder by drag (currently ▲ move-up only)
- [ ] PWA update prompt when a new service worker is waiting
- [ ] Optional MIDI / footswitch to advance songs hands-free
- [ ] Settings: click sound choice, accent on/off, default volume

## Live-use gotchas

- Wired in-ears beat Bluetooth — BT adds ~100–200 ms of latency nothing in code can fix.
- Launch the installed (home-screen) version for reliable full-screen + wake lock.
