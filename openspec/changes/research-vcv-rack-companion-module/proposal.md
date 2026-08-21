## Why

The capture path from VCV Rack is solved: the Chinenual MIDI Recorder writes a Standard MIDI File from inside the patch, and `research-vcv-rack-authoring-path` recommends it. But an SMF is only half of what Maybelle needs. The other half — the **manifest**: voice-to-ES-9-output binding, track naming, authored BPM, selection config, sample references — has no authoring-side home at all. Today a user captures MIDI in Rack and then hand-assembles the rest in Backstage, with nothing validating that the two agree until the material reaches the rack.

A **Maybelle VCV Rack module** could close that gap by exporting a complete, validated song bundle straight from the patch, and by importing one back for editing. Before committing to it, we need the level of effort, the format obligations on both sides, the licensing position, and an honest comparison against cheaper alternatives — most obviously a standalone converter written in the project's existing Python.

This also revisits a boundary. `research-vcv-rack-authoring-path` settled that the project does not ship VCV Rack. Shipping a *plugin* does not ship Rack, and VCV's Non-Commercial Plugin License Exception permits an MIT-licensed plugin provided it is free. But it would put the project's name on a **C++ artifact inside a third-party ecosystem**, on a platform-specific build matrix, tracking someone else's API. That is a real change to a project whose runtime surface is otherwise Python, and it deserves a decision rather than a drift.

## What Changes

- Add a research spike assessing whether the project should build, distribute, and maintain a **VCV Rack module for Maybelle bundle import/export**.
- Establish the **level of effort**: language and API, SDK and toolchain, per-platform build matrix, panel/SVG and manifest obligations, VCV Library submission, and ongoing maintenance across Rack releases.
- Establish the **licensing position**: whether an MIT-licensed module is permitted, on what conditions, and what would forfeit that — including the effect of incorporating Rack code and of ever charging for it.
- Define the **formats on both sides**: what the module would read and write toward Maybelle (SMF, manifest, bundle packaging, sample references) and what Rack itself obliges (plugin manifest, panel assets, module state serialization).
- Scope the module against what already exists: identify what it would add **beyond** the Chinenual MIDI Recorder, so the proposal is not re-solving MIDI capture.
- Compare against **lower-effort alternatives**: doing nothing and authoring the manifest in Backstage; a standalone converter in the existing Python core; contributing to an existing plugin; or the module.
- Assess the **sequencing risk** of building a C++ artifact against a manifest format that is not yet settled.
- Produce a **build / defer / decline recommendation** with the conditions that would change it.
- Do not implement a module, a converter, a panel, or any bundle writer in this change.

## Capabilities

### New Capabilities

- `vcv-rack-companion-module-research`: Defines how the project researches and records the effort, formats, licensing, and alternatives for a Maybelle-authored VCV Rack module providing song-bundle import and export.

### Modified Capabilities

- None.

## Impact

- Adds OpenSpec planning artifacts for the companion-module investigation.
- Consumes `research-vcv-rack-authoring-path` (the capture path and its recommendation) and `decide-song-bundle-manifest` (the format any exporter must write).
- Feeds `research-sampler-sample-sync`, since sample references captured at authoring time would be a natural thing for such a module to emit.
- If a module is recommended, it introduces the project's **first C++ artifact, first cross-platform binary build matrix, and first third-party-ecosystem distribution channel** — none of which exist today. `decide-base-platform` chose a Python core and is not otherwise affected, since this is authoring-side tooling that never runs on the Pi.
- Revisits, without overturning, the distribution boundary set in `research-vcv-rack-authoring-path`: the project still would not ship VCV Rack, but it would ship something that runs inside it.
- No production code, runtime dependencies, or hardware integration are introduced by this proposal.
