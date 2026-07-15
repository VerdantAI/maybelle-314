## ADDED Requirements

### Requirement: Research priority bands
The project SHALL classify active research efforts into explicit priority bands.

#### Scenario: Research efforts are prioritized
- **WHEN** the research roadmap is updated
- **THEN** each active research effort is assigned a priority band
- **AND** the roadmap records whether the effort blocks the base platform decision, supports validation/tooling, or explores later product expansion

### Requirement: Web-only research flagging
The project SHALL identify which research tasks can begin with web-only work.

#### Scenario: Web-first work is visible
- **WHEN** a research effort is scheduled
- **THEN** the roadmap records whether it can begin from web searches, official documentation, public repositories, package metadata, issue trackers, or license texts
- **AND** it identifies the early questions that can be answered before hardware or local bench testing is available

### Requirement: Blocking research channels
The project SHALL record blocking research channels for each research effort.

#### Scenario: Blockers are recorded
- **WHEN** a research effort is reviewed
- **THEN** the roadmap records required channels such as official docs, source review, hardware bench testing, DAW host testing, license review, artist outreach, live show domain research, or deferred hardware access
- **AND** it distinguishes web-only progress from completion criteria that require non-web evidence

### Requirement: Initial research schedule
The project SHALL maintain an initial schedule for executing research efforts.

#### Scenario: Research work is scheduled
- **WHEN** active research efforts are coordinated
- **THEN** the roadmap defines ordered passes or phases for the research
- **AND** each pass lists the research efforts, intended outputs, and blocking channel assumptions

### Requirement: Base platform decision inputs
The project SHALL identify which research outputs are required before finalizing the base platform decision.

#### Scenario: Base platform blockers are summarized
- **WHEN** the base platform decision is prepared
- **THEN** the roadmap lists required inputs from DAW interchange, Pi port topology, ES-9 profile validation, runtime/package research, OS selection, and fixture strategy
- **AND** it identifies any unresolved blockers that would make the platform decision provisional

### Requirement: Roadmap refresh criteria
The project SHALL define when the research roadmap must be updated.

#### Scenario: Roadmap is refreshed
- **WHEN** a research effort changes priority, discovers a new blocker, completes a scheduled pass, or produces a decision input
- **THEN** the roadmap is updated to reflect the new status
- **AND** related OpenSpec changes are cross-referenced where the finding should feed another decision
