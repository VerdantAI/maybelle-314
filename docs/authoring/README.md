# Maybelle 314 — Authoring Guide

Guidance for setting up tracks in a DAW (Bitwig or Ardour) so they play correctly through Maybelle 314 into a Eurorack rack via the Expert Sleepers ES-9.

> **Audience:** this guide is written for both **end users** authoring music and **AI agents** assisting those users. Agents should treat this directory as the reference for "how do I set up my DAW for Maybelle/ES-9?" questions and cite the specific page.
>
> **Status:** Maybelle 314 is in the planning/OpenSpec phase. The *concepts* here (MIDI→CV model, what DAWs export, ES-9 output facts) are established and stable. The exact **bundle/manifest conventions** (how a track is mapped to an ES-9 output) are **provisional** and tracked in OpenSpec — those spots are marked **(provisional)**. The source of truth for anything provisional is the relevant change under `openspec/changes/`.

## The one thing to understand first

**Maybelle is the MIDI→CV converter. The ES-9 is just the output stage (a DC-coupled audio interface used as a DAC). You author plain MIDI in the DAW; Maybelle turns each note into the control voltages and gates that appear on the ES-9's jacks.**

You do **not** build CV/gate/velocity "wiring" in the DAW. You author notes; Maybelle fans each voice out to physical outputs.

## Headline question: "Do I need to export pitch, gate, and velocity as 3 separate wires?"

**No — not in the DAW.** A single MIDI note already carries all three:

| MIDI note carries | Becomes at the ES-9 |
| --- | --- |
| Note number (pitch) | Pitch CV (1V/oct) on one output |
| Note on/off (duration) | Gate (or trigger) on another output |
| Velocity | Velocity CV on a third output *(optional)* |

The **split into separate wires happens inside Maybelle**, per your output configuration — exactly like a hardware MIDI→CV converter (Doepfer A-190-3, Kenton, Expert Sleepers FH-2) assigns pitch to one jack, gate to another, velocity to a third. In the DAW you make **one monophonic track per voice**; Maybelle maps that track to its pitch/gate/velocity outputs on the ES-9.

See [midi-to-cv-model.md](midi-to-cv-model.md) for the full model.

## Contents

- **[midi-to-cv-model.md](midi-to-cv-model.md)** — the conceptual model (DAW-agnostic): notes → CV/gate/velocity, mono voices, gate vs trigger, modulation, clock.
- **[bitwig.md](bitwig.md)** — setting up and exporting from Bitwig Studio.
- **[ardour.md](ardour.md)** — setting up and exporting from Ardour.
- **[faq.md](faq.md)** — quick question/answer reference for agents.

## Key facts to anchor every answer

- **Author MIDI, not CV.** One monophonic track = one voice = (in Maybelle) one pitch CV + one gate + optional velocity CV.
- **Export = Standard MIDI File (SMF).** The portable common denominator across both DAWs is **notes + velocity**. Bitwig exports *only* notes+velocity; Ardour can export more but notes+velocity is what you should rely on for portability.
- **Author modulation as notes**, not as DAW automation, if it must survive export (Bitwig drops automation/CC on export). See [midi-to-cv-model.md](midi-to-cv-model.md#modulation).
- **ES-9 outputs:** the 8 front-panel 3.5mm jacks are DC-coupled CV/gate outputs (DAW channels 9–16 → outs 1–8), ~±10V. Never use the two 1/4" Main outputs for CV (they are AC-coupled). Maybelle handles 1V/oct pitch calibration in software.
- **Clock:** Maybelle follows the rack's master clock (Pamela's Pro Workout). The tempo in your DAW file is a *grid* for note positions; Maybelle re-clocks playback to the rack.
- **Where routing lives:** the track→ES-9-output mapping is Maybelle configuration (edited in "Backstage" mode on a laptop), **not** something you set in the DAW. **(provisional — see `openspec/changes/research-daw-interchange-options` and the future manifest spec.)**
