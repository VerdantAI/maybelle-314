# Maybelle 314

Maybelle 314 is a planned Raspberry Pi 5 performance controller for a Eurorack system. It is intended to behave like a classical MIDI/CV controller with stored sequences: it follows the rack clock, reads authored sequence data, and emits control voltage, gates, triggers, and modulation through an Expert Sleepers ES-9. The Pi is not the rack voice and is not a sampler.

## Development Approach

This software is being developed with the active use of coding agents. Agents are used to help research platform choices, maintain OpenSpec planning artifacts, draft implementation work, edit code, run validation commands, and prepare commits. Human review remains part of the workflow, especially for hardware assumptions, music licensing, runtime safety, and production decisions.

## Current Direction

- Target hardware: Raspberry Pi 5, Expert Sleepers ES-9, official Raspberry Pi Touch Display 2 (5-inch, 720x1280 portrait, DSI, 5-finger capacitive), Eurorack control sources.
- Master clock: Pamela's Pro Workout. The Pi follows external clock/start/reset signals rather than owning tempo.
- Authoring workflow: tracks are authored in a DAW on another computer and loaded onto the Pi, likely as song bundles containing MIDI files plus metadata. Ardour and Bitwig Studio are both supported authoring tools; Bitwig is the current test bench while Ardour support is deliberately retained. Bitwig's MIDI export carries notes and velocity only (no automation, CC, or note expressions), which shapes how modulation is authored — see the modulation/LFO approach below.
- Companion content tooling: an offline, authoring-side toolset (separate from the Pi runtime) for creating tempo-synced LFO/modulation waveforms — Adventure Kid AKWF-style single-cycle shapes or multi-bar bounces of DAW automation — and for preparing Rossum Assimil8or presets. The LFO's shape is baked into a waveform stored in the sampler; its trigger timing is authored as a plain note in the MIDI file, which Maybelle emits as a gate/trigger through the ES-9 into the sampler's trig-in. The tooling points to the external A8Manager configurator (credit: Chris Roberts) for hands-on editing and includes a clean-room preset writer for generated content, keeping licensing MIT-compatible.
- Runtime controls: the rack provides CV inputs for song banks, song selection, channel banks, transport, and related performance parameters.
- Selection model: incoming CV is quantized to the number of available choices, similar to Assimil8or-style selection behavior, with debounce/hysteresis/latching to avoid unstable changes.
- Output model: the Pi outputs pitch CV, gates, triggers, stepped modulation, and other control signals through the ES-9 into the rack.
- Deferred hardware: Assimil8or runtime/sample playback remains a later spike and is not available for the first platform decision. Authoring-side content tooling for it (LFO waveforms and preset preparation) is now in scope, however, as offline tooling that does not depend on the module being present.

## Base Platform Decision

The active OpenSpec change is `decide-base-platform`. It exists to decide:

- Raspberry Pi OS baseline and image strategy.
- ES-9 I/O stack: JACK, PipeWire/JACK, PortAudio/sounddevice, ALSA direct, or another path.
- Runtime language/framework: Python plus a small local UI, Node/lightweight JavaScript, or Tauri/Rust.
- Package set with MIT-compatible/permissive licensing.
- Song bundle and manifest format.
- Kiosk/display approach for the 5-inch screen.

The current bias is toward a Python-centered runtime with a browser/kiosk UI, but the ES-9 I/O spike is expected to drive the real decision.

## Open Questions

- What exact event types does each authoring DAW export in the MIDI files we will use? (Known for Bitwig: notes + velocity only, from the Arrangement, as a Type-1 SMF; Ardour still to be inventoried.)
- How is a synced-LFO trigger's timing authored and emitted — as a dedicated MIDI note track that Maybelle converts to a gate/trigger through the ES-9 — and how does re-triggering keep the sampler-side LFO phase-locked to Pamela's clock?
- How will Pamela's clock/start/reset arrive at the ES-9: pulses, gates, divisions, or another signal shape?
- Which ES-9 input/output API gives stable enough timing on Raspberry Pi OS?
- Should the first image use Raspberry Pi OS Desktop for speed of validation, Lite for appliance behavior, or a custom image after the stack is proven?
- Which runtime CV parameters are required for the first performance workflow?

## Planning Artifacts

- `openspec/changes/decide-base-platform/` — active base-platform decision (OS image, ES-9 I/O stack, runtime language, song-bundle format).
- `openspec/changes/research-daw-interchange-options/` — DAW export/interchange research across Ardour and Bitwig.
- `openspec/changes/research-synced-lfo-sampler-authoring/` — tempo-synced LFO waveform authoring and Assimil8or content tooling research.
- Additional research spikes live under `openspec/changes/` (ES-9 profiles, Pi port topology, agent control surface, test-music fixtures, research roadmap).
