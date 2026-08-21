## Context

Maybelle 314 is a Raspberry Pi 5 stored-sequence controller that follows the rack master clock (Pamela's Pro Workout), reads rack CV for runtime selection, and emits pitch CV, gates, triggers, and modulation through an Expert Sleepers ES-9. The Pi is not the rack voice and is not a sampler.

The prior authoring contract was DAW-shaped: author monophonic note tracks in Bitwig or Ardour, export a Type-1 Standard MIDI File, and let Maybelle act as the MIDI→CV converter. `docs/authoring/` states this as its headline rule — *"Author MIDI, not CV."* That model exists because Bitwig's `File > Export MIDI` carries notes and velocity only; automation, CC, and note expressions are dropped, which is why `research-synced-lfo-sampler-authoring` had to route modulation around MIDI entirely by baking LFO shapes into sampler waveforms.

**VCV Rack now joins as a first-class source and the primary test bench.** Bitwig remains the default file editor, Ardour is retained, and other tools are supported where practical. This is an addition, not a replacement — and it is a useful one precisely because VCV Rack is the *hardest* producer to normalize. `research-daw-interchange-options` already concluded Maybelle should consume a source-agnostic bundle rather than any tool's native format; VCV Rack is the case that proves or breaks that conclusion.

**The core difficulty: a `.vcv` patch is a program, not a recording.** An SMF is a passive list of timed events any scheduler can replay. A VCV patch is a running graph of modules with internal state, sample-rate-dependent DSP, third-party plugin dependencies, and possibly randomness and feedback.

Three constraints, now settled, cut the option space down sharply:

1. **Maybelle never ships or hosts VCV Rack.** Users install and run Rack themselves; Maybelle sits strictly downstream. This removes the GPLv3/commercial licensing hazard entirely and takes "run a Rack engine on the Pi" off the table as something the project delivers. It also removes the ES-9 device-ownership conflict — Maybelle keeps sole ownership of its duplex stream, including the input channels for rack CV and clock.
2. **Capture signals and sequence, not baked audio.** The project does not intend to render VCV Rack (or any source) down to audio.
3. **Pamela's Pro Workout is the clock.** The authored BPM travels as reference metadata only.

Constraints 2 and 3 are the same constraint viewed twice. A baked audio render is fixed to wall-clock samples; re-clocking it to an external tempo needs varispeed or resampling, with real pitch and timing consequences. **Therefore the captured artifact must be positioned in musical time — bars, beats, ticks, or an equivalent tempo-relative grid.** That single requirement is the strongest filter in this spike, and it eliminates the rendered-audio path before evaluation rather than after.

What remains genuinely open is how *continuous* modulation survives that filter. Notes and gates are natively event-shaped and re-clock cleanly. A slowly evolving CV curve authored in VCV Rack is not event-shaped, and capturing it in musical time means choosing a representation — breakpoints, automation segments, a per-beat resolution grid, or stepped events — each with a resolution and interpolation cost. This is the same problem the MIDI path hit from the other direction, and it is where the spike should spend its effort.

Initial understanding, all of which the spike must confirm by inspection rather than assume:

- A Rack 2 `.vcv` patch is understood to be a compressed archive containing a `patch.json` plus per-module asset directories, where each module is identified by plugin slug, module slug, and version, and carries its own serialized state. Rack 1 patches were plain JSON. **Unverified — task 1.1.**
- The sequencers currently in use are **Impromptu Modular** (PhraseSeq16, GateSeq64, Foundry), a third-party plugin. Per-step data — CV, gate types, probability, ties — lives inside that plugin's own serialization, not in any Rack-level schema.
- VCV Rack is understood to have `CV-MIDI`-style modules capable of emitting MIDI to a port. Whether MIDI can be captured to a file, and which of note, velocity, CC, and clock survive, is **unverified — task 3.2**, and matters disproportionately because that path reuses the existing MIDI→CV engine, manifest contract, and authoring docs unchanged.
- `decide-base-platform` selected a Python core owning the ES-9 as a class-compliant USB device via one duplex PortAudio/ALSA stream (host channels 9–16 → ES-9 outs 1–8, DC-coupled). The licensing decision leaves this intact.

## Goals / Non-Goals

**Goals:**
- Record the multi-source authoring model and VCV Rack's role as primary test bench.
- Establish what a `.vcv` file contains and whether it is a stable enough source artifact given third-party plugin dependencies.
- Compare the surviving capture paths — MIDI out of Rack, patch transcription, tempo-relative CV/automation capture, and hybrids — with musical-time positioning as a precondition.
- Determine how continuous modulation is represented in musical time, and at what resolution.
- Confirm or revise the ES-9 as the output vehicle by comparison against alternatives.
- Define "canned support track" as a runtime concept and its relationship to the existing manifest voice/output model.
- Validate the source-agnostic bundle contract against VCV Rack, and identify which findings generalize to any future source.
- Produce decision evidence naming exactly what in `decide-song-bundle-manifest` and `decide-base-platform` needs revision.

**Non-Goals:**
- Implement a `.vcv` parser, capture tool, importer, or playback runtime.
- Choose the final manifest revision — that belongs to a follow-on decision change.
- Ship, bundle, host, or redistribute VCV Rack or any Rack engine.
- Bake audio renders as the primary interchange artifact.
- Rewrite `docs/authoring/` here (it is flagged as affected, not rewritten).
- Reopen the Python-core, ES-9 I/O, or kiosk/UI decisions — the licensing decision leaves all three intact.
- Decide the Assimil8or sampler runtime, which remains deferred.

## Decisions

### Musical time is a precondition, not a criterion

Any candidate artifact must position its content on a tempo-relative grid. This is not scored against other qualities — a path that cannot express position in musical time is eliminated, because Maybelle's defining behavior is following Pamela's Pro Workout and honoring external start/reset. The authored BPM is stored as reference metadata describing the grid the material was written against, and is used for display, validation, and fallback, never for playback timing.

*Alternative rejected:* capturing a baked audio render and varispeeding it at playback. It inverts the product's premise, degrades pitch and gate timing, and produces an artifact that cannot be meaningfully edited or validated downstream.

### VCV Rack stays outside the distribution boundary

Maybelle does not ship, bundle, host, link, or redistribute VCV Rack or any Rack engine. Users install and run Rack themselves and Maybelle consumes what comes out of it. This is now a project decision rather than an open question.

Three consequences follow, and all three are improvements: the GPLv3/commercial licensing hazard disappears; ARM64 availability for Rack and its plugins stops being a blocking concern, since nothing Rack-shaped runs on the Pi; and Maybelle retains **sole ownership of the ES-9 duplex stream**, which preserves the CV input path for runtime bank/song/transport selection. That last point had been the most likely practical blocker in the earlier framing.

*Alternative rejected:* relicensing Maybelle to GPLv3 to absorb an engine. Not proposed; it would need to be a deliberate, separate project-level decision.

### Adding VCV Rack widens the contract rather than replacing it

The bundle stays source-agnostic. VCV Rack, Bitwig, Ardour, and future tools are producers of the same normalized artifact; VCV Rack is simply the one being tested against first and the one that exercises the contract hardest. Nothing in `research-daw-interchange-options` or `decide-song-bundle-manifest` is discarded — the manifest's voice/output mapping, banks, selection, and calibration model all still stand, and the SMF path stays fully supported.

*Alternative rejected:* treating `.vcv` as the single v1 input and demoting the DAWs. It would strand completed work and leave the project with one unvalidated input contract instead of one validated and one new.

### Treat the `.vcv` patch as source of record, not the runtime artifact

The patch is what the musician edits and what version control should hold; the captured artifact is what the Pi plays. This is the familiar **source + build-output** pattern: the bundle carries the `.vcv` for provenance and the captured events for playback. It keeps "the middleware holds the file" true, and it means a wrong capture decision can be revisited without re-authoring music.

### Evaluate the MIDI path first, and honestly

If MIDI can be captured out of VCV Rack with adequate fidelity, it is by a wide margin the cheapest answer: it is natively event-shaped and tempo-relative, it reuses the existing MIDI→CV engine, manifest contract, and authoring documentation unchanged, and it makes VCV Rack just another producer of the artifact the DAWs already produce. Its likely limitation is continuous modulation resolution — the same wall the DAW path hit. The spike should establish early whether that wall is in the same place, because a positive result here collapses most of the remaining work.

### Evaluate transcription on its real costs

Parsing `patch.json` and lifting Impromptu step data into the manifest yields clean, normalized, natively re-clockable data. The cost is coupling to a third-party plugin's undocumented serialization, generalizing to no other module, and silently dropping anything the patch does that is not step-sequencer state. It should be scored against that reality, and is most defensible in hybrid form.

## Risks / Trade-offs

- **Continuous modulation may not survive musical-time capture at usable resolution** → This is the spike's central technical risk and replaces the licensing risk as the thing most likely to force a rethink. Mitigation: establish the required resolution from actual authored material before choosing a representation, and separate modulation that must be sample-accurate from modulation that is merely smooth.
- **`.vcv` is not a stable interchange format** → It is a save format for a specific Rack version plus a specific set of installed plugins, not a documented interchange contract. A plugin update or missing module can change or break a patch. Mitigation: pin/vendor plugin versions, record the failure behavior, and keep the captured artifact so a patch that no longer opens is not a lost song.
- **Transcription couples the project to one plugin vendor** → Impromptu Modular's serialization is not a public contract. Any transcription work is maintenance debt that a plugin update can invalidate. Prefer paths that do not depend on it, or confine the dependency to an authoring-side tool that can fail loudly.
- **Non-deterministic patches** → Random, noise, and feedback modules make a patch non-reproducible. Capturing freezes one take (arguably the point of a "canned" track); transcription drops the behavior. Record the intended semantics rather than picking silently.
- **Capture is a manual step the user must perform correctly** → Whatever the path, someone has to get material out of Rack in the right channel order at the right grid. Mitigation: specify the capture procedure precisely enough to document, and prefer paths whose mistakes are detectable by validation rather than audible only in the rack.
- **What is canned cannot be changed at runtime** → Runtime rack CV selects and transports canned tracks but cannot alter their content. This must be stated so it is authored around rather than discovered at a gig.
- **`docs/authoring/` teaches the opposite framing** → Its headline is "Author MIDI, not CV" and it covers only Bitwig and Ardour. Flag it as partly stale on landing so users and agents are not given contradictory guidance in the interim.
- **The direction is still being iterated** → Scope this spike to evidence and a recommendation. Avoid artifacts that assume the capture path is settled, and keep the source-agnostic contract genuinely source-agnostic so a change of mind is cheap.

## Open Questions

- Can MIDI be captured out of VCV Rack at all, and with what fidelity — notes, velocity, CC, clock? This is the highest-leverage unknown.
- What representation carries continuous CV in musical time, and at what resolution and interpolation? Breakpoints, automation segments, a per-beat grid, or stepped events?
- Which capture path wins, or which hybrid — and is the seam between event data and modulation data clean enough to be worth the split?
- What does the authored BPM actually govern? Confirmed: not playback timing. Open: whether it is used for validation, for display, for a free-running fallback when no rack clock is present, or all three.
- What happens when the rack clock is absent or stops — does a canned track free-run at the authored BPM, hold, or stop?
- Does the ES-9 remain the recommended vehicle, or does the survey surface a better fit for either the authoring-side capture step or the runtime output stage?
- Does the sampler-baked LFO workaround in `research-synced-lfo-sampler-authoring` still earn its complexity for VCV-sourced material, and does it remain necessary for the MIDI path?
- What is Bitwig's role now — still producing musical data, or increasingly the editor/arranger around material authored elsewhere?
