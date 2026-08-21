## 1. Patch File Inventory

- [x] 1.1 Save a representative VCV Rack patch containing the sequencers currently in use (Impromptu PhraseSeq16 / GateSeq64 / Foundry) plus an audio-out module, and inspect the `.vcv` container: archive format, contents, and directory layout. (`examples/vcv-rack/prog-riff-v1.vcv`; zstd-compressed tar → `patch.json`. See `research-findings.md` §1.1.)
- [x] 1.2 Inspect `patch.json`: module identity (plugin slug, module slug, version), per-module state, cable topology, engine sample rate, and any tempo representation. (No patch-level tempo; ports and params are integer-indexed. §1.2, §1.5.)
- [x] 1.3 Record where per-step sequence data actually lives for the Impromptu modules — step CV, gate types, gate probability, ties. (Pitch CV = plain 1V/oct floats; gates/probability/ties = undocumented bit-packed ints. §1.3.)
- [ ] 1.3a Determine how stable the Impromptu `attributes` bit layout is across plugin versions. (blocked: needs a second plugin release to compare)
- [ ] 1.4 Open the patch on a machine missing one or more referenced plugins; record the failure behavior and what patch data survives.
- [ ] 1.5 Record whether `.vcv` is documented as an interchange format or only as a version-coupled save format, and what pinning or vendoring would be needed to treat it as a source artifact. (Evidence already gathered: embeds a machine-specific device binding and unstable driver indices. §1.6.)
- [x] 1.6 Record portability hazards found in the saved patch: serialized `deviceName`, integer `driver` index, stray non-patch files inside the archive, and `dcFilter`. (§1.6.)
- [x] 1.7 Record why the example patch cannot drive CV as saved: `dcFilter: true`, stereo `AudioInterface2`, consumer audio device, and nothing cabled to the audio interface. (§1.7.)

## 2. Authoring-Time Signal Path

- [ ] 2.1 Connect VCV Rack on the authoring machine to the ES-9 as a class-compliant audio device; record the audio module used (`Audio-8`/`Audio-16`), driver, sample rate, and block size.
- [ ] 2.2 Map VCV Rack output channels to physical ES-9 jacks; confirm the DC-coupled front-panel outputs carry CV/gates and that the AC-coupled 1/4-inch mains are not used.
- [ ] 2.3 Measure VCV Rack's output voltage convention against the rack's expectations (1V/oct pitch, gate high level, trigger width); record any scaling or calibration offset.
- [ ] 2.4 Compare this channel map against the runtime map recorded in `docs/authoring/` (host channels 9–16 → outs 1–8) and record whether they agree.
- [ ] 2.5 Verify a sequence authored in VCV Rack plays correctly into the rack through the ES-9 end to end, and capture the working configuration as the reference bench setup. (Note: `prog-riff-v1.vcv` cannot serve as-is — see §1.7. Build the reference patch from `Audio-8`/`Audio-16` with `dcFilter` off.)
- [ ] 2.6 **Rebuild the sample patch as `examples/vcv-rack/prog-riff-v2.vcv`** with the Chinenual MIDI Recorder wired in — Clocked BPM out → recorder BPM in, one voice per recorder track, CC expander if needed, "Start at first note gate" enabled. Keep the musical content identical to v1 so the two stay comparable. Blocks task groups 2 and 3 bench work; checklist in `examples/vcv-rack/README.md`.
- [ ] 2.7 Optionally build a separate ES-9 monitoring variant using `Audio-8`/`Audio-16` with `dcFilter` off and the CV/gate outputs actually cabled.

## 3. Capture Path Comparison

- [ ] 3.1 Define the evaluation matrix, with **musical-time positioning as a precondition** rather than a scored criterion: fidelity, determinism, artifact size, Pi playback cost, implementation complexity, runtime-CV response, reuse of existing MIDI→CV and manifest work, and missing-plugin failure behavior.
- [x] 3.2 **Evaluate the MIDI path first.** Determine whether VCV Rack can emit MIDI to a port and whether it can be captured to a file. (**Confirmed viable.** Core `CV-MIDI`/`CV-Gate` emit to ports; the Chinenual MIDI Recorder writes SMF directly from inside the patch — 10 poly tracks, CC expander, and a BPM input using Impromptu's Clocked convention. See `research-findings.md` §3.)
- [ ] 3.2a Bench the Chinenual MIDI Recorder against `prog-riff-v2.vcv` (see task 2.6): capture timing accuracy vs. the 6 PPS grid, tempo write-through from Clocked, track count sufficiency, and CC fidelity. (blocked: needs the Rack bench)
- [ ] 3.2b Confirm whether the piece uses slides or ties, and decide whether slide becomes a manifest-side per-step attribute as recommended for probability. (blocked: needs Rack, or decoding `attributes`)
- [ ] 3.2c Record the full 4-phrase arrangement as a linear capture pass and confirm no phrase/sequence structure is needed downstream.
- [ ] 3.3 Evaluate **patch transcription**: extract Impromptu sequencer state from `patch.json` into manifest-shaped data, and record what the patch does that transcription would silently drop.
- [ ] 3.4 Evaluate **tempo-relative CV/automation capture**: how a continuous curve is captured as breakpoints, automation segments, a per-beat grid, or stepped events.
- [ ] 3.5 Evaluate **hybrids**: discrete sequence data as events plus continuous modulation as automation curves; record where the seam falls and how the two stay in sync.
- [ ] 3.6 Determine the modulation resolution actually required, measured against real authored material, and record which curves cannot be captured in musical time and what the fallback is.
- [ ] 3.7 Record what each path loses relative to running the patch live in VCV Rack, including generative, random, and feedback behavior, and state the intended semantics for non-deterministic patches.
- [ ] 3.8 Record baked audio rendering as examined and excluded, with the wall-clock/re-clocking reason, and note any narrow exception (such as a fixed-length one-shot with no tempo relationship) and its terms.
- [ ] 3.9 Score the surviving paths against the matrix and rank them.

## 4. Transfer Vehicle Survey

- [ ] 4.1 Compare the ES-9 against other DC-coupled audio interfaces, ADAT-based Expert Sleepers combinations, and MIDI-to-CV hardware converters.
- [ ] 4.2 For each candidate record channel count, DC coupling, voltage range, class-compliance and Linux/Pi support, driver or licensing constraints, and cost.
- [ ] 4.3 State whether any alternative is better for the authoring-side capture step, for the runtime output stage, or for both.
- [ ] 4.4 Confirm or revise the ES-9 as the recommended vehicle, with reasons.

## 5. Clock and Runtime Behavior

- [ ] 5.1 For each surviving capture path, record how position is expressed in musical time and how the authored BPM is stored as reference metadata. (Finding: capture BPM explicitly at capture time — it is only reachable as an unnamed param index on a third-party clock module. §1.5.)
- [ ] 5.2 Determine how the rack clock, start, and reset drive playback of a canned support track, and record timing accuracy and drift risk.
- [ ] 5.3 Decide what the authored BPM is actually used for — validation, display, free-running fallback, or all three.
- [ ] 5.4 Define behavior when the rack clock is absent or stops: free-run at authored BPM, hold, or stop.
- [ ] 5.5 Determine how ES-9 input CV for song banks, song selection, channel banks, and transport selects and controls canned support tracks, confirming quantization, debounce, hysteresis, and latching stay in Maybelle.
- [ ] 5.6 State explicitly what is not controllable at runtime once material is canned, so it is authored around rather than discovered at a gig.
- [ ] 5.7 Confirm Maybelle retains sole ownership of the ES-9 duplex stream, including the input channels used for rack CV and clock.

## 6. Multi-Source Contract Validation

- [ ] 6.1 Evaluate the source-agnostic bundle contract from `research-daw-interchange-options` against VCV Rack; record which bundle fields VCV Rack can produce directly, which need a capture step, and which it cannot produce.
- [ ] 6.2 State whether the contract must change to admit VCV Rack, or whether VCV Rack fits it through capture.
- [x] 6.3 Identify which findings generalize to any future authoring source, and which are VCV-specific. (Gate/tie are easier from MIDI than from `.vcv`; probability is unavailable from every source; portability hazards are VCV-only. See `research-findings.md` §2.)
- [ ] 6.6 Decide whether probability becomes a Maybelle-side manifest attribute rather than a sourced one. (Recommended in §2: no source can deliver it — VCV only via brittle decoding, Bitwig drops it on export, DAWproject has no field for it, Ardour lacks the concept.)
- [ ] 6.7 Define the convention for overlapping/legato notes and whether Maybelle retriggers the gate, since MIDI carries tie as note duration rather than as a flag.
- [ ] 6.8 Decide whether the PhraseSeq16 gate-type abstraction is worth preserving anywhere, or whether resolved gate edges suffice for every source.
- [ ] 6.4 Confirm the SMF path remains fully supported and record what changes, if anything, for Bitwig and Ardour.
- [ ] 6.5 Record the licenses of VCV Rack and the third-party plugins in use as user-facing information, and confirm no captured artifact or generated tooling inherits a license incompatible with the project's MIT-compatible constraint.

## 6b. Sample Reference Capture

- [ ] 6b.1 Record where sample references live in a VCV Rack patch — assets bundled inside the `.vcv` archive versus assets referenced by path outside it — and do the same for the DAW sources.
- [ ] 6b.2 For each referenced sample, define what is captured: logical name, source location, file format, and drift-detection identity such as a content hash and size.
- [ ] 6b.3 Record the failure behavior when a referenced sample is missing, renamed, or moved on the authoring machine.
- [ ] 6b.4 Decide whether samples travel inside the song bundle, are referenced from a shared library outside it, or both.
- [ ] 6b.5 Define how a sample reference binds to a destination on the target sampler, with the Rossum Assimil8or as the default.
- [ ] 6b.6 Define the provenance and licensing metadata that must accompany a sample, consistent with the project's music-licensing review requirement.
- [ ] 6b.7 Confirm the playback boundary in writing: the on-board sampler plays the sample, Maybelle emits the gate/trigger and modulation CV through the ES-9, and sampler storage provisioning belongs to `research-sampler-sample-sync`.

## 7. Decision Handoff

- [ ] 7.1 Recommend one capture path, with the rejected paths and the reason each was eliminated.
- [ ] 7.2 Define what a canned support track contains, how it is stored in a song bundle, how it binds to ES-9 outputs, and what provenance is retained alongside the `.vcv` source.
- [ ] 7.3 List which requirements of `decide-song-bundle-manifest` must be extended, and how.
- [ ] 7.4 Confirm which `decide-base-platform` decision points are unaffected by the licensing decision (Python core, ES-9 I/O stack, kiosk/web UI) and which reopen (song-bundle format).
- [ ] 7.5 State the impact on `research-synced-lfo-sampler-authoring`: whether sampler-baked LFOs still earn their complexity for VCV-sourced material, and whether they remain necessary for the MIDI path.
- [ ] 7.6 State Bitwig's and Ardour's roles going forward under the multi-source model.
- [x] 7.7 Add `docs/authoring/vcv-rack.md`, section the FAQ, and reframe the authoring README away from "Author MIDI, not CV" to cover all three tools. (Done 2026-08-20; VCV material marked provisional.)
- [ ] 7.8 Hand the sample-reference findings to `research-sampler-sample-sync`.
- [ ] 7.9 List the unresolved spikes and bench tests that must complete before any parser, capture tool, or playback code is written.

## 8. Verification

- [ ] 8.1 Review the research output against every `vcv-rack-authoring-path-research` requirement.
- [x] 8.2 Update `README.md` to reflect the multi-source authoring model, the VCV Rack direction, and the licensing boundary.
- [x] 8.3 Run `openspec validate` for this change. (passes, 2026-08-20)
