# CLICK — live metronome (PWA) — v1 Design

**Date:** 2026-06-14
**Status:** Approved, pre-implementation

A setlist metronome for live music. Build a list of songs (name + BPM + time
signature), tap one to load its tempo, hit Start. Switching songs changes the
click on the fly, even mid-play. PWA-first: no build step, no dependencies.

---

## 1. Scope

**In scope (v1):** the full app described in the README — the four core
must-haves PLUS the additive features documented in the architecture notes.

- **Core:** setlist of `{name, bpm, sig}`; tap-to-load tempo (works mid-play);
  drift-free click engine; on-device persistence.
- **Additive:** tap tempo, per-song time signatures, Wake Lock, keyboard
  shortcuts, offline service worker.

**Out of scope (v1):** everything in the README "Roadmap / next tasks" section
— count-in, subdivisions, per-song notes, multiple named setlists, export/import,
drag reorder, SW update prompt, MIDI/footswitch, settings panel.

---

## 2. Files & architecture

Single self-contained app, all relative paths, all files in one folder:

```
click-pwa/
├── index.html            # entire app: markup + <style> + <script>
├── manifest.webmanifest  # installability (name, theme, icons)
├── sw.js                 # service worker — cache-first app shell (offline)
├── README.md
└── docs/superpowers/specs/2026-06-14-click-pwa-design.md   # this file
```

- **No framework, no bundler.** Single self-contained `index.html` is the design.
- **Icons: skipped in v1.** No PNG icon files are produced. The manifest
  references a single inline SVG data-URI icon so it stays valid and installable;
  PNG icon slots (`icon-192`, `icon-512`, `icon-maskable`, `apple-touch-icon`)
  are documented as TODO for when real art exists. The README file tree lists
  those PNGs as the eventual target, not a v1 deliverable.

---

## 3. Click engine (core, must not drift)

Web Audio look-ahead scheduler, decoupling imprecise JS timers from
sample-accurate audio:

- A `setTimeout` loop runs every **25 ms** (`scheduler()`) and schedules every
  click that lands within the next **100 ms** lookahead window against
  `audioContext.currentTime` (`scheduleClick()`).
- Each scheduled beat advances `nextNoteTime` by `60 / bpm` seconds, so tempo
  changes take effect on the next scheduled beat without resetting the clock —
  this is what lets song-switching follow mid-play without drift.
- **Click sound:** triangle oscillator, fast exponential envelope (~35 ms).
  Downbeat (beat 1) = **1760 Hz and louder**; other beats = **1108 Hz**.
- **AudioContext** is created/resumed on the first user gesture (Start / tap) to
  satisfy browser autoplay policy.

### Visual sync

A `requestAnimationFrame` loop reads a queue of scheduled beats (each tagged with
its audio-clock time and beat index) and lights the LED row / pulses the readout
exactly when each click sounds — visuals driven by the audio clock, not by the JS
timer.

---

## 4. Data model & persistence

```js
{
  songs: [
    { id, name, bpm, sig: { beats, unit } }   // e.g. sig { beats: 4, unit: 4 }
  ],
  currentId            // id of the loaded song, or null
}
```

- Persisted under key `clicklist_v1`.
- **Storage abstraction** `store.get/set`: prefers `window.storage` (Anthropic
  artifact sandbox), falls back to `localStorage` (hosted), then in-memory.
  Storage failures degrade silently to in-memory.
- **Starts empty.** First run shows an empty-state prompt to add the first song.

---

## 5. Time signatures

- Per-song beats-per-bar, stored as `sig: { beats, unit }` (e.g. 4/4, 3/4, 6/8).
- The **LED row length follows `sig.beats`** for the current song.
- **Beat 1 is the accented red downbeat** (1760 Hz, louder, red LED); all other
  beats use the normal click (1108 Hz).
- `beats` and `unit` are editable in the add/edit song form.

---

## 6. UI & interaction (stage-first, dark)

Single screen, two zones.

**Metronome face (top)**
- Large amber JetBrains-Mono **BPM readout**.
- **LED beat row**, length = current song's `sig.beats`; beat 1 red.
- Big **Start / Stop** control.
- **± tempo nudge** (±1 BPM).
- Current **song name** (Oswald display font).
- **Wake-lock toggle** top-right.

**Setlist (below)**
- Tappable **rows** (name · BPM · sig). Tapping a row loads its tempo instantly,
  even while playing.
- Per-row actions: **edit**, **▲ move-up**, **delete**.
- **"+ Add song"** form: name, BPM, beats, unit.
- **Empty-state prompt** when there are no songs.

**Visual language:** dark background, amber LED-style readout, **red downbeat**
(matches real hardware metronomes). Fonts: **Oswald** (display) + **JetBrains
Mono** (numbers/data).

---

## 7. Additive features

- **Tap tempo:** `T` key or a Tap button; averages the last few inter-tap
  intervals into a BPM (resets if taps stall).
- **Wake Lock:** `navigator.wakeLock` acquired while playing, re-acquired on
  `visibilitychange` refocus; wrapped in try/catch so unsupported browsers
  degrade silently. Toggle in the top-right.
- **Keyboard shortcuts:** `space` start/stop · `←/→` change song · `↑/↓` tempo
  ±1 · `T` tap tempo. Ignored while typing in an input.
- **Service worker (`sw.js`):** cache-first app shell listing all files; a
  `CACHE` version constant is bumped whenever a cached file changes. Registered
  only over https / localhost (no-op from `file://`).

---

## 8. Error handling & edge cases

- **Autoplay:** AudioContext created/resumed on first user gesture.
- **BPM clamping:** constrained to a sane range (20–300) on entry and nudge.
- **Wake Lock:** unsupported / rejected → silent no-op, toggle reflects real state.
- **Storage:** any failure falls through to in-memory; app stays usable.
- **Delete current song:** picks a sensible next `currentId` (or null if list
  becomes empty) and stops or keeps playing coherently.
- **Service worker:** registration guarded so `file://` and unsupported browsers
  don't throw.

---

## 9. Testing

No build step and no framework, so v1 ships with a **manual smoke-test
checklist** rather than an automated suite (no test framework added unless we
later decide to):

1. Add a song; it appears in the setlist and persists across reload.
2. Tap a song while playing — tempo follows on the next beat, no audible glitch.
3. Drift check: run a few minutes against an external reference; stays locked.
4. Time signature: changing `beats` changes LED row length and downbeat position.
5. Tap tempo produces a believable BPM.
6. Keyboard shortcuts all work; ignored while typing in the add form.
7. Wake Lock keeps screen on while playing (on a supporting device).
8. Offline: load once online, go offline, reload — app shell still loads.
9. Install to home screen (Android/Chrome, iOS/Safari).

---

## 10. Conventions (keep)

- No framework, no bundler — single self-contained file is the design.
- Stage-first visual language; Oswald + JetBrains Mono.
- Bump `CACHE` in `sw.js` whenever a cached file changes.

---

## 11. Deployment

Static host — target is **Cloudflare Pages** (the original request). Any static
host works (Netlify drop, GitHub Pages, Vercel). All paths relative; every file
stays in the same folder.
