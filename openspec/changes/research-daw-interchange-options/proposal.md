## Why

Maybelle 314 depends on authored musical data moving cleanly from a DAW workflow into a Raspberry Pi runtime that behaves like a clock-following MIDI/CV controller. Before choosing the bundle format or deciding whether Ardour must run on the Pi, we need evidence about DAW-on-Pi viability, Ardour export behavior, cross-DAW abstraction options, open DAW interchange formats, and existing hardware/software precedents.

## What Changes

- Add a research spike for Ardour-on-Raspberry-Pi and Pi-based DAW/audio distributions.
- Compare direct Ardour session usage, Ardour stem/MIDI export, DAWproject interchange, Standard MIDI Files, and Maybelle-specific song bundles.
- Investigate whether the Maybelle import model can abstract beyond Ardour to Ableton Live, Bitwig Studio, Studio One, Logic Pro, Reaper, and other major DAWs.
- Treat Ardour and Bitwig Studio as co-equal supported authoring workflows — Bitwig is the current test bench and Ardour support is retained — while identifying DAW-agnostic contracts that could support additional DAWs later.
- Record the Bitwig MIDI export constraint as decision evidence: `File > Export MIDI` emits notes and velocity only (no automation, MIDI CC, note expressions/MPE, micro-pitch, sustain, tempo, or Clip Launcher data), from the Arrangement, as a Type-1 SMF. Modulation therefore cannot ride through Bitwig's MIDI export as automation.
- Research existing sequencers/modules that import DAW-like files, MIDI files, CV/gate recordings, or SD-card projects.
- Define the decision evidence needed before the base-platform decision chooses an authoring/runtime interchange contract.
- Do not implement file import, conversion, playback, or DAW integration in this change.

## Capabilities

### New Capabilities
- `daw-interchange-research`: Defines how the project researches, compares, and records DAW/session interchange options for the Pi runtime and authoring workflow.

### Modified Capabilities
- None.

## Impact

- Adds OpenSpec planning artifacts for DAW interchange research.
- May inform the active `decide-base-platform` change, especially runtime language, OS image, package choices, and song bundle format.
- No production code, runtime behavior, dependencies, or hardware integration changes are introduced by this proposal.
