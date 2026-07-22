# The MIDI → CV Model

This is the DAW-agnostic mental model. Both the Bitwig and Ardour guides build on it.

## Layers

```
   DAW (Bitwig / Ardour)          Maybelle 314 (Raspberry Pi)         ES-9            Rack
   ─────────────────────          ──────────────────────────         ────            ────
   author MIDI notes    ──SMF──▶  MIDI→CV engine (per manifest) ──▶  DC-coupled  ──▶ VCO / VCA /
   (one track per voice)          fans each voice out to             outputs         envelopes / etc.
                                   pitch CV + gate + velocity
```

- **The DAW** produces notes. Nothing more is required for a basic voice.
- **Maybelle** reads the Standard MIDI File plus its configuration ("manifest") and converts each note into control voltages/gates, re-clocked to the rack.
- **The ES-9** is the physical output — a DC-coupled USB audio interface whose jacks carry the voltages Maybelle produces. It does no MIDI conversion itself.

## A note already contains pitch, gate, and velocity

You never split these apart in the DAW. One MIDI note = pitch (note number) + gate (its on/off duration) + velocity. Maybelle decides which physical ES-9 output carries each of those:

- **Pitch CV** — 1V/oct. Maybelle owns the tuning/calibration in software, so you just author the correct notes.
- **Gate** — high while the note is held. Used to open envelopes/VCAs.
- **Velocity CV** — optional; a voltage proportional to note velocity, for accents/dynamics.

So one authored voice typically uses **up to three ES-9 outputs**, but that is a Maybelle output-mapping choice, not DAW work.

## One monophonic voice per track

A single pitch-CV output is **monophonic** — it can only be at one pitch at a time. Therefore:

- Make each melodic/bass track **monophonic** (no overlapping notes/chords) if it drives one pitch CV.
- A **chord** needs multiple voices → multiple tracks → multiple pitch/gate output pairs (one per note of the chord).
- Keep one musical voice per track and let the mapping assign it to outputs.

## Gate vs. trigger

- **Gate** = a level that stays high for the note's duration (melodic voices, sustained envelopes).
- **Trigger** = a short pulse at note-*on* only (drums/percussion, clock-like hits).

Which one an output emits is Maybelle configuration. For **percussion**, author short notes on a track; each note becomes a trigger on its assigned output. Note *number* can select which drum/trigger output (a percussion map). **(provisional mapping detail.)**

## Velocity

Velocity is authored per note (in both DAWs). Whether it is emitted as a velocity CV — and on which output — is Maybelle configuration. If you don't need velocity CV, you can ignore it; the note still plays pitch+gate.

## <a name="modulation"></a>Modulation (LFOs, sweeps, stepped CV)

Continuous modulation (filter sweeps, LFOs) is the one area with a real constraint:

- **Bitwig exports notes + velocity only** — automation, MIDI CC, and note expressions are **dropped** on MIDI export.
- **Ardour** can export more, but automation is not guaranteed to survive round-trips, so don't rely on it for portability.

**Therefore author modulation so it survives as notes**, not as DAW automation lanes:

- **Stepped modulation** — put notes on a dedicated track; each note's pitch/velocity encodes a CV value, each note is a step. Maybelle emits these as a stepped CV. **(provisional encoding.)**
- **Synced LFOs / evolving shapes** — bake the shape into a waveform in a sampler (e.g. Assimil8or) and use a **trigger note** in the MIDI to (re)start it; the trigger survives export while the shape lives in the sampler. See `openspec/changes/research-synced-lfo-sampler-authoring`.

## Clock and tempo

Maybelle **follows the rack's master clock** (Pamela's Pro Workout); it does not use your DAW file's tempo to set playback speed. The file's tempo/PPQ grid only defines *where* notes sit musically; Maybelle re-clocks them to the incoming clock. Author at a sensible tempo/grid; don't depend on exported tempo-map automation.

## ES-9 output facts (for accurate answers)

- **8× 3.5mm DC-coupled outputs**, ~±10V — these are Maybelle's CV/gate/velocity/mod outputs. They correspond to DAW audio channels **9–16 → physical outs 1–8** in the ES-9's hosted configuration.
- **14× 3.5mm DC-coupled inputs**, ~±10V — rack CV/clock *into* Maybelle (song/channel selection, transport, clock).
- **2× 1/4" Main outputs are AC-coupled — never use them for CV.** The headphone out is also not for CV.
- Maybelle handles **1V/oct scaling and calibration in software**; the ES-9 has no per-output pitch calibration of its own.

Source of truth: `openspec/changes/investigate-es9-config-profiles`.

## What is decided vs. provisional

- **Decided/stable:** the layered model, notes carry pitch/gate/velocity, mono-voice-per-track, gate vs trigger, the export constraints, and the ES-9 output facts.
- **Provisional:** the exact bundle/manifest format and how a specific track maps to specific ES-9 outputs (channel map, percussion map, stepped-mod encoding). Tracked in `openspec/changes/research-daw-interchange-options` and a future manifest-format proposal.
