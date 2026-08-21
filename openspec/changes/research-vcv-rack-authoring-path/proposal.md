## Why

The authoring workflow has broadened. **Sequences and CVs are now authored in VCV Rack**, which becomes the primary test bench, while **Bitwig Studio is the default file editor** and **Ardour is retained** — and the project intends to support any other authoring tool it reasonably can. VCV Rack natively produces the exact signal domain Maybelle emits — pitch CV, gates, triggers, and continuous modulation — so modulation no longer has to survive a MIDI round-trip, which was the constraint that shaped the entire prior model.

Adding VCV Rack does not narrow the interchange contract; it **stress-tests** it. The DAW-interchange research already concluded that Maybelle should consume a normalized, source-agnostic song bundle rather than any one tool's native format. VCV Rack is the hardest producer to normalize, because **a `.vcv` patch is a program, not a recording** — a running graph of modules with internal state, third-party plugin dependencies, and possible randomness, rather than a passive list of timed events. If the bundle contract holds for VCV Rack, it holds for the DAWs.

The open question is therefore not "which tool wins" but **what artifact moves from VCV Rack into Maybelle, and by what path**. The working assumption is that VCV-authored material becomes **canned support tracks** that Maybelle stores and plays back into the rack through the ES-9. Two constraints already narrow the answer: the project wants to capture **signals and sequence, not baked audio**, and playback is clocked by **Pamela's Pro Workout**, with the authored BPM carried only as reference. Together these mean the captured artifact must live in **musical time**, not wall-clock time. The ES-9 is the obvious output vehicle, but it should be confirmed against alternatives rather than assumed. The direction is still being iterated, so this spike is scoped to produce evidence and a recommendation, not to close the question.

## What Changes

- Establish **VCV Rack as a first-class authoring source and the primary test bench**, alongside Bitwig (default editor), Ardour (retained), and other tools as they are validated. This **reinforces** the source-agnostic bundle contract from `research-daw-interchange-options` rather than replacing it; the SMF path stays fully supported.
- Record the **licensing decision as settled**: Maybelle does not ship, bundle, host, or redistribute VCV Rack or any Rack engine. Users install and run VCV Rack themselves, and **Maybelle sits strictly downstream of it**. This removes the GPL/commercial-licensing hazard and takes "run a Rack engine on the Pi" off the table as something the project delivers.
- Draw the distinction the research must keep separate:
  - **The ES-9 as Maybelle's output stage** — already decided, unchanged, and shared by every authoring source.
  - **The interchange artifact and capture path from VCV Rack into Maybelle** — open, and the actual subject of this spike.
- Establish that captured material is **signals and sequence in musical time**, not a baked audio render. A render is fixed to wall-clock samples and cannot be re-clocked to the rack without varispeed or resampling, which is precisely what following Pamela's clock requires. The authored BPM travels as reference metadata describing the grid the material was written against.
- Survey and compare **capture/transfer paths** from VCV Rack into Maybelle as canned support tracks:
  - **MIDI out of VCV Rack** (the "if possible" case) — Rack's `CV-MIDI`-style modules driving a MIDI port or file capture, producing an SMF that reuses the existing MIDI→CV engine and manifest work outright.
  - **Patch transcription** — parse `patch.json` and lift sequencer state (step CV, gate types, probability, ties) into the manifest.
  - **Tempo-relative CV/automation capture** — record continuous modulation as breakpoints or a per-beat grid rather than as sample-rate audio.
  - **Hybrid** — transcribe or capture discrete sequence data as events, and continuous modulation as automation curves.
- Survey **alternatives to the ES-9** as the CV/gate transfer and output vehicle — other DC-coupled interfaces, ADAT-based Expert Sleepers combinations, and MIDI→CV hardware — so the ES-9 choice is confirmed by comparison rather than by default.
- Inventory the **`.vcv` file format**: container structure, `patch.json`, module/plugin identity and version pinning, per-module state, bundled assets, and behavior when a referenced plugin is missing.
- Resolve the **clock-authority conflict**: Maybelle follows Pamela's Pro Workout, but a VCV patch owns its own engine clock and sample rate. Determine how each capture path expresses position in musical time, how the authored BPM is recorded and used, and which paths cannot follow an external clock at all.
- Resolve the **runtime-CV question**: how rack-driven song-bank/song/channel-bank/transport selection applies to canned support tracks.
- Capture **sample references** for on-board samplers: the name, location, format, and content identity of the `.wav` assets authored material depends on, plus how those references are carried in the bundle and bound to a destination on the sampler. The default sampler is the **Rossum Assimil8or**. Provisioning the sampler's own storage is scoped to a separate change; this one only establishes what must be captured and represented.
- Define what **"canned support track"** means as a runtime concept, and how it relates to the existing voice/output mapping model in `decide-song-bundle-manifest`.
- Produce a **recommendation with decision evidence**, naming which parts of `decide-song-bundle-manifest` and `decide-base-platform` need revision.
- Do not implement a `.vcv` parser, renderer, capture tool, importer, or playback runtime in this change.

## Capabilities

### New Capabilities

- `vcv-rack-authoring-path-research`: Defines how the project researches and records the path from VCV Rack into Maybelle and onward into the Eurorack — covering the `.vcv` source format, the capture/transfer paths that produce canned support tracks, the ES-9 and its alternatives as the output vehicle, clock and runtime-CV authority, and the multi-source interchange contract.

### Modified Capabilities

- None. (`daw-interchange-research` and `song-bundle-manifest` are affected in substance, but both live in unarchived changes with no deployed spec under `openspec/specs/`; their revision is handled by the follow-on decision change rather than by a delta spec here.)

## Impact

- Adds OpenSpec planning artifacts for the VCV Rack authoring-path research spike.
- **Extends** `research-daw-interchange-options` with a fourth producer whose export story is materially different from a DAW's, and validates its source-agnostic-bundle recommendation against the hardest case.
- **Extends** `decide-song-bundle-manifest`: the manifest must describe canned support tracks and a VCV-sourced input, not only manifest-plus-SMF. The voice/output mapping, banks, selection, and calibration model all still stand.
- **Reopens one `decide-base-platform` decision point** — the song-bundle format. The Python core, the ES-9-as-class-compliant-USB-device I/O stack, and the kiosk/web UI decisions are unaffected, because the licensing decision keeps any Rack engine off the Pi. The ES-9 timing bench remains the gating hardware spike.
- Affects `docs/authoring/`, whose headline rule is "Author MIDI, not CV" and which currently covers only Bitwig and Ardour. It needs a VCV Rack page and a revised framing once this spike lands.
- Feeds the new `research-sampler-sample-sync` change, which owns provisioning the Assimil8or's SD card from the authoring side. This change supplies the sample references; that one consumes them.
- Touches `research-synced-lfo-sampler-authoring`: sampler-baked LFOs existed to work around MIDI's inability to carry modulation. If VCV-authored modulation can be captured directly, that workaround may be reducible in scope for VCV-sourced material while remaining necessary for the MIDI path.
- No production code, runtime dependencies, or hardware integration are introduced by this proposal.
