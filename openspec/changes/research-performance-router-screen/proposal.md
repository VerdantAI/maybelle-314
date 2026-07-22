## Why

In Performance mode on the Pi 5-inch touchscreen, the operator needs to see, at a glance, how the authored music is patched into the rack and whether it is actually firing. The core Performance view for this is a **router screen**: a diagram of the ES-9 with each authored track shown against the output it drives, and live per-output activity as the sequence plays. This turns an invisible software mapping into a physical, glanceable patchbay picture — the operator can confirm the right tracks hit the right jacks and immediately spot a dead or wrong output during a set. It depends on the ES-9 channel map (from the ES-9 profile), the song-bundle manifest, the portrait touchscreen, and a live activity stream from the runtime.

## What Changes

- Add a spike to design the Performance-mode **router screen**: an ES-9 panel diagram, the track-to-output mapping overlaid on it, and live activity as tracks play.
- Confirm the ES-9 front-panel jack layout so the diagram is accurate (8 DC outputs, 14 DC inputs, 2 AC main outs, headphone, S/PDIF, on a 16HP panel).
- Decide the asset approach: a self-drawn, MIT-clean SVG schematic of the ES-9 panel with individually addressable jacks — not a copyrighted product photo.
- Define how track-to-output mapping is sourced (the manifest / ES-9 profile channel map) and rendered on the diagram.
- Define the live activity model: per-output activity (gate on/off, trigger pulses, CV level, stepped modulation) streamed from the runtime to the UI, decoupled so it never disturbs real-time CV/gate timing.
- Define the visual language for different signal types and the portrait-layout fit, keeping it a read/monitor surface (routing edits stay in Backstage).
- Do not implement the router screen, the SVG asset, or the activity stream in this change.

## Capabilities

### New Capabilities

- `performance-router-screen-research`: Defines requirements for the Performance-mode ES-9 router/activity screen.

### Modified Capabilities

- None.

## Impact

- Adds OpenSpec planning artifacts for the Performance router screen.
- Builds on `investigate-es9-config-profiles` (channel map), `research-daw-interchange-options` (manifest), `research-touchscreen-emulation-and-ux` (portrait UI), and `research-performance-backstage-modes` (Performance surface); informs `decide-base-platform` (a live UI-update path from the runtime).
- No production code, runtime dependencies, hardware integration, UI, or bundled third-party imagery are introduced by this proposal.
