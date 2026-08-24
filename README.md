# CLICK — live metronome (PWA)

A setlist metronome for live music. Build a list of songs (name + BPM), tap one to load its tempo, hit **Start**. Switch songs and the click follows — even mid-play.

---

## Features

- **Setlist** — each song has a name, BPM, time signature and free-text notes (key, cues, lyric snippet); tap one to load its tempo instantly, even mid-play. Drag the ⠿ grip to reorder. Persists on the device.
- **Drift-free click** — Web Audio look-ahead scheduler, so a long set stays in time.
- **Free mode** — start the metronome with no song selected.
- **Beat control** — per-beat accent/mute, subdivisions ×1–×4, one-bar count-in, and an optional spoken beat count.
- **Tap tempo** and keyboard shortcuts: `space` start/stop · `←/→` song · `↑/↓` tempo · `T` tap.
- **Sound settings** — three click voices (beep / wood / low) and master volume.
- **Stage-ready** — dark LED-style readout, wake lock to keep the screen on, installable and fully offline as a PWA.

---

## Install to home screen

- **Android/Chrome:** menu → "Install app" / "Add to Home screen".
- **iPhone/Safari:** Share → "Add to Home Screen", then launch from that icon so it runs full-screen and the screen-lock toggle works.

---

## Roadmap / next tasks

- [ ] Real `apple-touch-icon.png` (iOS ignores the inline SVG icon)
- [ ] Multiple named setlists; export/import as JSON
- [ ] PWA update prompt when a new service worker is waiting
- [ ] Optional MIDI / footswitch to advance songs hands-free

## Live-use gotchas

- Wired in-ears beat Bluetooth — BT adds ~100–200 ms of latency nothing in code can fix.
- Launch the installed (home-screen) version for reliable full-screen + wake lock.

## Contributing

Issues and PRs welcome. Read [ARCHITECTURE.md](ARCHITECTURE.md) first — it covers how the click engine works and the house rules (no framework, no bundler, everything stays in `index.html`). Test on a phone, not just desktop; audio timing and wake lock behave differently there.

---

MIT © Jonathan Rehuel Lewerissa
