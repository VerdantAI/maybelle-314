## ADDED Requirements

### Requirement: Ardour-on-Pi assessment
The project SHALL assess whether running Ardour or a DAW-like environment on Raspberry Pi is relevant to Maybelle's runtime or authoring workflow.

#### Scenario: Pi DAW options are researched
- **WHEN** the DAW interchange research is performed
- **THEN** it compares Ardour-on-Pi, Raspberry Pi audio distributions, and headless or export-only Ardour workflows
- **AND** it records whether each option is suitable for runtime playback, authoring/export, reference only, or out of scope

### Requirement: Ardour export artifact inventory
The project SHALL inventory the artifacts Ardour can export for Maybelle song authoring.

#### Scenario: Ardour exports are evaluated
- **WHEN** an Ardour test session is exported
- **THEN** the research records which MIDI events, track names, tempo data, markers, time signatures, automation, stems, and file metadata are preserved
- **AND** it identifies which required Maybelle runtime concepts are missing from the exported artifacts

### Requirement: Interchange format comparison
The project SHALL compare candidate interchange formats before choosing the song-bundle input contract.

#### Scenario: Formats are compared
- **WHEN** the DAW interchange decision is drafted
- **THEN** it compares Ardour session files, Standard MIDI Files, Ardour stem exports, DAWproject, major-DAW export artifacts, and a Maybelle-specific manifest bundle
- **AND** each candidate is evaluated for data coverage, implementation complexity, licensing, stability, DAW support, and fit for clock-following CV/gate playback

### Requirement: Cross-DAW abstraction assessment
The project SHALL assess whether Maybelle can use a DAW-agnostic song-bundle contract while supporting Ardour first.

#### Scenario: Major DAWs are evaluated
- **WHEN** the DAW interchange research evaluates authoring sources
- **THEN** it includes Ardour, Ableton Live, Bitwig Studio, Studio One, Logic Pro, Reaper, and any other major DAW that appears relevant
- **AND** it records which DAW artifacts can preserve notes, CC automation, pitch bend, markers, tempo maps, time signatures, track names, stems, and machine-readable project metadata
- **AND** it identifies whether each DAW can produce the normalized Maybelle bundle directly, through DAWproject, through Standard MIDI Files plus stems, or only through a custom exporter/convention

### Requirement: Runtime importer boundary
The project SHALL keep DAW-specific parsing out of the Pi runtime unless explicitly justified by research.

#### Scenario: Runtime contract is normalized
- **WHEN** the research recommends a song-bundle approach
- **THEN** it defines which fields belong in the normalized Maybelle bundle
- **AND** it distinguishes runtime bundle parsing from authoring-side DAW-specific export or conversion logic

### Requirement: Existing project and module survey
The project SHALL survey existing projects and modules that demonstrate relevant DAW, MIDI-file, CV/gate, or SD-card workflows.

#### Scenario: Precedents are documented
- **WHEN** related projects are researched
- **THEN** the findings include Pi audio projects, hardware sequencers, and Eurorack modules or controllers that import MIDI files, play SD-card projects, record CV/gate data, or bridge DAW workflows to modular systems
- **AND** each precedent records the specific workflow pattern Maybelle may adopt or avoid

### Requirement: Decision evidence handoff
The project SHALL produce evidence that can be consumed by the base-platform and song-bundle decisions.

#### Scenario: Research is ready for follow-up decisions
- **WHEN** the DAW interchange research is complete
- **THEN** it recommends which artifacts should be first-class inputs to Maybelle song bundles
- **AND** it states whether the first implementation should be Ardour-only, Ardour-first with a DAW-agnostic bundle, or multi-DAW from the start
- **AND** it lists unresolved spikes that must be completed before implementing import or conversion code
