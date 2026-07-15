## Why

Maybelle 314 may need to react to live venue or performance triggers that request sound effects, patch changes, or visual cues in real time. Before committing this to the product shape, we need to understand whether the Pi/ES-9 controller can safely map incoming signals to samples, external samplers, patches, or show-control systems, and what live visual/audio software ecosystems expect.

## What Changes

- Add a research spike for live trigger ingestion and routing from venue, performer, rack, or show-control sources.
- Investigate how incoming gates, triggers, MIDI, OSC, audio/CV signals, and network cues could select samples, patches, song sections, or external actions.
- Research software used for live sound effects, theater/show control, VJ/visual effects, lighting, and interactive performance control.
- Compare routing targets, including ES-9 outputs, Eurorack patches, external samplers, DAWs, show-control applications, visual software, and future Assimil8or workflows.
- Identify timing, latency, reliability, safety, configuration, and operator-feedback requirements for live-triggered effects.
- Do not implement live trigger routing, sample playback, or show-control integrations in this change.

## Capabilities

### New Capabilities

- `live-trigger-sample-routing-research`: Defines research requirements for mapping live input signals to samples, patches, cues, and external show-control or visual/sound-effect systems.

### Modified Capabilities

- None.

## Impact

- Adds OpenSpec planning artifacts for a live trigger/sample routing investigation.
- May inform base platform selection, ES-9 profile validation, Pi port topology, song-bundle metadata, agent control surface design, and deferred Assimil8or support.
- No production code, runtime dependencies, hardware configuration, or third-party media integrations are introduced by this proposal.
