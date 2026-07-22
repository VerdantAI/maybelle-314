## ADDED Requirements

### Requirement: Target panel confirmation
The project SHALL confirm the target touchscreen panel and its Pi display/touch stack.

#### Scenario: Panel characteristics are recorded
- **WHEN** the touchscreen research is performed
- **THEN** it records the panel interface (DSI/HDMI), resolution, touch controller, orientation, and driver requirements on the Pi 5
- **AND** it records the display/touch stack relevant to rendering and touch input

### Requirement: Emulation feasibility assessment
The project SHALL assess whether the display and touch can be emulated for development without the physical panel.

#### Scenario: Emulation options are evaluated
- **WHEN** the research evaluates emulation
- **THEN** it evaluates web/kiosk emulation at 800x480 with browser touch emulation, a fixed-size desktop-window stand-in, and full-system Pi 5 emulation via QEMU
- **AND** it records which options are feasible, noting that QEMU has no Pi 5 machine type and does not model GPU or DSI/USB touch
- **AND** it recommends a development emulation approach sufficient to build the UI without the panel

### Requirement: Hardware validation boundary
The project SHALL identify what must be validated on real hardware despite emulation.

#### Scenario: Hardware-only properties are listed
- **WHEN** the research defines the emulation boundary
- **THEN** it lists properties that require the physical panel, including contrast under venue lighting, capacitive touch accuracy, refresh/latency, orientation, and finger ergonomics
- **AND** it defines a minimal on-hardware validation checklist to run before finalizing UI layouts

### Requirement: Initial UI/UX direction
The project SHALL produce an initial UI/UX direction for the 800x480 performance touchscreen.

#### Scenario: UI/UX direction is defined
- **WHEN** the research addresses UI/UX
- **THEN** it defines 800x480 landscape layout constraints, touch-target sizing, type scale, and contrast/dark-theme for low light
- **AND** it identifies the core glanceable status and any manual-override interactions
- **AND** it defines the division of responsibilities between the touchscreen, CV-driven selection, and the Bluetooth/phone control surface

### Requirement: Decision evidence handoff
The project SHALL produce evidence consumable by the base-platform and related decisions.

#### Scenario: Research is ready for follow-up decisions
- **WHEN** the touchscreen emulation and UI/UX research is complete
- **THEN** it recommends the emulation approach and how emulability should weigh in the UI-framework choice
- **AND** it lists follow-up spikes required before implementing the UI or an emulation harness
- **AND** it cross-references `decide-base-platform` and `research-bluetooth-control-channel`
