# Maybelle 314

Maybelle 314 is a planned Raspberry Pi 5 performance controller for a Eurorack system. It is intended to behave like a classical MIDI/CV controller with stored sequences: it follows the rack clock, reads authored sequence data, and emits control voltage, gates, triggers, and modulation through an Expert Sleepers ES-9. The Pi is not the rack voice and is not a sampler.

## Design Principle: stay small

Maybelle 314 should be **as small as possible**, filling gaps with existing permissively-licensed projects rather than rebuilding them. The burden of proof sits on building: a proposal that specifies implementation work should record what was considered for reuse and why it was rejected. The register of candidates, the admission criteria, and the short list of capabilities that genuinely have no substitute live in `openspec/changes/decide-build-vs-reuse/`.

Two corollaries the register has already had to apply: **check the license in the repository, not in the write-up** — A8Manager is widely described as open source but carries no license at all, so it cannot be a dependency. And **reuse is not reuse-at-any-size** — a small app with forty dependencies is not small, so a large binary pulled in to do a small job is weighed against the stdlib code it would replace.

### Upstream we are tracking

- **[A8Manager issue #165](https://github.com/cpr2323/A8Manager/issues/165)** — we have asked Chris Roberts whether he is open to attaching an MIT or other open-source license to [A8Manager](https://github.com/cpr2323/A8Manager). A8Manager manages Assimil8or presets *and* sample files and is functionally a strong match for work in `research-sampler-sample-sync`, but the repository currently carries no license, so it cannot be a dependency, wrapped, vendored, or invoked as a shipped component — and a Linux-build contribution would have undefined terms for both sides. Pointing users at it is unaffected either way. **If this issue resolves with an open-source license, `research-sampler-sample-sync` should be re-scoped again** — the card-layout, filename, and WAV-constraint work currently in its scope may become reuse instead of research. A Linux build (absent today, and which the author notes JUCE should make straightforward) becomes worth contributing at that point too.

## Development Approach

This software is being developed with the active use of coding agents. Agents are used to help research platform choices, maintain OpenSpec planning artifacts, draft implementation work, edit code, run validation commands, and prepare commits. Human review remains part of the workflow, especially for hardware assumptions, music licensing, runtime safety, and production decisions.

## Current Direction

- Target hardware: Raspberry Pi 5, Expert Sleepers ES-9, official Raspberry Pi Touch Display 2 (5-inch, 720x1280 portrait, DSI, 5-finger capacitive), Eurorack control sources.
- Master clock: Pamela's Pro Workout. The Pi follows external clock/start/reset signals rather than owning tempo.
- Authoring workflow: **multi-source**. VCV Rack, Bitwig Studio, Ardour, and other tools where practical all feed the same source-agnostic song bundle. **Sequences and CVs are authored in VCV Rack**, which is now the primary test bench; **Bitwig is the default file editor**; Ardour is retained. Adding VCV Rack widens the contract rather than replacing it — the MIDI/SMF path stays fully supported. Bitwig's MIDI export constraint (notes and velocity only, no automation/CC/note expressions) still applies to that path but is no longer the binding constraint on modulation authoring.
- Capture model: Maybelle stores **canned support tracks** — **signals and sequence, not baked audio**. A baked render is fixed to wall-clock samples and cannot be re-clocked; captured material is therefore positioned in **musical time** (bars/beats/ticks), with the authored BPM carried as reference metadata only. Pamela's Pro Workout governs playback tempo. The open problem is that a `.vcv` patch is a *program*, not a *recording* — see `research-vcv-rack-authoring-path`. Direction is still being iterated.
- VCV Rack licensing boundary (decided): the project **does not ship, bundle, host, link, or redistribute VCV Rack or any Rack engine**. Users install and run VCV Rack themselves and Maybelle sits strictly downstream. This removes the GPL/commercial licensing hazard, keeps any Rack engine off the Pi, and leaves Maybelle in sole ownership of the ES-9 duplex stream — including the input channels used for rack CV and clock.
- Sample assets: authored material may reference `.wav` samples played by an on-board sampler in the rack — currently the **Rossum Assimil8or** as the default. The bundle captures each sample's name, location, format, and content identity, and an offline authoring-side tool syncs the sampler's SD card to match. Maybelle emits the gate/trigger and modulation CV through the ES-9; it does not read, serve, or play sample content. See `research-sampler-sample-sync`.
- Companion content tooling: an offline, authoring-side toolset (separate from the Pi runtime) for creating tempo-synced LFO/modulation waveforms — Adventure Kid AKWF-style single-cycle shapes or multi-bar bounces of DAW automation — and for preparing Rossum Assimil8or presets. The LFO's shape is baked into a waveform stored in the sampler; its trigger timing is authored as a plain note in the MIDI file, which Maybelle emits as a gate/trigger through the ES-9 into the sampler's trig-in. The tooling points to the external A8Manager configurator (credit: Chris Roberts) for hands-on editing and includes a clean-room preset writer for generated content, keeping licensing MIT-compatible. Note that A8Manager currently carries **no license** — see [Upstream we are tracking](#upstream-we-are-tracking).
- Runtime controls: the rack provides CV inputs for song banks, song selection, channel banks, transport, and related performance parameters.
- Two operating contexts: **Performance** (at the rack, on the 5-inch touchscreen — glanceable, real-time, robust) and **Backstage** (configuration of triggers, channels, banks, mappings, and calibration, done on a computer/laptop at standard size). Research direction: one shared core with two role-specific responsive views that can run simultaneously (client/server), plus a safety gate so Backstage edits never disrupt live output — not two separate apps and not a single-screen toggle.
- Selection model: incoming CV is quantized to the number of available choices, similar to Assimil8or-style selection behavior, with debounce/hysteresis/latching to avoid unstable changes.
- Output model: the Pi outputs pitch CV, gates, triggers, stepped modulation, and other control signals through the ES-9 into the rack.
- Deferred hardware: Assimil8or runtime/sample playback remains a later spike and is not available for the first platform decision. Its storage and WAV format constraints are being pinned down ahead of it by `research-sampler-sample-sync`. Authoring-side content tooling for it (LFO waveforms and preset preparation) is now in scope, however, as offline tooling that does not depend on the module being present.

## Base Platform Decision

The active OpenSpec change is `decide-base-platform`. It exists to decide:

- Raspberry Pi OS baseline and image strategy.
- ES-9 I/O stack: JACK, PipeWire/JACK, PortAudio/sounddevice, ALSA direct, or another path.
- Runtime language/framework: Python plus a small local UI, Node/lightweight JavaScript, or Tauri/Rust.
- Package set with MIT-compatible/permissive licensing.
- Song bundle and manifest format (now specified in `decide-song-bundle-manifest`).
- Kiosk/display approach for the 5-inch screen.

The decision is now recorded in `openspec/changes/decide-base-platform/decision-record.md`: a **Python core + local web/kiosk UI (Chromium) on Raspberry Pi OS Bookworm**, with the **ES-9 as a class-compliant USB audio device** (PortAudio/`sounddevice` or ALSA), a **FastAPI local server + SSE** for the Performance/Backstage web surfaces, and **file-based song bundles**. The recommendation is final for language, UI, persistence, OS, and dependencies; the **ES-9 I/O timing bench remains the gating spike** that finalizes the I/O layer (with a Rust/C timing-core fallback if Python timing proves insufficient).

## Open Questions

- How does VCV-authored material get captured as signals and sequence — as MIDI out of VCV Rack, as transcribed `patch.json` sequencer state, as a tempo-relative CV/automation stream, or a hybrid? (The central question of `research-vcv-rack-authoring-path`.)
- Can MIDI be captured out of VCV Rack at all, and with what fidelity? A positive answer reuses the existing MIDI→CV engine, manifest, and authoring docs unchanged.
- What representation carries continuous CV in musical time, at what resolution and interpolation — breakpoints, automation segments, a per-beat grid, or stepped events?
- What does the authored BPM govern? (Confirmed: not playback timing. Open: validation, display, free-running fallback, or all three.) What happens when the rack clock is absent or stops?
- Is the ES-9 the best vehicle for getting authored triggers/CV/MIDI into the rack, or does the survey of DC-coupled interfaces and MIDI→CV hardware surface a better fit?
- What are the Assimil8or's actual card, layout, filename, and WAV format constraints?
- What exact event types does each authoring DAW export in the MIDI files we will use, for the retained SMF path? (Known for Bitwig: notes + velocity only, from the Arrangement, as a Type-1 SMF; Ardour still to be inventoried.)
- How is a synced-LFO trigger's timing authored and emitted — as a dedicated MIDI note track that Maybelle converts to a gate/trigger through the ES-9 — and how does re-triggering keep the sampler-side LFO phase-locked to Pamela's clock?
- How will Pamela's clock/start/reset arrive at the ES-9: pulses, gates, divisions, or another signal shape?
- Which ES-9 input/output API gives stable enough timing on Raspberry Pi OS?
- Should the first image use Raspberry Pi OS Desktop for speed of validation, Lite for appliance behavior, or a custom image after the stack is proven?
- Which runtime CV parameters are required for the first performance workflow?

## Documentation

- `docs/authoring/` — authoring guide for VCV Rack, Bitwig, and Ardour, written for both users and assisting agents. Start at `docs/authoring/README.md`. Includes `vcv-rack.md` (authoring in VCV Rack and capturing to MIDI via the Chinenual MIDI Recorder) and a sectioned `faq.md`. Backstage mode surfaces this same guidance as in-app diagrams. VCV Rack material is marked **(provisional)** against `research-vcv-rack-authoring-path`.

- `examples/vcv-rack/` — example VCV Rack patches used as test fixtures for the authoring-path research. `prog-riff-v1.vcv` is the first: a 7-step riff at 137 BPM using Impromptu PhraseSeq16, GateSeq64, and Clocked. Its inventory is recorded in `openspec/changes/research-vcv-rack-authoring-path/research-findings.md`.

## Planning Artifacts

- `openspec/changes/research-vcv-rack-authoring-path/` — **active**: the path from VCV Rack into Maybelle and onward into the rack — capture paths, musical-time artifacts, the ES-9 and its alternatives, and sample-reference capture. Extends the two changes below rather than replacing them.
- `openspec/changes/research-sampler-sample-sync/` — **active**: authoring-side syncing of the on-board sampler's SD card (Assimil8or default) with the samples a song bundle references.
- `openspec/changes/decide-build-vs-reuse/` — **active**: the reuse-before-build principle, the register of existing projects per capability (A8Manager, rclone, Chinenual, the Python stack), and the irreducible core the project must own.
- `openspec/changes/research-vcv-rack-companion-module/` — **active**: whether to build and maintain a Maybelle VCV Rack module for song-bundle import/export. Effort, licensing, and distribution researched; leaning toward a Python converter first.
- `openspec/changes/decide-base-platform/` — active base-platform decision (OS image, ES-9 I/O stack, runtime language, song-bundle format). Only the bundle-format point is reopened; the Python core, ES-9 I/O stack, and kiosk/web UI decisions are unaffected.
- `openspec/changes/research-daw-interchange-options/` — DAW export/interchange research across Ardour and Bitwig. Its source-agnostic-bundle recommendation is now being stress-tested against VCV Rack.
- `openspec/changes/decide-song-bundle-manifest/` — the v1 song-bundle + manifest format (the central data contract binding authored material to ES-9 output, banks, selection, and modulation). Being extended to carry VCV-sourced material and sample references; the voice/output mapping, banks, selection, and calibration model all still stand.
- `openspec/changes/research-synced-lfo-sampler-authoring/` — tempo-synced LFO waveform authoring and Assimil8or content tooling research.
- Additional research spikes live under `openspec/changes/` (ES-9 profiles, Pi port topology, agent control surface, test-music fixtures, research roadmap).
