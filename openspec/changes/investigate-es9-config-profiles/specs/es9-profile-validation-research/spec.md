## ADDED Requirements

### Requirement: ES-9 configuration tool assessment
The project SHALL assess the official ES-9 configuration tool before deciding whether to build validation or profile generation on top of it.

#### Scenario: Official tool behavior is documented
- **WHEN** ES-9 profile validation research is performed
- **THEN** it records the official tool version, supported firmware version, save/load behavior, SysEx commands observed, and configurable areas relevant to Maybelle
- **AND** it records any licensing, browser, Web MIDI, SysEx, or redistribution constraints

### Requirement: Configuration dump format inventory
The project SHALL inventory the ES-9 saved configuration dump format before implementing a parser or generator.

#### Scenario: Config dump format is inspected
- **WHEN** a saved ES-9 config dump is analyzed
- **THEN** the research records the header, versioning fields, payload size, checksum or terminator behavior if present, hosted/standalone scope, and fields for routing, DC blocking, DC offsets, MIDI channels, mixer state, links, EQ, and smoothing
- **AND** it identifies which fields are required for Maybelle patch safety

### Requirement: Maybelle patch profile model
The project SHALL define a semantic Maybelle patch profile model separate from raw ES-9 configuration bytes.

#### Scenario: Patch intent is represented
- **WHEN** the profile model is drafted
- **THEN** it maps rack functions such as Pamela clock input, reset input, runtime CV selection, song bank CV, channel bank CV, pitch CV outputs, gates, triggers, and modulation outputs to expected ES-9 physical and USB channels
- **AND** it records expected signal direction, voltage range, calibration assumptions, and latch or safety behavior where relevant

### Requirement: ES-9 profile validation rules
The project SHALL define validation rules that compare a Maybelle patch profile with an ES-9 configuration.

#### Scenario: Config is validated against patch intent
- **WHEN** a user-provided ES-9 config is checked against a Maybelle patch profile
- **THEN** validation reports whether required inputs and outputs are routed to the expected channels
- **AND** it warns about DC blocking, DC offsets, stereo links, mixer routing, hosted/standalone slot mismatch, or MIDI channel settings that conflict with the patch profile

### Requirement: Profile utility recommendation
The project SHALL recommend a future implementation approach for ES-9 profile support.

#### Scenario: Research recommends an implementation path
- **WHEN** ES-9 profile validation research is complete
- **THEN** it recommends whether Maybelle should validate downloaded `.syx` files, generate `.syx` files, wrap the official HTML tool, provide manual setup checklists, or combine these approaches
- **AND** it lists unresolved hardware and license spikes required before any utility writes configuration to an ES-9
