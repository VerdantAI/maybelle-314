## ADDED Requirements

### Requirement: Multi-source authoring contract
The project SHALL treat VCV Rack as an additional first-class authoring source without narrowing support for other tools.

#### Scenario: Tool roles are recorded
- **WHEN** the VCV Rack authoring-path research is performed
- **THEN** it records that VCV Rack authors sequences and CVs and is the primary test bench, that Bitwig Studio is the default file editor, that Ardour is retained, and that additional tools are supported where practical
- **AND** it records which prior constraints stop binding for VCV-sourced material, specifically Bitwig's notes-and-velocity-only MIDI export and the resulting inability to carry modulation
- **AND** it confirms the Standard MIDI File path remains fully supported rather than deprecated

#### Scenario: The bundle contract is tested against the hardest producer
- **WHEN** the source-agnostic song-bundle contract from `research-daw-interchange-options` is evaluated against VCV Rack
- **THEN** the research states which bundle fields VCV Rack can produce, which it cannot, and which it can produce only through a capture or conversion step
- **AND** it states whether the contract must change to admit VCV Rack, or whether VCV Rack fits the existing contract through capture
- **AND** it identifies which findings generalize to any future authoring source

### Requirement: Licensing boundary
The project SHALL keep VCV Rack outside its own distribution and dependency surface.

#### Scenario: The downstream boundary is honored
- **WHEN** any capture path or playback model is evaluated
- **THEN** it is evaluated on the basis that Maybelle does not ship, bundle, host, link, or redistribute VCV Rack or any Rack engine
- **AND** the user is assumed to install and run VCV Rack themselves, with Maybelle strictly downstream of it
- **AND** any option that would require the project to distribute or embed a Rack engine is recorded as out of scope rather than compared on merit

#### Scenario: Third-party plugin licensing is still recorded
- **WHEN** the research documents the authoring-side toolchain
- **THEN** it records the licenses of VCV Rack and of the third-party plugins in use, as user-facing information
- **AND** it confirms that no captured artifact or generated tooling inherits a license incompatible with the project's MIT-compatible constraint

### Requirement: Output stage and interchange artifact are distinguished
The project SHALL keep the ES-9's role as Maybelle's output stage separate from the question of what artifact moves from VCV Rack into Maybelle.

#### Scenario: The two questions are stated separately
- **WHEN** the research describes the end-to-end path
- **THEN** it states that the ES-9 is Maybelle's output stage for every authoring source, and that this is already decided
- **AND** it states that the open question is the interchange artifact and capture path between VCV Rack and Maybelle
- **AND** it does not treat a decision about one as a decision about the other

### Requirement: VCV patch file inventory
The project SHALL inventory what a VCV Rack patch file contains before relying on it as a source artifact.

#### Scenario: The `.vcv` container and patch data are inspected
- **WHEN** a representative VCV Rack patch is saved and inspected
- **THEN** the research records the container format, the structure of `patch.json`, how modules and their plugin slugs/versions are identified, what per-module state is persisted, and what bundled assets travel with the patch
- **AND** it records how sample rate, engine tempo, and cable topology are represented

#### Scenario: Portability and missing-plugin behavior are assessed
- **WHEN** a patch is opened on a machine lacking one or more referenced plugins
- **THEN** the research records the failure behavior and what patch data survives
- **AND** it states whether third-party plugin dependency — including the Impromptu Modular sequencers currently in use — makes `.vcv` a stable enough source artifact, and what pinning or vendoring would be required

### Requirement: Musical-time artifact
The project SHALL require that captured material be expressed in musical time so that the rack clock governs playback.

#### Scenario: The artifact is tempo-relative
- **WHEN** any capture path is evaluated
- **THEN** it is evaluated on the basis that the captured artifact carries events and signals positioned in musical time (bars, beats, ticks, or an equivalent tempo-relative grid), not in wall-clock time or audio samples
- **AND** the authored BPM is carried as reference metadata only, describing the grid the material was written against
- **AND** playback tempo is governed by the rack master clock, with the authored BPM used for display, validation, and fallback rather than for timing

#### Scenario: Baked audio is excluded as the primary artifact
- **WHEN** rendering VCV Rack output to audio is considered
- **THEN** the research records that a baked audio render is fixed to wall-clock time and cannot be re-clocked to the rack without varispeed or resampling
- **AND** it records this as the reason the project captures signals and sequence rather than baking audio
- **AND** it states whether any narrow exception is justified, such as a fixed-length one-shot with no tempo relationship, and on what terms

### Requirement: Capture path comparison
The project SHALL compare the candidate paths by which VCV-authored signals and sequence become a canned support track that Maybelle stores and plays back.

#### Scenario: Candidate capture paths are evaluated
- **WHEN** the capture approach is researched
- **THEN** it compares at minimum: MIDI captured out of VCV Rack, transcription of `patch.json` sequencer state, capture of a tempo-relative CV/automation event stream, and hybrids of these
- **AND** each path is evaluated for fidelity to the authored patch, determinism, artifact size, Pi playback cost, implementation complexity, ability to follow the rack clock, ability to respond to runtime rack CV, reuse of existing MIDI→CV and manifest work, and failure behavior when a plugin or asset is missing
- **AND** each path records what is lost relative to running the patch live in VCV Rack, including generative, random, and feedback behavior

#### Scenario: Continuous modulation is representable without baking audio
- **WHEN** a path's handling of continuous CV and modulation is evaluated
- **THEN** the research states how a continuous curve is represented in musical time — as breakpoints, automation segments, a per-beat resolution grid, or stepped events
- **AND** it records the resolution and interpolation the representation implies, and whether that is sufficient for the modulation being authored
- **AND** it states which curves, if any, cannot be captured this way and what the fallback is

#### Scenario: The MIDI capture path is evaluated on its own terms
- **WHEN** the possibility of getting MIDI out of VCV Rack is researched
- **THEN** the research records whether Rack can emit MIDI to a port or capture it to a file, which of note, velocity, CC, and clock survive, and what tooling the user needs
- **AND** it states how much of the existing MIDI→CV engine, manifest contract, and authoring documentation this path reuses unchanged
- **AND** it states what this path cannot carry that the rendered-audio path can

### Requirement: Transfer vehicle survey
The project SHALL confirm the ES-9 as the CV/gate vehicle by comparison rather than by default.

#### Scenario: Alternatives to the ES-9 are examined
- **WHEN** the transfer and output vehicle is researched
- **THEN** the research compares the ES-9 against other DC-coupled audio interfaces, ADAT-based Expert Sleepers combinations, and MIDI-to-CV hardware converters
- **AND** each candidate records channel count, DC coupling, voltage range, class-compliance and Linux/Pi support, licensing or driver constraints, and cost
- **AND** it states whether any alternative is better for the authoring-side capture step, for the runtime output stage, or for both
- **AND** it confirms or revises the ES-9 as the recommended vehicle, with reasons

### Requirement: Sample reference capture
The project SHALL capture the sample assets that authored material depends on, so that on-board samplers can be provisioned from the bundle.

#### Scenario: Sample references are extracted from authored material
- **WHEN** material referencing sample assets is authored in VCV Rack or another supported tool
- **THEN** the research records where sample references live in the source artifact, including assets bundled inside a `.vcv` archive and assets referenced by absolute or relative path outside it
- **AND** it records, for each referenced sample, the logical name, the source location, the file format, and enough identity to detect drift such as a content hash and size
- **AND** it records what happens when a referenced sample is missing, renamed, or moved on the authoring machine

#### Scenario: Sample references are represented in the bundle
- **WHEN** the song-bundle contract is extended to carry sample references
- **THEN** the research states whether samples travel inside the bundle, are referenced from a shared library outside it, or both
- **AND** it states how a sample reference binds to its destination on the target sampler, given that the Rossum Assimil8or is the default
- **AND** it records the provenance and licensing information that must accompany a sample, consistent with the project's music-licensing review requirement

#### Scenario: The playback boundary is kept clear
- **WHEN** sampler-bound material is described
- **THEN** the research confirms that the on-board sampler plays the sample and Maybelle does not, consistent with the Pi being neither the rack voice nor a sampler
- **AND** it states that Maybelle's role is emitting the gate/trigger and any modulation CV to the sampler through the ES-9
- **AND** it points at the separate sampler-sync work for how the sampler's own storage is provisioned

### Requirement: Canned support track definition
The project SHALL define what a canned support track is as a runtime concept.

#### Scenario: The concept is specified
- **WHEN** the research recommends a capture path
- **THEN** it defines what a canned support track contains, how it is stored in a song bundle, and how it binds to ES-9 outputs
- **AND** it states how canned support tracks relate to the voice/output mapping model already defined in `decide-song-bundle-manifest`
- **AND** it states whether the raw `.vcv` file, the captured artifact, or both are retained in the bundle, and what provenance is recorded

### Requirement: ES-9 signal path definition
The project SHALL define the signal path from VCV Rack, through the ES-9, into the Eurorack for both authoring time and runtime.

#### Scenario: The authoring-time path is defined
- **WHEN** VCV Rack on the authoring machine is connected to the ES-9
- **THEN** the research records the audio module and channel configuration used, the ES-9 device channel numbering, and which ES-9 jacks carry the resulting CV, gates, and triggers
- **AND** it confirms the DC-coupled front-panel outputs are used and the AC-coupled 1/4-inch main outputs are not
- **AND** it records the voltage range, scaling, and any calibration difference between VCV Rack's output convention and the rack's expectations

#### Scenario: The runtime path is defined
- **WHEN** the recommended capture path is described
- **THEN** the research states how the captured artifact reaches ES-9 outputs at playback time, and whether the channel map matches the one recorded for the MIDI path in `docs/authoring/`
- **AND** it confirms Maybelle retains sole ownership of the ES-9 duplex stream, including the input channels used for rack CV selection and clock

### Requirement: Clock authority resolution
The project SHALL resolve the conflict between Maybelle following the rack master clock and a VCV Rack patch owning its own engine clock.

#### Scenario: Re-clocking is analyzed per capture path
- **WHEN** each candidate capture path is evaluated
- **THEN** the research states how the captured material is re-clocked to Pamela's Pro Workout, including how start and reset are honored
- **AND** it identifies which paths produce a fixed-tempo artifact requiring varispeed or resampling, and which produce a clock-parameterized artifact Maybelle can stretch at playback
- **AND** it records the resulting timing accuracy and drift risk, separating slow modulation from pitch and gate timing
- **AND** any path that cannot follow an external clock is recorded as disqualified

### Requirement: Runtime CV selection compatibility
The project SHALL determine how rack-driven runtime selection applies to canned support tracks.

#### Scenario: Selection behavior is specified
- **WHEN** the recommended capture path is described
- **THEN** the research states how ES-9 input CV for song banks, song selection, channel banks, and transport selects and controls canned support tracks
- **AND** it confirms that quantization, debounce, hysteresis, and latching remain in Maybelle
- **AND** it states what is not controllable at runtime once material is canned, so the limitation is authored around rather than discovered later

### Requirement: Decision evidence handoff
The project SHALL produce evidence that can be consumed by the follow-on decision change.

#### Scenario: Research is ready for follow-up decisions
- **WHEN** the VCV Rack authoring-path research is complete
- **THEN** it recommends one capture path and states the v1 VCV input contract
- **AND** it lists which requirements of `decide-song-bundle-manifest` must be extended, and how
- **AND** it states which `decide-base-platform` decision points reopen and which are confirmed unaffected
- **AND** it states the impact on `research-synced-lfo-sampler-authoring`, given that sampler-baked LFOs existed to work around MIDI's inability to carry modulation
- **AND** it lists the unresolved spikes and bench tests that must complete before any parser, capture tool, or playback code is written
