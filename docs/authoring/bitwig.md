# Authoring in Bitwig Studio for Maybelle 314

Read [midi-to-cv-model.md](midi-to-cv-model.md) first. Bitwig is the current test-bench DAW; Ardour is also supported.

## Track setup

1. **One monophonic instrument track per voice.** Each track that drives a pitch CV should play one note at a time. For a chord or a polyphonic part, use one track per voice.
2. **Author on the Arranger timeline**, not (only) the Clip Launcher — MIDI export comes from the Arrangement (see Export below).
3. **Name tracks clearly and consistently.** Track identity is how Maybelle's configuration will bind a voice to ES-9 outputs. **(provisional — mapping is set in Maybelle, not Bitwig.)**
4. **Percussion:** a track of short notes → triggers; note number can select the drum/trigger output. **(provisional map.)**
5. You can use any instrument to *hear* your part while authoring — the sound is only a monitor; Maybelle uses the notes, not Bitwig's audio.

## What Bitwig exports — important

`File → Export MIDI…` produces a **Type-1 Standard MIDI File** containing **notes + velocity only**, for the whole **Arrangement**. It **does not** export:

- automation or MIDI CC,
- note expressions / MPE / micro-pitch,
- sustain (CC64), tempo, or Clip Launcher content.

Consequences for Maybelle authoring:

- **Pitch, gate, and velocity all come through** (they are part of each note) — this is everything a basic voice needs.
- **Do not rely on Bitwig automation for modulation** — it is discarded. Author modulation as **notes on a dedicated track**, or bake LFO shapes into a sampler and fire them with **trigger notes** (see [midi-to-cv-model.md](midi-to-cv-model.md#modulation)).
- There is **no native per-clip MIDI export**; export the arrangement (or use the new-tab / mute-others workarounds if you need a single clip).

> Note: whether a newer Bitwig version restores MIDI CC export is unconfirmed — verify against the installed version before relying on CC. Reference: `openspec/changes/research-daw-interchange-options`.

## Export checklist

- [ ] Each voice is a **monophonic** Arranger track (chords split into multiple tracks).
- [ ] Modulation you need is authored as **notes** (not automation).
- [ ] Velocities set as intended (they carry through).
- [ ] `File → Export MIDI…` → save the `.mid`.
- [ ] Hand the `.mid` to the Maybelle bundle; set track→ES-9-output mapping in Maybelle (Backstage). **(provisional workflow.)**

## Common pitfalls

- **"My filter sweep didn't play."** Bitwig didn't export it — automation is dropped. Re-author it as notes/steps or a sampler-baked LFO.
- **"My chords are glitching on one output."** A pitch CV is monophonic; give each chord note its own track/voice.
- **"I drew note expression / per-note pitch."** Gorgeous in Bitwig, but discarded on export. Keep pitch in the notes themselves.
