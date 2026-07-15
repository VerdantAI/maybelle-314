## ADDED Requirements

### Requirement: Live trigger source research
The project SHALL research live trigger sources that could initiate sample, patch, cue, or control-routing actions.

#### Scenario: Trigger sources are inventoried
- **WHEN** live trigger routing research is performed
- **THEN** it identifies candidate input sources including rack gates, triggers, CV, audio pulses, MIDI controllers, USB MIDI devices, ES-9 inputs, OSC messages, network cues, DAW cues, and venue show-control systems
- **AND** it records expected connection paths, electrical or protocol constraints, timing expectations, and first-performance relevance for each source

### Requirement: Routing target research
The project SHALL research possible targets for live-triggered routing actions.

#### Scenario: Routing targets are compared
- **WHEN** a live trigger target is evaluated
- **THEN** the research records whether Maybelle would emit ES-9 CV/gate/trigger outputs, send MIDI, send OSC or network messages, control external sampler hardware, control DAW/show software, select a patch, or play/route audio directly
- **AND** it records latency, reliability, configuration, and operator-feedback implications for that target

### Requirement: Show-control software surface research
The project SHALL research the live visual, sound-effect, lighting, and show-control software surface relevant to Maybelle integration.

#### Scenario: Software categories are mapped
- **WHEN** show-control software research is performed
- **THEN** it categorizes tools by role, such as audio cue playback, DAW playback, VJ/visual effects, lighting/show control, interactive media, modular control, and protocol bridging
- **AND** it records common integration protocols, including MIDI, MIDI Show Control, MIDI Time Code, OSC, network APIs, DMX-family protocols, Art-Net, sACN, LTC/SMPTE, and audio/CV trigger paths where relevant

### Requirement: Trigger qualification and safety research
The project SHALL research how live triggers should be qualified before routing actions are fired.

#### Scenario: Trigger safety behavior is specified
- **WHEN** live trigger behavior is evaluated
- **THEN** the research identifies debounce, hysteresis, edge detection, latching, arming, safe-mode, rate-limit, and rejection behaviors needed for reliable operation
- **AND** it records failure modes such as duplicate triggers, missed triggers, unstable CV selection, wrong sample selection, unavailable target devices, and disconnected outputs

### Requirement: Sample ownership decision support
The project SHALL compare whether Maybelle should play samples, trigger external samplers, or send cues to external software.

#### Scenario: Sample ownership models are evaluated
- **WHEN** sample-related live trigger workflows are researched
- **THEN** the research compares Pi-based sample playback, ES-9-routed trigger outputs, external sampler control, DAW/show-control cueing, and future Assimil8or workflows
- **AND** it recommends which models should be supported first, deferred, or excluded from Maybelle's core scope

### Requirement: Live trigger configuration implications
The project SHALL identify configuration and validation needs for live trigger routing.

#### Scenario: Configuration needs are identified
- **WHEN** a live trigger workflow is proposed
- **THEN** the research records required mapping data, input calibration, output assignment, target identity, sample or cue identity, operator labels, dry-run behavior, and validation checks
- **AND** it identifies how those needs relate to ES-9 profile validation, song bundles, channel banks, and the agent control surface
