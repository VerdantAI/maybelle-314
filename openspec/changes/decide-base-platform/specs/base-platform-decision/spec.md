## ADDED Requirements

### Requirement: Candidate platform shortlist
The project SHALL define a shortlist of candidate base platforms before selecting one.

#### Scenario: Candidate shortlist is prepared
- **WHEN** the base-platform decision is drafted
- **THEN** it lists at least two credible candidate platforms
- **AND** each candidate includes its runtime, primary language or framework, Raspberry Pi OS baseline, ES-9 I/O approach, persistence approach, deployment target, display model, and local development model

### Requirement: Evaluation criteria
The project SHALL define evaluation criteria before recommending a base platform.

#### Scenario: Criteria are documented
- **WHEN** the base-platform decision is drafted
- **THEN** it includes criteria covering product fit, ES-9 I/O feasibility, external clock following, runtime CV parameter handling, development velocity, maintainability, deployment complexity, data needs, testing approach, operational burden, ecosystem maturity, licensing compatibility, and team familiarity

### Requirement: Candidate comparison
The project SHALL compare every shortlisted candidate against the documented evaluation criteria.

#### Scenario: Candidates are evaluated consistently
- **WHEN** the base-platform decision compares candidate platforms
- **THEN** each candidate is evaluated against the same criteria
- **AND** the comparison identifies material strengths, weaknesses, and assumptions for each candidate

### Requirement: Selected platform decision
The project SHALL record one selected base platform with explicit rationale.

#### Scenario: Platform is selected
- **WHEN** the base-platform decision is accepted
- **THEN** it identifies the selected platform
- **AND** it explains why the selected platform is preferred over the rejected candidates
- **AND** it records the assumptions that would cause the decision to be revisited

### Requirement: Implementation impact summary
The project SHALL summarize the implementation consequences of the selected base platform.

#### Scenario: Follow-up implementation is directed
- **WHEN** the base-platform decision is accepted
- **THEN** it describes the expected repository structure, dependency management approach, local development commands, test strategy, environment configuration, Raspberry Pi OS image assumptions, display/kiosk setup, ES-9 I/O setup, and deployment assumptions needed by follow-up implementation work

### Requirement: Permissive dependency review
The project SHALL review candidate open source packages for MIT-compatible or permissive licensing before adopting them as core dependencies.

#### Scenario: Package candidates are screened
- **WHEN** the base-platform decision evaluates a candidate stack
- **THEN** it lists the candidate packages needed for MIDI parsing, ES-9 I/O, CV buffering, configuration, validation, local UI, and testing
- **AND** it records each package license
- **AND** it flags any non-permissive dependency or native system dependency that requires separate review

### Requirement: Hardware spike precedence
The project SHALL resolve critical ES-9 hardware I/O feasibility before treating the GUI framework as final.

#### Scenario: I/O spike informs framework selection
- **WHEN** the base-platform decision recommends a framework
- **THEN** it references evidence from an ES-9 spike for clock input, runtime CV input, and CV/gate output
- **AND** it explains how the selected framework supports the proven I/O path
