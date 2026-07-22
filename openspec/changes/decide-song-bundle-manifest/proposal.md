## Why

Multiple research spikes now converge on one artifact: the **song-bundle manifest** — the Maybelle-owned data that accompanies the exported Standard MIDI File(s) and expresses everything a DAW export cannot. The DAW-interchange research recommended a bundle of "Type-1 SMF(s) + a project-owned manifest" that owns the channel→ES-9 map, banks, calibration, and CV-selection behavior; the ES-9, touchscreen, router-screen, Performance/Backstage, and synced-LFO spikes all read or write pieces of it; and the authoring docs describe a voice model the manifest must encode. Before any import/runtime/editor code, we need to decide the v1 manifest format and its contract.

## What Changes

- Decide the **song-bundle structure**: what a bundle contains (manifest + SMF(s) + optional waveform/sample/stem assets) and how it is packaged.
- Decide the **manifest serialization**: canonical format, a published schema, and versioning.
- Define the **voice/output mapping model**: how SMF tracks/channels become voices and bind to ES-9 outputs (pitch CV, gate/trigger, velocity), consistent with the MIDI→CV model in `docs/authoring/`.
- Define the **runtime selection configuration**: CV-input-driven song-bank/song/channel-bank/transport selection with quantization, debounce, hysteresis, and latching (the README selection model).
- Define **modulation and asset references**: stepped-CV tracks, sampler-baked LFO trigger references, and bundled waveform/sample assets.
- Define the **relationship to the reusable ES-9 rig profile** (from `investigate-es9-config-profiles`): the manifest references a profile rather than duplicating rig/calibration data.
- Define **validation and licensing**: schema validation with pass/warn/fail, and a permissively-licensed format.
- Do not implement a parser, validator, importer, runtime, or editor in this change.

## Capabilities

### New Capabilities

- `song-bundle-manifest`: Defines the v1 Maybelle song-bundle structure and manifest format that binds authored MIDI to ES-9 output, banks, selection, and modulation.

### Modified Capabilities

- None.

## Impact

- Adds OpenSpec planning artifacts defining the central data contract for the project.
- Consumes `research-daw-interchange-options`, `investigate-es9-config-profiles`, `research-synced-lfo-sampler-authoring`, `research-performance-router-screen`, `research-performance-backstage-modes`, and `docs/authoring/`; informs `decide-base-platform` (bundle format is one of its decision points) and the future import/runtime/editor work.
- No production code, runtime dependencies, or hardware integration are introduced by this proposal.
