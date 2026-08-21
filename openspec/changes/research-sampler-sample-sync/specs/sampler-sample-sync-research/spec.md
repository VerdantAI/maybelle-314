## ADDED Requirements

### Requirement: Sampler storage and sample constraint inventory
The project SHALL inventory the target sampler's storage and sample constraints before designing any sync.

#### Scenario: Assimil8or storage is inventoried
- **WHEN** the sampler sync research is performed
- **THEN** it records the Rossum Assimil8or's card type, required filesystem format, capacity limits, and directory layout
- **AND** it records how samples bind to banks, presets, and channels, and where preset data lives relative to sample files
- **AND** any detail that could not be confirmed against the module or its documentation is marked as unverified rather than assumed

#### Scenario: Sample format constraints are inventoried
- **WHEN** the accepted sample formats are researched
- **THEN** the research records supported WAV encodings, bit depths, sample rates, channel counts, and length or size limits
- **AND** it records filename constraints, including length, permitted characters, and case sensitivity
- **AND** it records the module's behavior when given a file that violates a constraint

### Requirement: Sync model
The project SHALL define what synchronization compares and how it detects drift.

#### Scenario: Drift detection is specified
- **WHEN** the sync model is defined
- **THEN** it identifies the sample assets referenced by the song bundle as the authoritative set, and the card as the target to be reconciled
- **AND** it detects drift by content identity such as a hash, plus size, rather than by filename alone
- **AND** it classifies each reference as present and matching, present but differing, missing from the card, or present on the card but unreferenced

#### Scenario: Multi-song and shared-library cards are reconciled
- **WHEN** a card serves more than one song
- **THEN** the research states how the referenced sets of several songs are combined into one target state
- **AND** it states what happens to samples already on the card that no current song references, defaulting to leaving them untouched
- **AND** it states how capacity exhaustion is reported and resolved

### Requirement: Safety model
The project SHALL treat write safety to removable media as a first-class requirement.

#### Scenario: Writes are previewed before they happen
- **WHEN** a sync is requested
- **THEN** the tool produces a dry-run diff of every intended create, overwrite, and deletion before any write occurs
- **AND** destructive operations require explicit confirmation rather than being implied by running the sync
- **AND** a blind mirror-delete of the card is never the default behavior

#### Scenario: Writes are verified and interruption is survivable
- **WHEN** a sync writes to the card
- **THEN** written files are verified against their expected content identity after writing
- **AND** the research defines the behavior when the card is removed or the tool is interrupted mid-write, such that the card is left in a diagnosable state
- **AND** the research states how the user recovers from a partial sync

#### Scenario: Source assets are never mutated
- **WHEN** any conversion, renaming, or normalization is performed
- **THEN** the original sample on the authoring machine is left unmodified
- **AND** derived artifacts are written to a separate location with their relationship to the original recorded

### Requirement: Name and slot mapping
The project SHALL define how a logical sample reference becomes a concrete file and destination on the sampler.

#### Scenario: Mapping is deterministic
- **WHEN** a bundle's sample reference is mapped to the card
- **THEN** the research defines how the logical name becomes a filename satisfying the sampler's constraints, and how that mapping is recorded so it is reproducible
- **AND** it defines how a destination bank, preset, and channel slot is chosen or read from the manifest
- **AND** it defines how filename collisions and renames are resolved without silently rebinding a song to the wrong sample

### Requirement: Conversion and validation
The project SHALL define what happens when a referenced sample does not satisfy the sampler's constraints.

#### Scenario: Non-conforming samples are handled explicitly
- **WHEN** a referenced sample violates a format, length, or naming constraint
- **THEN** the research states whether the tool rejects, warns, or converts, and on what criteria
- **AND** any conversion records what was changed, so a change in audible content is never silent
- **AND** validation results are reported as pass, warn, or fail, consistent with the manifest validation model in `decide-song-bundle-manifest`

### Requirement: Runtime boundary
The project SHALL keep sample provisioning out of the Pi runtime.

#### Scenario: The boundary is stated
- **WHEN** the tool's scope is described
- **THEN** it is defined as offline authoring-side tooling that runs on the authoring machine, not on the Pi
- **AND** the research confirms Maybelle emits gates, triggers, and modulation CV to the sampler through the ES-9 and does not read, serve, or play sample content
- **AND** it states whether the Pi has any role at all in sample provisioning, including none

### Requirement: Generalization beyond the default sampler
The project SHALL assess whether the sync model extends to samplers other than the Assimil8or.

#### Scenario: A second sampler is considered
- **WHEN** the sync model is defined
- **THEN** the research separates sampler-agnostic concerns — drift detection, safety, diffing, validation reporting — from Assimil8or-specific concerns such as layout, filename rules, and preset binding
- **AND** it states what a second sampler target would require
- **AND** it records whether generalizing now is worth the cost, or whether the Assimil8or-specific path should be built first behind a clean seam

### Requirement: Provenance and licensing
The project SHALL record provenance and licensing for sample content moved by the tool.

#### Scenario: Sample provenance travels with the reference
- **WHEN** a sample is referenced by a bundle and synced to a card
- **THEN** the research defines what provenance and licensing metadata is recorded, consistent with the project's music-licensing review requirement
- **AND** it states that the tool does not bundle or redistribute third-party sample content as part of the project
- **AND** it states how generated content, such as the LFO waveforms from `research-synced-lfo-sampler-authoring`, is distinguished from user-supplied or third-party content

### Requirement: Decision evidence handoff
The project SHALL produce evidence sufficient to specify the tool before it is built.

#### Scenario: Research is ready for implementation planning
- **WHEN** the sampler sync research is complete
- **THEN** it recommends a sync model, a safety model, and a mapping scheme
- **AND** it lists which requirements of `decide-song-bundle-manifest` must be extended to express sample references and sampler destinations
- **AND** it states how generated LFO waveforms from `research-synced-lfo-sampler-authoring` travel through the same sync path
- **AND** it lists the unresolved spikes and hardware checks that must complete before any card I/O or conversion code is written
