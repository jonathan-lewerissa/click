# CLICK — architecture

How `index.html` is put together, and the rules for changing it.

## How it works

- **Timing — look-ahead scheduler.** `setTimeout` runs every 25 ms and schedules any clicks landing within the next 100 ms using the Web Audio clock (`audioContext.currentTime`). This decouples imprecise JS timers from sample-accurate audio, so the click never drifts. See `scheduler()` / `scheduleClick()`.
- **Click sound.** Three presets (`beep` / `wood` / `low`) in the `SOUNDS` table — waveform, down/beat/sub frequencies, decay, loudness trim. Master volume rides an `audio.master` gain node. Tweak in `SOUNDS` / `scheduleClick()`.
- **Beats.** Per-beat accent levels (tap an LED: normal → accent → mute), subdivisions ×1–×4, optional spoken beat count (`speechSynthesis`, fired `VOICE_LEAD` early), and a one-bar count-in. Subdivision/voice are per-song when a song is loaded, otherwise global.
- **Free mode.** With no song selected the metronome runs on `state.bpm` at 4/4 — start it without a setlist.
- **Visual sync.** A `requestAnimationFrame` loop reads a queue of scheduled beats and lights the LED row / pulses the readout exactly when each click sounds.
- **Storage abstraction.** `store.get/set` prefers `window.storage` (Anthropic artifact sandbox), falls back to `localStorage` (hosted), then in-memory. Persisted under key `clicklist_v1` as `{ songs: [{id,name,bpm,sig,accents,sub,voice}], currentId, bpm, sub, voice, sound, vol, count }`.
- **Wake Lock.** `navigator.wakeLock` keeps the screen on while playing; re-acquired on tab refocus. Toggle in the top-right.
- **Keyboard.** `space` start/stop · `←/→` change song · `↑/↓` tempo · `T` tap tempo.

## Conventions (please keep)

- **No framework, no bundler.** Single self-contained file is the design. Don't add React/Vite unless we deliberately decide to.
- **Stage-first visual language.** Dark background, amber LED-style readout, **red downbeat** (matches real hardware metronomes). Fonts: Oswald (display) + JetBrains Mono (numbers/data). Bump `CACHE` in `sw.js` whenever a cached file changes.
