## ADDED Requirements

### Requirement: Agent workflow inventory
The project SHALL inventory Maybelle workflows that could benefit from AI-agent assistance before designing an agent control surface.

#### Scenario: Agent workflows are identified
- **WHEN** agent control-surface research is performed
- **THEN** it lists candidate workflows for DAW export inspection, song-bundle generation, ES-9 profile validation, Pi topology reporting, configuration review, fixture generation, and test execution
- **AND** it classifies each workflow as source-system-only, Pi-safe, or hardware-affecting

### Requirement: Structured CLI assessment
The project SHALL evaluate a structured CLI as the baseline agent control surface.

#### Scenario: CLI conventions are defined
- **WHEN** the agent control surface is drafted
- **THEN** it defines conventions for `--json`, `--dry-run`, explicit paths, deterministic exit codes, schema validation, stable error codes, and human-readable summaries
- **AND** it identifies which commands agents can run safely without direct hardware access

### Requirement: MCP feasibility assessment
The project SHALL evaluate whether an MCP server would improve agent access to Maybelle workflows.

#### Scenario: MCP resources and tools are mapped
- **WHEN** MCP feasibility is researched
- **THEN** it maps candidate MCP resources, tools, and prompts to existing or proposed CLI/core operations
- **AND** it records which MCP capabilities are useful only after schemas and CLI commands are stable

### Requirement: Safety boundary model
The project SHALL define safety boundaries for agent-accessible operations.

#### Scenario: Agent operation risk is classified
- **WHEN** an operation is proposed for agent access
- **THEN** it is classified as read-only, reversible file-writing, hardware-adjacent validation, hardware write, or live performance control
- **AND** the research specifies required safeguards such as dry-run defaults, diff output, explicit confirmation, local-only execution, or exclusion from agent tooling

### Requirement: Repository guidance strategy
The project SHALL define repository guidance for agents that is concise, operational, and testable.

#### Scenario: Guidance files are planned
- **WHEN** agent guidance is proposed
- **THEN** it identifies which files or skill/command definitions should exist for Codex, Claude, and other agents
- **AND** it states what each guidance artifact should contain, how it should avoid context bloat, and how it should be validated against real agent workflows

### Requirement: Published skill assessment
The project SHALL decide whether Maybelle should publish reusable agent skills for Codex, Claude, or other compatible agents.

#### Scenario: Skill publication is evaluated
- **WHEN** agent skill support is researched
- **THEN** it compares repo-local skills, published reusable skills, MCP prompts, and ordinary documentation
- **AND** it identifies which workflows are appropriate for a skill, which commands or schemas they depend on, and which hardware-affecting operations must be excluded or gated
- **AND** it records versioning, compatibility, installation, testing, and maintenance requirements for any published skill

### Requirement: Agent-readiness recommendation
The project SHALL recommend a first implementation path for agent support.

#### Scenario: Research produces a recommendation
- **WHEN** agent control-surface research is complete
- **THEN** it recommends whether the first implementation should be CLI-only, CLI plus repo guidance, CLI plus repo-local skills, CLI plus published skills, CLI plus MCP adapter, or another approach
- **AND** it lists unresolved security, hardware, and usability spikes before agents can perform non-read-only operations
