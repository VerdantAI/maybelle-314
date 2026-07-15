## Context

Maybelle 314 is currently shaped as a Raspberry Pi 5 MIDI/CV performance controller that follows external rack clock and emits control signals through an ES-9. A related but distinct capability is live event routing: a venue, performer, rack module, or show-control system may produce a trigger that should launch a sound effect, select a patch, fire a cue, or route a control signal to an external device.

This research must decide whether Maybelle should own any part of that live cue workflow, only expose integration points, or explicitly defer it. The answer affects base platform choice, ES-9 routing assumptions, port topology, configuration UX, and future Assimil8or or external sampler support.

The relevant live software surface includes audio cue players, theater/show-control systems, VJ and visual effects tools, lighting systems, DAWs, modular control utilities, and protocol bridges. Candidate protocols and signal types include analog triggers/gates/CV, audio pulses, MIDI note/CC/program change, MIDI Show Control, MIDI Time Code, OSC, network APIs, Art-Net, sACN, DMX, LTC/SMPTE, and application-specific cue formats.

## Goals / Non-Goals

**Goals:**
- Identify live trigger input types Maybelle could receive from the rack, performers, venue systems, or external controllers.
- Identify routing targets: ES-9 outputs, Eurorack destinations, external samplers, DAWs, visual tools, lighting/show-control tools, and future Assimil8or workflows.
- Research live sound-effect and visual/show-control software categories, protocols, and practical integration points.
- Define latency, determinism, debouncing, safety, failure-mode, and operator-feedback questions that must be answered before implementation.
- Decide whether Maybelle should route live triggers directly, emit cues to other systems, or remain focused on stored MIDI/CV song playback.

**Non-Goals:**
- Implement sample playback or live cue routing.
- Select a production show-control stack.
- Commit to Pi-based audio playback.
- Commit to Assimil8or support while that hardware is unavailable.
- Replace dedicated venue show-control, lighting, or VJ software.

## Decisions

### Treat Live Triggers as a Separate Research Track

Live trigger/sample routing SHALL be researched separately from the base song playback path.

Rationale: song playback follows a known clocked sequence model. Live triggers are asynchronous, operator-facing, and may have stricter latency and safety behavior.

Alternatives considered:
- Fold this into base platform selection: efficient, but risks overloading the first platform decision.
- Defer completely until Assimil8or work: simple, but misses relevant port/protocol decisions now.

### Research Protocols Before Products

The investigation SHALL classify tools by the protocols they accept and emit, not only by product name.

Rationale: Maybelle needs stable integration surfaces. OSC, MIDI, trigger/gate/CV, network APIs, DMX-family protocols, and timecode are more durable design inputs than any single application.

Alternatives considered:
- Pick one show-control product first: faster, but likely too narrow.
- Only support rack-level CV: simpler, but may prevent venue or visual-system integration.

### Separate Trigger Interpretation From Routing Action

The research SHALL distinguish trigger detection, event qualification, mapping, routing, and target execution.

Rationale: a noisy gate, a MIDI note, and an OSC cue can all represent "fire boom," but they require different validation and transport behavior before becoming a Maybelle event.

Alternatives considered:
- Directly patch every input to an output: low abstraction, but hard to validate and unsafe for live cueing.
- Model everything as MIDI: useful for many controllers, but incomplete for CV, OSC, show-control, and lighting protocols.

### Keep Sample Ownership Open

The research SHALL compare at least three models: Maybelle triggers an external sampler, Maybelle sends cue messages to external software, and Maybelle plays or routes audio itself.

Rationale: the project has already deferred Assimil8or and does not currently intend the Pi to be the rack voice. Live SFX may be better handled by dedicated samplers or show-control software.

Alternatives considered:
- Make the Pi a sampler: gives full control, but expands storage, timing, audio I/O, licensing, and operator UX scope.
- Only emit CV triggers: keeps Maybelle pure, but may not satisfy venue/visual workflows.

## Risks / Trade-offs

- Live triggers require lower latency than song selection -> Mitigation: measure acceptable trigger-to-output timing before implementation.
- Venue/show-control protocols expand scope quickly -> Mitigation: document protocol categories and defer product-specific integrations until a target workflow exists.
- Bad trigger mapping can fire the wrong sample or cue -> Mitigation: require validation, clear operator feedback, arm/safe states, and dry-run tooling.
- ES-9 audio/CV routing may not be enough for external show systems -> Mitigation: include USB MIDI, network, and controller port topology in research.
- Pi-based sample playback may conflict with the "controller, not voice" product shape -> Mitigation: keep sample playback as one option, not the default assumption.

## Migration Plan

No runtime migration is required because this change adds planning artifacts only. Research findings should feed into base platform selection, port topology, ES-9 profile validation, song-bundle schema, control-surface design, and any later Assimil8or investigation.

Rollback is limited to removing this OpenSpec change before implementation.

## Open Questions

- What live trigger sources are realistic for the first performance environment: rack gates/CV, MIDI controllers, OSC, venue show-control, or network cues?
- Is "fire sample" a Maybelle-owned action, an external sampler trigger, or a message to another show-control system?
- What are acceptable trigger-to-action latency and jitter limits for SFX, visual cues, and rack patch changes?
- Which live software categories and protocols are most relevant: QLab-style cue playback, DAWs, VJ tools, lighting consoles, Max/Pure Data, TouchDesigner-style systems, or modular-specific tools?
- What operator feedback is required on the 5-inch display when a live trigger is armed, received, rejected, or fired?
