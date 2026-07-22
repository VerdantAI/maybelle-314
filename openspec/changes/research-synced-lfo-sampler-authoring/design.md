## Context

Maybelle 314 is a clock-following MIDI/CV controller that reads authored sequence data and emits CV, gates, triggers, and modulation through the ES-9. Modulation is a problem for the authoring path: the supported DAWs (Ardour and Bitwig, with Bitwig as the current test bench) do not export continuous modulation through MIDI in a portable way. Bitwig's `File > Export MIDI` carries notes and velocity only.

The use case that motivates this research is a tempo-synced LFO, and specifically *when to trigger it*. The proposed pattern separates the LFO into two parts along the line of what MIDI export can and cannot carry:

- The LFO **shape** (a continuous curve) is authored in the DAW, baked to a waveform file, and stored in a sampler such as the Rossum Assimil8or. This does not need to survive MIDI export.
- The LFO **trigger timing** is authored as a note on a dedicated MIDI track. This survives Bitwig/Ardour MIDI export cleanly, and Maybelle converts it to a gate/trigger out the ES-9 into the sampler channel's trig-in.

Initial research suggests:
- Adventure Kid AKWF single-cycle waveforms are mono, 16-bit, 44.1 kHz WAV files, single-cycle (600-sample convention, not mandatory). The library is available as CC-BY 3.0 (adventurekid.se, credit Kristoffer Ekstrand / Adventure Kid) or CC0 (the AKWF-FREE GitHub mirror). Waveforms Maybelle generates itself carry no license entanglement.
- The Assimil8or has no dedicated LFO section: a custom LFO is a loaded sample. Each channel has a gate/trigger input and three CV inputs assignable to parameters including Pitch, Sample Start, Loop Start, Loop Length, Phase Modulation, and Release. Playback is one-shot or gated with an attack/release envelope.
- Users sync a looping sample to an external clock by matching loop length to a musical duration and clocking the trig-in (e.g. a /8 clock for a 4-bar loop, /4 for 2 bars), optionally feeding a synced ramp into a Sample-Start/Scrub CV; re-triggering each cycle or phrase corrects drift, and driving Pitch CV from a clock-derived voltage makes playback rate follow tempo.
- Existing Assimil8or configurators (A8Manager by Chris Roberts / cpr2323, plus jpgsloan/Assimil8, DJmerkury/assimil8orMaker, digitalohm/assimil8or-helpers, biomassa/assimil8or) all ship without a license file, i.e. all rights reserved. None can be legally vendored, forked, or bundled.

## Goals / Non-Goals

**Goals:**
- Define a repeatable research process for authoring tempo-synced LFO waveforms and preparing sampler presets.
- Pin down the waveform formats to support: single-cycle (steady repeating LFO) and multi-bar bounce of DAW automation (complex synced arc).
- Document how the Assimil8or plays and syncs a stored waveform as an LFO, and how Maybelle's trigger note keeps it phase-locked to Pamela's clock.
- Establish the licensing boundary: which existing tools can only be pointed to, and what a clean-room preset writer must implement.
- Produce decision evidence for the song-bundle metadata, ES-9 trigger allocation, and the deferred Assimil8or runtime spike.

**Non-Goals:**
- Implement waveform generation, WAV export, or a preset writer.
- Vendor, fork, or redistribute any existing Assimil8or configurator's source.
- Require the Assimil8or module to be present for the authoring tooling to be useful.
- Decide the full runtime sample-playback or Assimil8or integration model.

## Decisions

### Split the LFO into DAW-baked Shape and MIDI-authored Trigger

The waveform shape is baked in the DAW and stored in the sampler; the trigger timing is authored as a dedicated MIDI note track that Maybelle emits as a gate/trigger.

Rationale: this keeps everything Maybelle must emit inside the portable subset of MIDI export (a note), offloads the continuous curve to the sampler, and keeps the workflow DAW-agnostic across Ardour and Bitwig.

Alternatives considered:
- Carry the LFO as MIDI CC/automation: not portable across the supported DAWs (Bitwig drops it on export).
- Have Maybelle synthesize the LFO voltage itself in real time: possible for simple shapes, but raises the Pi's real-time CV load and duplicates what the sampler already does well.

### Support Two Waveform Flavors

The tooling research covers single-cycle waveforms (looped, for steady repeating LFOs) and multi-bar bounces of DAW automation (one-shot, for evolving synced arcs).

Rationale: the two cover the common LFO needs; single-cycle matches AKWF conventions, and a multi-bar bounce is the most faithful capture of a DAW-designed synced modulation.

Alternatives considered:
- Single-cycle only: cannot represent a modulation curve that evolves over a phrase.
- Multi-bar only: wasteful for a simple repeating shape that a single cycle plus clock division expresses.

### Point to Configurators, Clean-room the Writer

Because every existing Assimil8or configurator is all-rights-reserved, the tooling will point to A8Manager (with credit to Chris Roberts) for hands-on GUI editing and implement a clean-room preset writer, from the Rossum manual, for content Maybelle generates.

Rationale: the preset *data format* is not copyrightable, but the existing tools' *source* is off-limits, and the README requires MIT-compatible licensing. Pointing plus clean-room keeps Maybelle permissive while still leveraging the mature external tool.

Alternatives considered:
- Vendor/fork an existing tool: legally blocked by the missing license.
- Shell-launch the user's installed A8Manager: acceptable as an optional convenience, but not a substitute for a writer Maybelle owns.

## Risks / Trade-offs

- Baked waveform drifts against the rack clock -> Mitigation: match loop length to a musical duration and re-trigger each cycle/phrase; optionally track tempo via Pitch CV.
- Waveform content licensing is mishandled -> Mitigation: prefer self-generated or CC0 waveforms; where CC-BY AKWF content is used, carry attribution to Kristoffer Ekstrand.
- Configurator licensing is mishandled -> Mitigation: never bundle configurator source; point/launch only, and clean-room the writer from the manual.
- Clean-room preset writer diverges from the real format -> Mitigation: validate generated presets against the Assimil8or manual and against files A8Manager reads without error.
- Scope creep into full Assimil8or runtime integration -> Mitigation: keep this change to offline authoring-side tooling research only.

## Migration Plan

No runtime migration is required because this change adds planning artifacts only. Research output should feed the song-bundle metadata design, ES-9 trigger allocation, and the deferred Assimil8or runtime spike.

Rollback is limited to removing this OpenSpec change before implementation.

## Open Questions

- What is the minimal Assimil8or preset structure needed to define an "LFO channel" (loaded waveform, loop vs one-shot, trig-in retrigger routing, Pitch-CV tempo tracking)?
- For a synced LFO, what loop-length-to-clock-division mapping does the tooling emit, and how is the trigger note quantized against Pamela's clock?
- Should multi-bar bounces be produced by a DAW render/bounce step, by an internal waveform generator, or both?
- Which ES-9 outputs are allocated to LFO trigger/gate duty, and how does that interact with the pitch/gate/velocity outputs from the main sequence?
- Does A8Manager (or any listed tool) expose a documented preset format or file that can serve as a clean-room reference target without reusing its source?
- How is waveform attribution (CC-BY AKWF vs CC0 vs self-generated) recorded in the song bundle or tooling output?
