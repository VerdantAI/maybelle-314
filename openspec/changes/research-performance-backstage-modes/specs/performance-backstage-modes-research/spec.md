## ADDED Requirements

### Requirement: Architecture decision
The project SHALL decide whether Performance and Backstage are two simultaneous views, two switched modes, or a hybrid, grounded in precedent.

#### Scenario: Architecture is decided
- **WHEN** the Performance/Backstage research is performed
- **THEN** it compares two-simultaneous-views (client/server), two-full-modes, and hybrid options
- **AND** it recommends an architecture with rationale drawn from performance/show-control and kiosk/multi-frontend precedents
- **AND** it records whether Performance and Backstage can run simultaneously and the primary Backstage workflow (pre-show versus live-connected)

### Requirement: Live-output safety gate
The project SHALL define how configuration edits are prevented from disrupting a live performance.

#### Scenario: Safe editing is specified
- **WHEN** the research addresses editing during a performance
- **THEN** it defines a safety-mode gate in which Backstage edits are staged and applied transactionally rather than mutating live output mid-cue
- **AND** it specifies that the Performance surface is a locked, minimal, safe surface

### Requirement: Runtime independence from optional clients
The project SHALL require that Performance and the runtime function without any Backstage or phone client connected.

#### Scenario: Standalone operation is guaranteed
- **WHEN** no Backstage or phone client is connected (including with networking off)
- **THEN** the research specifies that the Performance surface and runtime remain fully functional
- **AND** it treats Backstage and the phone surface as optional clients, not dependencies

### Requirement: Surface responsibilities and shared model
The project SHALL define each surface's responsibilities and their mapping to the shared song-bundle manifest.

#### Scenario: Surfaces form one coherent model
- **WHEN** the research defines the surfaces
- **THEN** it assigns responsibilities to Performance (touchscreen status/override/panic), Backstage (full manifest editor plus live monitoring), and the Bluetooth/phone subset
- **AND** it maps all three to a single shared song-bundle manifest model and identifies shared versus surface-specific logic
- **AND** it reconciles with `research-touchscreen-emulation-and-ux` and `research-bluetooth-control-channel`

### Requirement: In-app authoring help and diagrams
Backstage SHALL provide helpful, easy-to-access diagrams and guidance for common authoring/setup questions, sourced from the maintained authoring docs.

#### Scenario: Setup help is available in Backstage
- **WHEN** a user is configuring in Backstage
- **THEN** Backstage surfaces accessible diagrams/guidance for common questions (the MIDI→CV model, the ES-9 output map, and how to set up tracks in Bitwig/Ardour)
- **AND** the content is sourced from the maintained authoring docs (`docs/authoring/`) so the in-app help and the documentation share one source of truth

### Requirement: Decision evidence handoff
The project SHALL produce evidence consumable by the base-platform and manifest decisions.

#### Scenario: Research is ready for follow-up decisions
- **WHEN** the Performance/Backstage research is complete
- **THEN** it recommends the architecture and client/server model for `decide-base-platform`
- **AND** it lists follow-up spikes before implementing the runtime server, the Backstage editor, or the mode state machine
- **AND** it feeds the future song-bundle/manifest-format proposal
