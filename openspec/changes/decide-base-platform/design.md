## Context

The repository currently has OpenSpec configuration but no application code, existing capability specs, dependency manifests, or deployment configuration. The base-platform decision should therefore be made as a planning artifact before implementation introduces framework-specific structure.

This change establishes the decision workflow and required outputs. It does not select a platform by itself; it defines how the project will select one and what must be documented so implementation can proceed consistently.

The target product is a Raspberry Pi 5 performance application for a 5-inch display. Pamela's Pro Workout is the master clock; the Pi runtime must follow clock/start/reset/selection signals arriving through the ES-9 and schedule authored MIDI playback accordingly. The Pi also receives rack-provided CV runtime parameters for song banks, channel banks, selection, and related performance state. The Pi outputs control voltages, gates, triggers, and modulation signals through the ES-9 into the Eurorack; it is not responsible for generating the rack voice, final audio signal, or sampler behavior.

## Goals / Non-Goals

**Goals:**
- Define a repeatable process for evaluating candidate base platforms.
- Require the selected platform decision to cover runtime, language/framework, persistence, deployment, local development, testing, and operational constraints.
- Require the decision to account for external-clock following, rack-provided CV runtime parameters, MIDI-file interpretation, ES-9 CV/gate output, and Raspberry Pi kiosk/display operation.
- Require the decision to select a Raspberry Pi OS baseline or define the path to a custom image.
- Require the decision to identify MIT-compatible/permissive packages for core functionality and flag any licensing exceptions.
- Produce a durable decision record that later OpenSpec changes and implementation tasks can reference.
- Make trade-offs explicit enough to support revisiting the decision if project constraints change.

**Non-Goals:**
- Implement application scaffolding, dependency manifests, deployment files, or runtime code.
- Choose a specific platform without candidate analysis.
- Optimize for every possible future product direction before core constraints are known.

## Decisions

### Use a Decision Record as the Output

The base-platform decision will be captured in a repository document rather than only in discussion notes or issue comments.

Rationale: implementation changes need a stable, reviewable source of truth. A committed document can be referenced from future specs and tasks, reviewed in pull requests, and updated through normal change control.

Alternatives considered:
- Ad hoc chat or issue discussion: fast, but easy to lose and hard to enforce.
- Immediate implementation scaffold: creates momentum, but risks encoding an unreviewed platform choice into the codebase.

### Evaluate a Shortlist of Candidate Platforms

The decision process will compare a small set of credible candidates rather than surveying the full ecosystem.

Rationale: a focused shortlist keeps the decision practical while still exposing meaningful trade-offs. Candidate entries should include enough information to explain why the selected option fits better than rejected alternatives.

Alternatives considered:
- Single-candidate justification: cheaper, but weak at revealing assumptions.
- Exhaustive comparison matrix: thorough, but likely to delay the project before requirements are mature.

### Score Against Project-Relevant Criteria

The decision record will define criteria before making the recommendation. At minimum, criteria must cover fit to expected product shape, development velocity, maintainability, deployment complexity, data needs, testing approach, operational burden, ecosystem maturity, and team familiarity.

Rationale: explicit criteria keep the decision anchored to project needs instead of personal preference or default tooling.

Alternatives considered:
- Pick by popularity: useful signal, but insufficient for project-specific constraints.
- Pick by current familiarity only: reduces ramp-up cost, but may hide deployment or maintenance risks.

### Treat Implementation Impact as Part of the Decision

The decision record will summarize what implementation work follows from the selection, including expected repository structure, dependencies, local commands, environments, and deployment assumptions.

Rationale: a base-platform decision is only useful if it gives the next implementation change enough direction to avoid re-litigating core setup choices.

Alternatives considered:
- Keep the decision purely conceptual: cleaner, but leaves too much ambiguity for the first implementation task.

### Let Hardware I/O Spikes Drive the Framework Choice

The base-platform decision will evaluate GUI frameworks only after identifying the viable ES-9 I/O path for clock input, runtime CV input, and CV/gate output.

Rationale: this product is fundamentally a rack-controlled stored-sequence CV controller. UI technology is secondary to stable timing, multichannel ES-9 access, and deterministic runtime state handling.

Alternatives considered:
- Choose Flask, Node, or Tauri first: simple to discuss, but risks optimizing the easy part while leaving the I/O layer unresolved.
- Build a full application before the spike: creates code quickly, but may force a rewrite if the I/O stack changes.

### Treat Raspberry Pi OS as a Platform Component

The decision record will compare Raspberry Pi OS Desktop, Raspberry Pi OS Lite with a kiosk/display stack, and a later custom image path.

Rationale: OS choice affects ES-9 visibility, JACK/PipeWire/ALSA behavior, low-latency scheduling, display support, service startup, update strategy, and reproducibility.

Alternatives considered:
- Ignore OS until implementation: fast initially, but pushes hard integration risk later.
- Start with a custom image immediately: reproducible, but premature before the ES-9 and UI stacks are proven.

### Prefer MIT-Compatible and Permissive Dependencies

Candidate packages will be screened for MIT, BSD, Apache-2.0, PSF, or similarly permissive licenses, with exceptions called out explicitly.

Rationale: the project should keep distribution and appliance-image options simple. Known candidates include Mido, python-rtmidi, JACK-Client, sounddevice, NumPy, Pydantic, PyYAML, Flask, Gunicorn, Uvicorn, pytest, and Tauri.

Alternatives considered:
- Accept any technically useful dependency: expands choices, but creates avoidable licensing and redistribution review work.
- Require MIT-only dependencies: too restrictive because BSD, Apache-2.0, and PSF licensed packages are also practical and permissive for this use case.

## Risks / Trade-offs

- Premature selection before product requirements are clear -> Mitigation: document assumptions and revisit triggers in the decision record.
- Criteria become too generic to guide a real choice -> Mitigation: require candidate scoring and explicit rationale tied to the project context.
- Decision record drifts from implementation -> Mitigation: require future platform-scaffolding changes to reference the accepted decision.
- Over-analysis delays implementation -> Mitigation: limit the candidate shortlist and require only decision-critical evidence.
- UI framework choice hides I/O risk -> Mitigation: spike ES-9 input/output timing before committing to the runtime framework.
- OS image choice hides appliance risk -> Mitigation: record the baseline image, display stack, service manager approach, and path to a reproducible custom image if needed.
- Dependency licensing creates later redistribution friction -> Mitigation: record package licenses and flag non-permissive dependencies before adoption.

## Migration Plan

No runtime migration is required because this change adds planning artifacts only. After acceptance, the next implementation change should create or update the decision record, then subsequent scaffolding work should follow the selected platform.

Rollback is limited to reverting the OpenSpec change artifacts before implementation begins.

## Open Questions

- What product shape is the first platform decision optimizing for: Raspberry Pi performance runtime only, or a shared runtime plus authoring/SD-card companion tool?
- Which Raspberry Pi OS image should be flashed first: Desktop for fast validation, Lite plus kiosk packages for appliance behavior, or a custom image after the stack is proven?
- Which ES-9 I/O layer should be used for the first proof: JACK/PipeWire JACK, PortAudio through sounddevice, ALSA direct, or another path?
- Are there hosting, licensing, offline, privacy, or data-residency constraints that must be treated as hard requirements?
- Who is the approver for the base-platform decision record?
- What Standard MIDI File data will be exported from Ardour and treated as authoritative by the runtime: tempo, notes, CC, markers, program changes, pitch bend, time signatures, or other events?
- How will Pamela's clock be represented at the ES-9 input: audio pulse train, MIDI clock via an external converter, trigger/reset pulses, or another signal shape?
- What runtime parameter inputs are required at minimum: song bank, song selection, channel bank, start/run, reset, BPM/clock division, mute/enable, or other performance controls?
- How should incoming CV parameters be quantized, debounced, latched, and displayed so accidental voltage jitter does not change songs or channel banks mid-performance?
- If rendered CV stems are used, are they fixed-tempo modulation assets, manually selected per BPM range, regenerated live from MIDI/control data, or handled by external hardware that supports tempo-aware playback?
- Deferred: when an Assimil8or is available, evaluate whether the system should generate Assimil8or SD-card folders/presets and trigger sample playback through ES-9 outputs.
