## ADDED Requirements

### Requirement: Raspberry Pi 5 port inventory
The project SHALL document the Raspberry Pi 5 ports and power paths relevant to Maybelle before selecting a hardware topology.

#### Scenario: Pi port inventory is recorded
- **WHEN** Pi port topology research is performed
- **THEN** it records USB-C power, USB-A host ports, GPIO header, display connectors, Ethernet/PoE+ option, PCIe option, and any service/debug ports relevant to Maybelle
- **AND** it identifies which ports are candidates for power, ES-9 data, display, storage, service access, and external controllers

### Requirement: Power topology comparison
The project SHALL compare Pi power options before selecting an enclosure or rack integration approach.

#### Scenario: Power options are evaluated
- **WHEN** power topology research is performed
- **THEN** it compares official USB-C supply, rack-powered 5V conversion, PoE+ HAT, powered USB hub dependencies, and any other credible power source
- **AND** it records current capacity, mechanical fit, grounding/noise risk, boot reliability, serviceability, and safety concerns for each option

### Requirement: ES-9 USB connection validation
The project SHALL validate how the Pi connects to the ES-9 before assuming the runtime I/O stack.

#### Scenario: ES-9 data path is tested
- **WHEN** ES-9 topology research is performed
- **THEN** it tests a Pi USB-A host port to ES-9 USB-C connection with an appropriate data cable
- **AND** it records device enumeration, audio/MIDI visibility, sample-rate support, channel count, latency observations, hotplug behavior, and whether a powered hub is required

### Requirement: External MIDI controller input paths
The project SHALL evaluate how external MIDI controllers can provide Maybelle runtime input.

#### Scenario: Controller paths are compared
- **WHEN** external MIDI controller research is performed
- **THEN** it compares direct USB MIDI to Pi, DIN MIDI through a USB interface, DIN MIDI through the ES-9 MIDI breakout, and controller CV/gate outputs through ES-9 inputs
- **AND** it records port requirements, device naming stability, latency, power requirements, configuration complexity, and fit for song/track/channel selection

### Requirement: Recommended topology handoff
The project SHALL produce a recommended baseline topology and fallback options for future implementation.

#### Scenario: Topology research is complete
- **WHEN** the Pi port topology research is complete
- **THEN** it recommends a baseline wiring and power topology for the first hardware spike
- **AND** it lists fallback topologies, unresolved hardware tests, required cables/adapters, and runtime device-discovery assumptions
