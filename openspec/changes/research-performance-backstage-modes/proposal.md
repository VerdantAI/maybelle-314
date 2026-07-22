## Why

Maybelle 314 has two distinct usage contexts that want different interfaces. **Performance** runs at the rack on the 5-inch touchscreen (720x1280 portrait) — glanceable, finger-operated, robust, real-time. **Backstage** is configuration — defining triggers, channel maps, banks, ES-9 output assignments, calibration, and CV-selection behavior — done on a computer/laptop with a standard-size interface and a keyboard. We need to decide the architecture that serves both: are they two simultaneous views of one running system, or two full modes the tool switches between? The answer shapes the base-platform UI framework, the runtime's client/server model, and how config edits are kept from disrupting a live performance.

## What Changes

- Add a research spike to decide the Performance vs Backstage architecture: two simultaneous views (client/server) versus two full modes (a switched state), versus a hybrid.
- Research precedents for edit-vs-perform separation in performance/show-control software (QLab blind/live, lighting-console blind editing, cue systems' show/edit modes) and for single-core / multiple-responsive-frontend (kiosk + remote admin) architectures.
- Define how Backstage config edits are staged/applied so they never disrupt live output, and how Performance stays fully functional if Backstage is disconnected.
- Define the relationship between the Backstage laptop surface, the Performance touchscreen, and the lightweight Bluetooth/phone surface, and how each maps to the shared song-bundle manifest.
- Recommend the architecture and the responsibilities of each surface.
- Do not implement either UI, the runtime server, or a mode/state machine in this change.

## Capabilities

### New Capabilities

- `performance-backstage-modes-research`: Defines research requirements for the Performance vs Backstage interface architecture and the safe division between live output and configuration.

### Modified Capabilities

- None.

## Impact

- Adds OpenSpec planning artifacts for the Performance/Backstage architecture decision.
- Informs `decide-base-platform` (UI framework and runtime client/server model), `research-touchscreen-emulation-and-ux` (the Performance surface), `research-bluetooth-control-channel` (the lightweight remote surface), `investigate-agent-control-surface` (shared core + thin façades), and the future song-bundle/manifest format.
- No production code, runtime dependencies, hardware integration, or UI are introduced by this proposal.
