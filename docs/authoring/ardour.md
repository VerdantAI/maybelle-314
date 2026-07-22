# Authoring in Ardour for Maybelle 314

Read [midi-to-cv-model.md](midi-to-cv-model.md) first. Ardour is a supported authoring DAW alongside Bitwig.

## Track setup

1. **One monophonic MIDI track per voice.** As with any pitch-CV target, one note at a time per voice; split chords/polyphony into multiple tracks.
2. **Name tracks clearly and consistently** — track identity is how Maybelle configuration binds a voice to ES-9 outputs. **(provisional — mapping is set in Maybelle, not Ardour.)**
3. **Velocity** is authored per note (Ardour shows it as a "lollipop" velocity lane: right-click the track header → *Automation → Velocity*). Velocity carries through in the MIDI export as part of each note.
4. **Percussion:** short notes → triggers; note number can select the output. **(provisional map.)**

## What Ardour exports

Ardour supports richer MIDI editing than Bitwig — full CC automation lanes, velocity, etc. — but for Maybelle you should **rely on the portable common denominator: notes + velocity**. Reasons:

- Complex automation (e.g. bender/CC lanes) is **not guaranteed to survive** SMF export/round-trips.
- Keeping to notes+velocity means the **same authoring conventions work in both Ardour and Bitwig**, so a piece is portable between them.

So even though Ardour *can* carry more, author as if only **notes + velocity** will arrive — and put modulation into **notes** (or sampler-baked LFOs fired by trigger notes), per [midi-to-cv-model.md](midi-to-cv-model.md#modulation).

> The exact set of events Ardour writes to an SMF (CC, tempo map, markers) still needs a bench pass with a real export; until then, treat notes+velocity as the contract. Reference: `openspec/changes/research-daw-interchange-options`.

## Export checklist

- [ ] Each voice is a **monophonic** MIDI track (chords split across tracks).
- [ ] Modulation you need is authored as **notes** (portable), not only as CC/automation.
- [ ] Velocities set as intended.
- [ ] Export the MIDI (Ardour's MIDI export / region export to `.mid`).
- [ ] Hand the `.mid` to the Maybelle bundle; set track→ES-9-output mapping in Maybelle (Backstage). **(provisional workflow.)**

## Ardour-specific notes

- **Running Ardour on the Pi is not required** — author on your laptop/desktop and load the export onto the Pi. (Ardour on a Pi is feasible but out of scope for the runtime; reference `openspec/changes/research-daw-interchange-options`.)
- If you use CC automation for a controllable parameter, confirm it actually appears in the exported `.mid` before depending on it; otherwise convert it to notes.
