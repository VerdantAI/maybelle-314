## ADDED Requirements

### Requirement: ES-9 diagram and asset approach
The project SHALL define an accurate, permissively-licensed ES-9 diagram for the router screen.

#### Scenario: Diagram asset is specified
- **WHEN** the router-screen research is performed
- **THEN** it confirms the ES-9 front-panel jack layout (8 DC outputs, 14 DC inputs, 2 AC main outs, headphone, S/PDIF) from official documentation
- **AND** it specifies a self-drawn, MIT-clean SVG schematic with one addressable element per jack, using no copyrighted product imagery
- **AND** it defines how the diagram scopes to the active ES-9 profile and how expanders could be added later

### Requirement: Track-to-output mapping display
The project SHALL define how the track-to-output mapping is sourced and rendered, read-only in Performance.

#### Scenario: Mapping is rendered
- **WHEN** the research defines the mapping display
- **THEN** it reads track-to-output associations from the song-bundle manifest / ES-9 profile channel map
- **AND** it renders them on the corresponding output jacks (with handling for unmapped jacks)
- **AND** it confirms the router displays mapping only, with routing edits reserved for Backstage

### Requirement: Live activity model decoupled from timing
The project SHALL define live per-output activity and its isolation from real-time output timing.

#### Scenario: Activity streaming is specified
- **WHEN** the research defines live activity
- **THEN** it defines per-output activity for gate on/off, trigger pulses, pitch/CV level, and stepped modulation
- **AND** it defines a one-way stream from the runtime to the UI sampled at a bounded UI refresh, such that the real-time CV/gate thread never blocks on or is paced by the UI
- **AND** it selects/compares a stream transport and relates it to the Performance/Backstage client-server model

### Requirement: Signal-type visual language and portrait layout
The project SHALL define the router's visual language and its portrait-panel layout.

#### Scenario: Visual language is defined
- **WHEN** the research defines the visuals
- **THEN** it specifies a distinct representation per signal type (gate, trigger, pitch/CV, stepped modulation) that never encodes state by color alone
- **AND** it defines a 720x1280 portrait layout with the ES-9 diagram as a primary Performance view plus status and manual-override controls
- **AND** it decides whether tapping a jack does anything (mapped-track detail or a guarded override) or the screen is read-only

### Requirement: Decision evidence handoff
The project SHALL produce evidence consumable by the base-platform and schema decisions.

#### Scenario: Research is ready for follow-up decisions
- **WHEN** the router-screen research is complete
- **THEN** it recommends the asset approach, mapping source, and activity-stream design
- **AND** it lists follow-up spikes before implementing the SVG, the activity stream, or the screen
- **AND** it feeds the runtime-to-UI live-update path into `decide-base-platform` and the manifest/ES-9-profile schemas
