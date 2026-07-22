## Why

Maybelle 314 now has several parallel research tracks, and some of them block base architecture decisions while others can run later or entirely from web research. A persisted research roadmap will keep the work ordered, identify blocker channels, and make it clear which questions should be answered before implementation begins.

## What Changes

- Add a research roadmap capability that prioritizes active OpenSpec research efforts.
- Classify each research track by dependency level, blocking channel, and whether it can be started with web-only research.
- Define an initial schedule that front-loads low-cost web research while preserving hardware-dependent investigations for bench sessions.
- Identify research outputs that must feed into `decide-base-platform` before the base platform is selected.
- Do not perform the research, select the platform, or archive existing research changes in this change.

## Capabilities

### New Capabilities

- `research-roadmap-prioritization`: Defines how Maybelle prioritizes, schedules, and tracks blocking channels across active research efforts.

### Modified Capabilities

- None.

## Impact

- Adds OpenSpec planning artifacts for research coordination.
- May guide ordering for `decide-base-platform`, `research-daw-interchange-options`, `investigate-es9-config-profiles`, `investigate-pi-port-topology`, `investigate-agent-control-surface`, `research-open-test-music-fixtures`, `research-live-trigger-sample-routing`, `research-synced-lfo-sampler-authoring`, `research-touchscreen-emulation-and-ux`, `research-bluetooth-control-channel`, and `research-performance-backstage-modes`.
- No production code, dependencies, hardware integration, or runtime behavior changes are introduced by this proposal.
