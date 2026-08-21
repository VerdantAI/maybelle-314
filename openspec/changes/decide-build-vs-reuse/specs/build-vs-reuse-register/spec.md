## ADDED Requirements

### Requirement: Reuse-before-build principle
The project SHALL prefer existing permissively-licensed software over building its own, and SHALL place the burden of proof on building.

#### Scenario: A proposal specifies new implementation work
- **WHEN** any OpenSpec proposal specifies a capability to be built
- **THEN** it states what existing projects were considered for that capability, or states that none were found and how the search was conducted
- **AND** where an existing project was rejected, the reason is recorded — license, platform coverage, maintenance health, or an unfillable gap — rather than left implicit
- **AND** a proposal that specifies building something an existing project already does is treated as incomplete until that comparison is recorded

#### Scenario: Smallness is treated as a design goal
- **WHEN** scope is evaluated for any change
- **THEN** the project's stated goal of remaining as small as possible is applied as a criterion
- **AND** capabilities outside the irreducible core are assumed to be someone else's problem until shown otherwise

### Requirement: Build-vs-reuse register
The project SHALL maintain a register mapping each capability it needs to a candidate project and a verdict.

#### Scenario: The register records a verdict per capability
- **WHEN** a capability Maybelle needs is identified
- **THEN** the register records the capability, the candidate project or projects, the license, the platform coverage, and a verdict of reuse, reuse-with-gaps, or build
- **AND** where the verdict is reuse-with-gaps, the specific remaining gap is named
- **AND** where the verdict is build, the reason no existing project suffices is recorded

#### Scenario: Limitations are recorded alongside strengths
- **WHEN** a candidate project is entered in the register
- **THEN** its limitations are recorded with the same prominence as its capabilities, including platform gaps, unstated or incompatible licenses, and maintenance status
- **AND** a candidate is not recorded as a solution when a limitation would prevent its use in the project's actual environment

### Requirement: Dependency admission criteria
The project SHALL apply consistent criteria before admitting a dependency.

#### Scenario: A candidate is evaluated
- **WHEN** an existing project is proposed for reuse
- **THEN** it is evaluated for license compatibility with the project's MIT-compatible constraint, platform coverage including Linux and the Raspberry Pi where relevant, maintenance health, and the size of any gap it leaves
- **AND** the evaluation distinguishes a dependency that is *linked or vendored* from one that is *invoked as a separate tool* or merely *pointed at for the user to install*, since the licensing consequences differ
- **AND** the result is recorded in the register rather than only in the change that prompted the evaluation

#### Scenario: A candidate is close but not sufficient
- **WHEN** an existing project covers most but not all of a capability
- **THEN** the research states whether to wrap it, contribute upstream, or build, and why
- **AND** building is chosen only when wrapping and contributing are both shown to be worse, not merely slower

### Requirement: Irreducible core
The project SHALL name the capabilities that have no credible existing substitute.

#### Scenario: The core is stated explicitly
- **WHEN** the register is compiled
- **THEN** it names the capabilities the project must own, and states for each why no existing project fills the need
- **AND** the core is kept as small as the evidence allows
- **AND** anything not in the core is treated as a candidate for reuse rather than as planned implementation work

### Requirement: Application to existing changes
The project SHALL apply the register to changes already drafted.

#### Scenario: Drafted changes are re-scoped
- **WHEN** the register identifies an existing project overlapping a drafted change
- **THEN** that change is re-scoped to specify only what remains genuinely project-specific
- **AND** `research-sampler-sample-sync` is re-scoped against A8Manager and rclone
- **AND** `research-vcv-rack-companion-module` records the register's effect on its recommendation
