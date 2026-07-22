## Why

The project needs a documented base-platform decision before application architecture, dependencies, deployment, and development workflow can be chosen coherently. Making this decision explicitly now reduces rework and gives later implementation changes a stable technical foundation.

## What Changes

- Introduce a base-platform decision process with explicit evaluation criteria, candidate comparison, and a recorded decision.
- Define the required decision outputs so future implementation work can reference a single source of truth.
- Require the decision to cover Raspberry Pi OS baseline, ES-9 I/O stack, runtime language/framework, persistence approach, deployment target, local development workflow, display/kiosk approach, and operational constraints.
- Require the decision to consider MIT-compatible/permissive open source packages for MIDI parsing, realtime I/O, audio/CV buffering, configuration, validation, local UI, and testing.
- Record the actual decision in `decision-record.md` (Python core + local web/kiosk UI on Raspberry Pi OS Bookworm; ES-9 as class-compliant USB audio), synthesized from the executed research spikes and the manifest decision.
- Do not implement the selected platform in this change; the ES-9 I/O timing bench remains the gating spike before the I/O layer is finalized.

## Capabilities

### New Capabilities
- `base-platform-decision`: Defines how the project evaluates, selects, and records its base platform.

### Modified Capabilities
- None.

## Impact

- Adds OpenSpec planning artifacts for the base-platform decision and supporting hardware/software spikes.
- No production code, APIs, dependencies, deployment configuration, or runtime behavior changes are introduced by this proposal.
