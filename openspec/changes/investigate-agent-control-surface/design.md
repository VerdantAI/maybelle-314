## Context

Maybelle 314 will have multiple workflows that agents can help with on the source/development system: DAW export inspection, song-bundle creation, ES-9 profile validation, Pi port-topology reports, configuration review, and test fixture generation. These workflows are structured, file-oriented, and validation-heavy, which makes them good candidates for Codex, Claude, and other coding agents.

The performance Pi is different. It may be connected to a live rack and should not expose broad write/control surfaces to autonomous agents. Agent support should therefore be designed primarily for the source/development machine, with explicit boundaries around anything that can alter runtime state, write hardware config, or arm playback.

Relevant integration patterns:
- MCP provides a standard way for AI applications to access external systems using resources, tools, prompts, JSON-RPC, and local stdio or remote HTTP transports. MCP documentation explicitly frames tools as executable functions, resources as context data, and prompts as reusable workflows.
- Agent-friendly command surfaces need deterministic behavior, schemas, structured output, dry-run modes, clear errors, and test fixtures.
- Repository guidance files such as `AGENTS.md`, `CLAUDE.md`, and Codex/Claude skill or command files can help agents when kept small, concrete, and free of conflicting instructions.
- Reusable agent skills can package Maybelle-specific workflows for use outside this repository, but publishing a skill creates maintenance, versioning, documentation, and safety obligations beyond local repo guidance.

## Goals / Non-Goals

**Goals:**
- Define the best control surface for agents operating on the Maybelle source/development system.
- Identify CLI commands, JSON schemas, resources, prompts, and validation workflows that make agent work reliable.
- Decide whether Maybelle should expose an MCP server, repo guidance files, local agent skills, published reusable agent skills, or only a conventional CLI at first.
- Define safety boundaries for read-only inspection, reversible file generation, hardware-affecting operations, and performance runtime control.
- Make all agent-facing operations inspectable by humans through logs, dry-runs, diffs, and deterministic outputs.

**Non-Goals:**
- Implement an MCP server, CLI, or agent skill.
- Give agents direct unattended control over a live Pi/rack performance system.
- Support cloud access to private song bundles or hardware state by default.
- Optimize for one agent vendor at the cost of a normal command-line workflow.
- Publish an agent skill before the underlying CLI schemas and safety model are stable.

## Decisions

### Make the CLI the Primary Control Surface

The research will treat a normal command-line interface as the primary agent integration point. Commands should support `--json`, `--dry-run`, explicit input/output paths, deterministic exit codes, and stable schemas.

Rationale: all major coding agents can run local commands, inspect files, and reason about structured output. A CLI also remains useful to humans and CI without any agent-specific runtime.

Alternatives considered:
- MCP first: attractive for richer agent UX, but premature before the underlying operations are stable.
- GUI automation: brittle and hard for agents to inspect.
- Natural-language-only workflows: convenient, but hard to validate or reproduce.

### Treat MCP as an Adapter Over Stable Core Commands

If MCP is adopted, it should wrap stable internal services or CLI operations rather than becoming the only way to perform Maybelle tasks.

Rationale: MCP can expose resources, tools, and prompts to Codex, Claude, and other clients, but the underlying behavior should remain testable outside MCP. This keeps Maybelle usable by agents, humans, and CI.

Alternatives considered:
- Build all automation directly as MCP tools: good agent ergonomics, but weak standalone operability.
- Avoid MCP entirely: simpler, but may miss a portable integration surface supported by many agent clients.

### Separate Read-Only, File-Writing, and Hardware-Affecting Actions

Agent-facing operations will be classified by risk. Read-only inspection and validation can be broadly available. File generation should default to dry-run/diff. Hardware-affecting operations, such as uploading ES-9 config or controlling runtime playback, must require explicit human confirmation and may be excluded from agent surfaces.

Rationale: Maybelle interacts with real hardware and rack patches. The cost of an incorrect write is higher than a normal source-code edit.

Alternatives considered:
- One uniform permission model: simpler, but too coarse for hardware workflows.
- No hardware-adjacent agent tools: safest, but prevents useful validation and setup assistance.

### Keep Repository Guidance Minimal and Operational

The research will define guidance files that tell agents where schemas live, how to run validation, how to generate fixtures, and which operations are unsafe.

Rationale: agent guidance is useful when it is short, concrete, and tied to commands. Long or vague guidance can increase exploration and create conflicts.

Alternatives considered:
- No guidance files: leaves agents to rediscover workflows.
- Large comprehensive manuals: likely to bloat context and go stale.

### Evaluate Published Skills After Workflow Stabilization

The research will decide whether Maybelle should publish reusable agent skills, but the default path is to publish only after CLI commands, schemas, fixtures, and safety boundaries are stable.

Rationale: a published skill is useful when it packages repeatable domain workflows, such as validating song bundles or preparing ES-9 patch reports, for agents working outside the main repository. Publishing too early risks distributing stale instructions, unsafe hardware assumptions, or commands that change frequently.

Alternatives considered:
- Publish a skill immediately: increases discoverability, but creates maintenance and safety risk before workflows are proven.
- Never publish skills: simplest, but misses a useful path for source-system setups, companion tools, or other repositories that need Maybelle-aware workflows.

## Risks / Trade-offs

- Agent executes unsafe hardware action -> Mitigation: classify actions, require dry-run by default, and gate hardware writes behind explicit confirmation or exclude them.
- Agent cannot infer workflow from CLI output -> Mitigation: provide JSON schemas, examples, stable errors, and machine-readable status reports.
- MCP adds maintenance burden -> Mitigation: build CLI/core first and use MCP only as an adapter.
- Repo guidance becomes stale or contradictory -> Mitigation: keep guidance short and test it through agent workflow probes.
- Published skill becomes stale or unsafe -> Mitigation: require versioned workflows, explicit compatibility metadata, safety disclaimers, and probe tests before publishing.
- Runtime Pi is treated like a development host -> Mitigation: document source-system-only workflows and explicitly disable unsafe runtime operations.

## Migration Plan

No runtime migration is required because this change adds planning artifacts only. Research output should feed into future CLI/API design, MCP support, schema design, repo guidance, and developer workflow proposals.

Rollback is limited to removing this OpenSpec change before implementation.

## Open Questions

- Which operations should be available as CLI commands before any MCP server exists?
- Which Maybelle resources would be useful to expose to agents: schemas, song bundles, ES-9 profiles, topology reports, validation results, logs, or fixtures?
- Should agent support live in the main app package, a separate developer tool package, or a separate MCP server?
- What is the minimum guidance file set: `AGENTS.md`, `CLAUDE.md`, Codex skills, Claude commands, or generated docs?
- Should Maybelle publish reusable agent skills, keep skills repo-local, or avoid skills until the CLI and schemas stabilize?
- If a skill is published, what package name, versioning policy, compatibility matrix, installation path, and safety boundaries should it declare?
- Should the performance Pi expose any agent-accessible surface, or should agent workflows remain source-system-only?
