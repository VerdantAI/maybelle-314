## ADDED Requirements

### Requirement: Bluetooth stack assessment
The project SHALL assess the Raspberry Pi 5 Bluetooth/BLE stack for a control channel.

#### Scenario: Bluetooth capabilities are researched
- **WHEN** the Bluetooth control-channel research is performed
- **THEN** it records the Pi 5 radio and BlueZ capabilities for a GATT server/peripheral and advertising
- **AND** it compares BLE (GATT) against classic Bluetooth BR/EDR + SPP, including iOS/Android client constraints
- **AND** it notes the runtime dependency and service/packaging implications for the base platform

### Requirement: Event triggering scope
The project SHALL determine which events Bluetooth can trigger and how they relate to clock-tight control.

#### Scenario: Trigger events are classified
- **WHEN** the research evaluates event triggering
- **THEN** it describes how a central writes GATT characteristics to trigger events and how the Pi notifies state back
- **AND** it classifies each candidate event (bank select, transport, load-song, cues) as Bluetooth-appropriate or reserved for clock-tight CV/MIDI
- **AND** it records BLE latency/reliability characteristics that justify the classification

### Requirement: Update participation assessment
The project SHALL determine whether and how Bluetooth participates in software and content updates.

#### Scenario: Update transport is evaluated
- **WHEN** the research evaluates updates over Bluetooth
- **THEN** it records BLE throughput limits and estimated transfer times for song-bundle-sized versus full-image payloads
- **AND** it compares direct-over-BLE transfer with a BLE-authorized, Wi-Fi/USB-delivered update model
- **AND** it defines safe-apply requirements including integrity checks, atomic apply, and rollback to avoid bricking

### Requirement: Simple authentication recommendation
The project SHALL recommend a simple operator-friendly authentication scheme and record its limitations.

#### Scenario: Authentication is defined
- **WHEN** the research evaluates authentication
- **THEN** it describes BLE pairing/bonding with a passkey (PIN), including a display-shown passkey flow
- **AND** it records known passkey-pairing weaknesses and the unauthenticated "Just Works" case
- **AND** it recommends a layered approach that combines bonding with an application-layer PIN/shared-secret gate, plus credential provisioning, rotation, and revocation

### Requirement: Security and safety containment
The project SHALL define how the Bluetooth channel is contained so it does not compromise runtime timing or safety.

#### Scenario: Containment is specified
- **WHEN** the research assesses the security surface
- **THEN** it defines a bounded command set with no remote shell or broad administration
- **AND** it defines fail-safe behavior when the channel is idle or a device disconnects
- **AND** it states that runtime timing and safe operation do not depend on the Bluetooth channel

### Requirement: Decision evidence handoff
The project SHALL produce evidence consumable by dependent decisions.

#### Scenario: Research is ready for follow-up decisions
- **WHEN** the Bluetooth control-channel research is complete
- **THEN** it recommends the Bluetooth role, update participation, and authentication scheme
- **AND** it lists follow-up spikes required before implementing a GATT server, pairing flow, or update mechanism
- **AND** it cross-references `decide-base-platform`, `investigate-agent-control-surface`, and `research-live-trigger-sample-routing`
