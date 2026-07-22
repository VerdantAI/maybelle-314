## ADDED Requirements

### Requirement: Song-bundle structure
The project SHALL define the structure of a Maybelle song bundle.

#### Scenario: Bundle layout is defined
- **WHEN** the manifest format is decided
- **THEN** a bundle is a directory (optionally zipped for transport) containing a manifest, one or more Standard MIDI Files, and optional asset files
- **AND** asset kinds (waveforms, samples, optional stems) and path resolution/sandboxing are defined
- **AND** binary/media assets are kept out of the manifest itself

### Requirement: Manifest serialization and versioning
The project SHALL define the manifest serialization, schema, and versioning.

#### Scenario: Serialization is decided
- **WHEN** the manifest format is decided
- **THEN** the canonical serialization is JSON with a published JSON Schema 2020-12
- **AND** a top-level SemVer `schema_version` governs compatibility (unknown major rejected; unknown minor/patch tolerated additively)
- **AND** the format uses only permissively-licensed tooling and authored (not implicitly generated) timestamps

### Requirement: Voice and output mapping
The project SHALL define how authored MIDI binds to ES-9 outputs.

#### Scenario: The MIDI→CV voice model is encoded
- **WHEN** the manifest defines playback mapping
- **THEN** each voice binds an SMF track or MIDI channel to ES-9 output roles for pitch CV, gate or trigger, and optional velocity, consistent with `docs/authoring/midi-to-cv-model.md`
- **AND** percussion is expressed as a note→output-role trigger map
- **AND** the voice-identity selector (track vs channel) and its default are defined

### Requirement: Runtime selection and clock configuration
The project SHALL define the CV-driven selection and clock configuration.

#### Scenario: Selection and clock are configured
- **WHEN** the manifest defines runtime control
- **THEN** each selection control (song bank, song, channel bank, transport) is configured with a CV input, number of choices, debounce, hysteresis, and latching per the README selection model
- **AND** the clock/transport configuration defines external clock/start/reset inputs and that the file grid is re-clocked to the rack

### Requirement: Modulation, assets, and profile linkage
The project SHALL define modulation references, bundled assets, and the ES-9 profile link.

#### Scenario: Modulation and rig linkage are defined
- **WHEN** the manifest defines modulation and rig binding
- **THEN** modulation is expressed as references (stepped-CV tracks; sampler-LFO trigger + waveform + target), never as DAW automation
- **AND** the manifest references a reusable ES-9 rig profile by role ids rather than hard-coding physical jacks, with any embedded profile snapshot's authority defined
- **AND** bundled asset entries link to their modulation/sampler references

### Requirement: Validation and decision handoff
The project SHALL define validation and feed dependent decisions.

#### Scenario: Validation and handoff are defined
- **WHEN** the manifest format is complete
- **THEN** it defines schema validation with pass/warn/fail semantics
- **AND** it feeds the bundle-format decision into `decide-base-platform` and lists follow-up spikes (importer, validator, editor) before implementation
- **AND** it marks runtime-dependent fields provisional pending `decide-base-platform` and the hardware bench
