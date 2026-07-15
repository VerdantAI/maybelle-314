## Why

Maybelle 314 depends on the ES-9 routing the right USB channels, physical inputs, DC-coupled outputs, mixer paths, and hosted/standalone configuration to the correct rack patches. The Expert Sleepers ES-9 configuration tool is a simple HTML/Web MIDI SysEx app with save/load support, so we should investigate whether Maybelle can build a validator or profile maker around its configuration format before relying on manual setup.

## What Changes

- Add a research spike for ES-9 configuration profile generation and validation.
- Investigate the official ES-9 web configuration tool, saved `.syx` config dump format, hosted/standalone slots, routing selectors, input DC blocking, output DC offsets, MIDI channels, mixer state, stereo links, EQ, and smoothing.
- Define how a Maybelle patch profile could map expected ES-9 inputs/outputs to rack functions such as Pamela clock, reset, runtime CV selection, pitch CV, gates, triggers, and modulation.
- Determine whether Maybelle should validate an existing ES-9 config, generate a config for upload through the official tool, fork/wrap the HTML tool, or build an independent profile utility.
- Do not implement ES-9 config parsing, SysEx upload, firmware changes, or hardware writes in this change.

## Capabilities

### New Capabilities
- `es9-profile-validation-research`: Defines how the project researches ES-9 configuration profiles, validation rules, and profile-maker feasibility for Maybelle patch safety.

### Modified Capabilities
- None.

## Impact

- Adds OpenSpec planning artifacts for ES-9 configuration validation/profile research.
- May inform `decide-base-platform`, the ES-9 I/O spike, song-bundle output mapping, and runtime safety checks.
- No production code, ES-9 configuration changes, dependencies, or runtime behavior changes are introduced by this proposal.
