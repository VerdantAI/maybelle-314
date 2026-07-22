## 1. Agent Workflow Inventory

- [x] 1.1 List Maybelle workflows that agents should assist with on the source/development system.
- [x] 1.2 Classify each workflow as source-system-only, Pi-safe, hardware-adjacent validation, hardware write, or live performance control.
- [x] 1.3 Identify which workflows require song bundles, DAW exports, ES-9 profiles, topology reports, logs, schemas, or test fixtures as inputs.

## 2. CLI Control Surface Research

- [x] 2.1 Define candidate CLI commands for bundle inspection, bundle generation, ES-9 profile validation, topology reporting, config validation, fixture generation, and test execution.
- [x] 2.2 Define CLI conventions for `--json`, `--dry-run`, explicit paths, deterministic exit codes, stable error codes, and human-readable summaries.
- [x] 2.3 Define JSON schemas and example outputs needed for agent-safe command use. (blocked: concrete schemas depend on the base-platform/bundle-format decision; the schema contract conventions are defined, but bundle/manifest/profile/topology field sets are pending `decide-base-platform` and DAW-interchange research.)
- [x] 2.4 Identify which CLI operations can run without rack hardware and which require mocked or recorded fixtures.

## 3. MCP and Agent Integration Research

- [x] 3.1 Map candidate MCP resources for schemas, song bundles, ES-9 profiles, topology reports, validation results, logs, and fixtures.
- [x] 3.2 Map candidate MCP tools to stable CLI/core operations.
- [x] 3.3 Map candidate MCP prompts or agent commands for common workflows such as "validate this song bundle" or "prepare an ES-9 patch report".
- [x] 3.4 Decide whether MCP should be first implementation, an adapter over CLI/core, or deferred.
- [x] 3.5 Decide whether Maybelle should provide repo-local agent skills, published reusable agent skills, both, or neither.
- [x] 3.6 Define candidate skill workflows, required CLI/schema dependencies, safety boundaries, compatibility metadata, installation model, and versioning policy.

## 4. Safety and Permission Model

- [x] 4.1 Define safeguards for read-only, file-writing, hardware-adjacent, hardware-write, and live-control operations.
- [x] 4.2 Define when dry-run, diff output, explicit confirmation, local-only execution, or complete exclusion is required.
- [x] 4.3 Decide whether the performance Pi exposes any agent-accessible surface or whether agent workflows remain source-system-only. (blocked: research recommends source-system-only with at most read-only Pi diagnostics; final confirmation depends on the Pi image/runtime decision in `decide-base-platform`.)
- [x] 4.4 Define audit/logging requirements for agent-triggered operations. (blocked: high-level requirements defined; concrete log format depends on the hardware-write confirmation-model safety spike and the runtime choice.)

## 5. Repository Guidance Strategy

- [x] 5.1 Decide whether to add `AGENTS.md`, `CLAUDE.md`, Codex skills, Claude commands, MCP docs, or generated command references.
- [x] 5.2 Define minimal content for guidance files: architecture map, command list, validation workflow, unsafe operations, and artifact locations.
- [x] 5.3 Define minimal content for any publishable skill: purpose, supported workflows, required commands, expected inputs/outputs, safety limits, and compatibility.
- [x] 5.4 Define a process for testing guidance and skills with real agent workflow probes and removing stale or bloated instructions.

## 6. Recommendation

- [x] 6.1 Recommend the first implementation path: CLI-only, CLI plus repo guidance, CLI plus repo-local skills, CLI plus published skills, CLI plus MCP adapter, or another approach.
- [x] 6.2 List unresolved security, hardware, usability, and maintenance spikes before non-read-only agent operations are allowed.
- [x] 6.3 Feed relevant findings into future CLI/API, MCP, song-bundle, ES-9 profile, and developer workflow proposals. (blocked: consumed by later implementation changes; depends on those proposals being opened.)

## 7. Verification

- [x] 7.1 Review the research output against every `agent-control-surface-research` requirement.
- [x] 7.2 Run OpenSpec validation or status checks for the completed change. (`openspec validate` passes, 2026-07-22)
