## ADDED Requirements

### Requirement: Synced-LFO authoring model
The project SHALL define how a tempo-synced LFO is authored across the DAW, the sampler, and the Maybelle runtime.

#### Scenario: LFO shape and trigger are separated
- **WHEN** the synced-LFO authoring research is performed
- **THEN** it defines that the LFO shape is baked into a waveform stored in the sampler while the LFO trigger timing is authored as a dedicated MIDI note track
- **AND** it records that the trigger note survives Ardour and Bitwig MIDI export and is emitted by Maybelle as a gate/trigger through the ES-9 into the sampler channel's trig-in

### Requirement: Waveform format inventory
The project SHALL inventory the waveform formats used to store LFO shapes.

#### Scenario: Waveform flavors are documented
- **WHEN** waveform formats are researched
- **THEN** the findings cover single-cycle (Adventure Kid AKWF-style) and multi-bar bounce-of-automation waveforms, with their WAV specifications and playback intent (looped repeating LFO vs one-shot synced arc)
- **AND** the licensing and attribution of each candidate waveform source are recorded, and self-generated waveforms are noted as free of third-party obligations

### Requirement: Sampler sync behavior
The project SHALL document how the sampler plays a stored waveform as an LFO and stays synchronized to an external clock.

#### Scenario: Sampler LFO sync is characterized
- **WHEN** the Assimil8or (or comparable sampler) is researched
- **THEN** the findings record playback modes, gate/trigger-in retriggering, and relevant CV mappings (Pitch, Sample Start, Loop Start/Length)
- **AND** they describe how loop length plus clock division and re-triggering keep the waveform phase-locked to Pamela's clock, including tempo-following via Pitch CV

### Requirement: Configurator tooling boundary and licensing
The project SHALL determine which existing sampler configurator tooling can be reused and which must be reimplemented, consistent with the project's permissive-licensing requirement.

#### Scenario: Configurator licensing is resolved
- **WHEN** existing Assimil8or configurators are evaluated
- **THEN** the research records each tool's license status and confirms which tools are all-rights-reserved and therefore cannot be vendored or forked
- **AND** it recommends pointing to (and optionally launching) an external tool such as A8Manager with proper credit, while defining a clean-room, Maybelle-owned preset writer for generated content that reuses no reserved source

### Requirement: Trigger and bundle integration evidence
The project SHALL produce evidence for how synced-LFO authoring integrates with the song bundle and ES-9 output allocation.

#### Scenario: Integration evidence is ready for follow-up decisions
- **WHEN** the synced-LFO authoring research is complete
- **THEN** it identifies the song-bundle metadata fields needed to associate a trigger track, a stored waveform, and a sampler channel/output
- **AND** it states how ES-9 outputs are allocated to LFO trigger/gate duty alongside the main sequence's pitch, gate, and velocity outputs
- **AND** it lists unresolved spikes that must be completed before implementing waveform generation, preset writing, or trigger emission
