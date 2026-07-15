## Why

Maybelle 314 will need configuration, validation, bundle generation, ES-9 profile checks, and hardware-topology diagnostics that are well suited to AI-assisted workflows on the source/development system. To make Codex, Claude, and similar agents effective without giving them unsafe direct control over performance hardware, the project needs an intentional agent control surface with structured commands, machine-readable outputs, clear safety boundaries, and repo guidance.

## What Changes

- Add a research spike for an AI-agent-friendly control surface for Maybelle.
- Investigate best practices for local CLI commands, structured JSON output, schemas, dry-run modes, validation commands, logs, state snapshots, and deterministic test fixtures.
- Investigate whether Maybelle should expose an MCP server, publish reusable agent skills/prompts, provide repository guidance files, or use a combination of these.
- Decide whether any Maybelle agent skill should be published for reuse outside this repository, and what workflows, safeguards, versioning, and installation model it would require.
- Define which operations agents may perform on the source/development system, which require human confirmation, and which must never be available on the performance Pi.
- Identify how agents can inspect song bundles, DAW exports, ES-9 profiles, hardware topology reports, and runtime configuration without needing direct rack access.
- Do not implement MCP servers, CLI commands, agent skills, or runtime automation in this change.

## Capabilities

### New Capabilities
- `agent-control-surface-research`: Defines how the project researches and records an AI-agent-friendly control surface for configuring, validating, and operating Maybelle development workflows.

### Modified Capabilities
- None.

## Impact

- Adds OpenSpec planning artifacts for agent integration research.
- May inform future CLI/API design, MCP support, published agent skills, repo guidance, validation tools, configuration schemas, and source-system workflows.
- No production code, runtime behavior, hardware control, dependencies, or agent integrations are introduced by this proposal.
