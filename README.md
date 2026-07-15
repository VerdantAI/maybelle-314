# Maybelle 314

Maybelle 314 is a planned Raspberry Pi 5 performance controller for a Eurorack system. It is intended to behave like a classical MIDI/CV controller with stored sequences: it follows the rack clock, reads authored sequence data, and emits control voltage, gates, triggers, and modulation through an Expert Sleepers ES-9. The Pi is not the rack voice and is not a sampler.

## Development Approach

This software is being developed with the active use of coding agents. Agents are used to help research platform choices, maintain OpenSpec planning artifacts, draft implementation work, edit code, run validation commands, and prepare commits. Human review remains part of the workflow, especially for hardware assumptions, music licensing, runtime safety, and production decisions.

## Current Direction

- Target hardware: Raspberry Pi 5, Expert Sleepers ES-9, 5-inch display, Eurorack control sources.
- Master clock: Pamela's Pro Workout. The Pi follows external clock/start/reset signals rather than owning tempo.
- Authoring workflow: tracks are authored in Ardour on another computer and loaded onto the Pi, likely as song bundles containing MIDI files plus metadata.
- Runtime controls: the rack provides CV inputs for song banks, song selection, channel banks, transport, and related performance parameters.
- Selection model: incoming CV is quantized to the number of available choices, similar to Assimil8or-style selection behavior, with debounce/hysteresis/latching to avoid unstable changes.
- Output model: the Pi outputs pitch CV, gates, triggers, stepped modulation, and other control signals through the ES-9 into the rack.
- Deferred hardware: Assimil8or support is a later spike. It may eventually affect sample/preset management, but it is not available for the first platform decision.

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

- What exact event types does Ardour export in the MIDI files we will use?
- How will Pamela's clock/start/reset arrive at the ES-9: pulses, gates, divisions, or another signal shape?
- Which ES-9 input/output API gives stable enough timing on Raspberry Pi OS?
- Should the first image use Raspberry Pi OS Desktop for speed of validation, Lite for appliance behavior, or a custom image after the stack is proven?
- Which runtime CV parameters are required for the first performance workflow?

## Planning Artifacts

- `openspec/changes/decide-base-platform/proposal.md`
- `openspec/changes/decide-base-platform/design.md`
- `openspec/changes/decide-base-platform/specs/base-platform-decision/spec.md`
- `openspec/changes/decide-base-platform/tasks.md`
