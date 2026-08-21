## ADDED Requirements

### Requirement: Scope against existing tooling
The project SHALL establish what a Maybelle module would add beyond tools that already exist.

#### Scenario: The gap is stated before the effort is estimated
- **WHEN** the companion-module research begins
- **THEN** it records what the Chinenual MIDI Recorder and other existing Rack modules already provide
- **AND** it states specifically what a Maybelle module would add that those do not, rather than restating MIDI capture
- **AND** if no material gap is found, that is recorded as a finding and the remaining research is scoped down accordingly

### Requirement: Level-of-effort assessment
The project SHALL estimate the cost of building and shipping a VCV Rack module before recommending one.

#### Scenario: Build effort is estimated
- **WHEN** the effort assessment is performed
- **THEN** it records the implementation language and API version, the SDK and toolchain required, and the scaffolding tools available
- **AND** it records the non-code obligations: panel artwork, the plugin manifest, and module state serialization
- **AND** it estimates effort for a first working module separately from effort for a releasable one

#### Scenario: Distribution effort is estimated
- **WHEN** distribution is assessed
- **THEN** it records the platforms and architectures that must be built, and whether one machine can produce them all
- **AND** it records the VCV Library submission process, what the developer supplies, what the maintainers do, and how updates are released
- **AND** it records any ethics, naming, or content guidelines the plugin must satisfy

#### Scenario: Maintenance burden is estimated
- **WHEN** ongoing cost is assessed
- **THEN** it records the API and ABI compatibility policy across major and minor Rack releases
- **AND** it records the historical migration cost between major versions as evidence rather than estimate
- **AND** it states what the project commits to if it publishes a module, including the consequence of abandoning one users depend on

### Requirement: Licensing position
The project SHALL determine whether a module can be released under an MIT-compatible license.

#### Scenario: Permitted licenses are recorded
- **WHEN** licensing is researched
- **THEN** it records what licenses VCV permits for Rack plugins and under what conditions
- **AND** it records the exact conditions attached to any exception that permits a permissive license
- **AND** it records what would forfeit that position, including incorporating Rack source and charging for the plugin
- **AND** it states whether the project's MIT-compatible constraint is satisfiable, and with what obligations

#### Scenario: The distribution boundary is re-examined
- **WHEN** the module is assessed against the boundary set in `research-vcv-rack-authoring-path`
- **THEN** the research confirms that shipping a plugin is distinct from shipping or hosting VCV Rack itself
- **AND** it states what new surface the project takes on by distributing inside a third-party ecosystem
- **AND** it records whether that is consistent with the project's stated downstream position

### Requirement: Format definition
The project SHALL define the formats a module would read and write on both sides.

#### Scenario: Maybelle-side formats are defined
- **WHEN** the export and import surfaces are specified
- **THEN** the research records what the module would write toward Maybelle: note and gate data, the manifest, bundle packaging, and sample references
- **AND** it records what the module would read when importing an existing bundle, and what it can and cannot reconstruct in a patch
- **AND** each format is tied to the contract in `decide-song-bundle-manifest` rather than invented

#### Scenario: Rack-side obligations are recorded
- **WHEN** the Rack-facing surface is specified
- **THEN** the research records the plugin manifest fields, panel asset requirements, and how module state is serialized into a patch
- **AND** it records how a module performs file input and output, and cites existing modules as precedent

### Requirement: Alternatives comparison
The project SHALL compare the module against cheaper ways of closing the same gap.

#### Scenario: Alternatives are evaluated on equal terms
- **WHEN** the recommendation is formed
- **THEN** it compares at minimum: doing nothing and authoring the manifest in Backstage; a standalone converter in the project's existing Python core; contributing to an existing third-party plugin; and building the module
- **AND** each is evaluated for effort, maintenance, licensing, user experience, and how much of the gap it actually closes
- **AND** the comparison states which alternatives are complementary rather than mutually exclusive

#### Scenario: Sequencing risk is assessed
- **WHEN** timing is considered
- **THEN** the research states the risk of building against a song-bundle manifest format that is not yet settled
- **AND** it states what would need to be true before module work should start

### Requirement: Recommendation
The project SHALL produce an actionable recommendation rather than an open comparison.

#### Scenario: A decision is recommended
- **WHEN** the research is complete
- **THEN** it recommends building, deferring, or declining the module, with reasons
- **AND** it states the conditions that would change the recommendation
- **AND** if deferral is recommended, it names what should be done instead in the meantime
- **AND** it lists what must be settled before any module implementation begins
