# Authoring FAQ (agent quick reference)

Short, citable answers for common user questions. Link the user to the deeper page when relevant. Mark **(provisional)** items as such — the source of truth is the referenced `openspec/changes/` entry.

---

**Q: Do I need to export pitch, gate, and velocity as three separate wires/tracks in the DAW?**
No. A MIDI note already contains pitch (note number), gate (its duration), and velocity. You author **one monophonic track per voice**. Maybelle splits that into separate pitch-CV / gate / velocity outputs on the ES-9 — that split happens in Maybelle, not the DAW, and is set in Maybelle's configuration. See [midi-to-cv-model.md](midi-to-cv-model.md).

**Q: So how does one note become three outputs?**
Maybelle is a MIDI→CV converter (like a Doepfer A-190-3 or Expert Sleepers FH-2). For each voice it derives: pitch CV (1V/oct), a gate, and optionally a velocity CV, and sends each to an assigned ES-9 output. The ES-9 is only the output stage (a DC-coupled DAC).

**Q: Which ES-9 outputs do my voices come out of?**
The ES-9's **8 front-panel 3.5mm jacks** are the DC-coupled CV/gate outputs (DAW channels 9–16 → outs 1–8), ~±10V. **Never use the two 1/4" Main outs — they're AC-coupled** and won't pass CV. The exact voice→output assignment is set in Maybelle (Backstage), **(provisional)**. Reference: `openspec/changes/investigate-es9-config-profiles`.

**Q: Can a track play chords?**
Not into a single pitch CV — one pitch output is monophonic. Use **one track per chord note** (one voice each). See [midi-to-cv-model.md](midi-to-cv-model.md).

**Q: My LFO / filter sweep / automation didn't come through. Why?**
Bitwig's MIDI export is **notes + velocity only** — automation/CC/note-expression are dropped. (Ardour can export more but it's not reliably portable.) Author modulation as **notes** (stepped CV on a dedicated track) or bake the LFO shape into a sampler and fire it with a **trigger note**. See [midi-to-cv-model.md](midi-to-cv-model.md#modulation) and `openspec/changes/research-synced-lfo-sampler-authoring`.

**Q: Does velocity have to be used?**
No. If you don't route a velocity CV, the note still plays pitch + gate. Velocity carries through the export regardless (it's part of the note).

**Q: Gate vs trigger — which do I get?**
A **gate** stays high for the note's length (melodies, envelopes). A **trigger** is a short pulse at note-on (drums). Which one an output emits is Maybelle config; for percussion, author short notes. **(provisional map.)**

**Q: What tempo should I author at? Will my tempo automation work?**
Author at any sensible tempo/grid — but Maybelle **follows the rack's master clock** (Pamela's Pro Workout) and re-clocks playback, so the file's tempo is only a positional grid. Don't rely on exported tempo-map automation.

**Q: Do I set the track-to-output routing in Bitwig/Ardour?**
No. Routing (which voice → which ES-9 output) is Maybelle configuration, done in **Backstage** mode on a laptop, not in the DAW. **(provisional — reference `openspec/changes/research-performance-backstage-modes` and `research-daw-interchange-options`.)**

**Q: Which DAW should I use?**
Both Bitwig and Ardour are supported. Bitwig is the current test bench. Whichever you use, **author to the notes+velocity common denominator** so the piece is portable between them.

**Q: Do I need to run the DAW on the Raspberry Pi?**
No. Author on your computer, export a Standard MIDI File, and load it onto the Pi.

---

### How agents should use this
- Anchor answers to the **notes+velocity** contract and the **"split happens in Maybelle, not the DAW"** principle.
- Distinguish **decided** facts (model, export limits, ES-9 output facts) from **(provisional)** conventions (exact mapping/manifest), and cite the OpenSpec change for provisional items.
- When unsure about a provisional detail, say it's not finalized and point to `openspec/changes/`, rather than inventing a convention.
