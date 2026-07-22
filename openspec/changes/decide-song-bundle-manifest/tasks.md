## 1. Bundle Structure

- [ ] 1.1 Decide the bundle layout (directory with `manifest.json` + `.mid`(s) + optional `assets/`) and the zip transport wrapper.
- [ ] 1.2 Define asset kinds (waveforms, samples, optional stems) and how asset paths are resolved and sandboxed.

## 2. Manifest Serialization and Versioning

- [ ] 2.1 Decide the canonical serialization (JSON) and publish a JSON Schema 2020-12 sketch.
- [ ] 2.2 Define `schema_version` (SemVer) and forward/backward-compatibility rules (reject unknown major; tolerate unknown minor/patch).
- [ ] 2.3 Confirm determinism (authored `created` timestamp; no implicit generation) and permissive licensing.

## 3. Voice / Output Mapping

- [ ] 3.1 Define how SMF tracks/channels map to voices and to ES-9 output roles (pitch_cv, gate/trigger, velocity), matching `docs/authoring/midi-to-cv-model.md`.
- [ ] 3.2 Define the percussion note→output(role) trigger map.
- [ ] 3.3 Decide the voice-identity selector (track name vs MIDI channel) and the default.

## 4. Selection and Clock

- [ ] 4.1 Define the CV-selection config per control (`cv_in`, `choices`, `debounce_ms`, `hysteresis`, `latch`) per the README selection model.
- [ ] 4.2 Define the clock/transport config (external clock/start/reset inputs) and how the file grid is re-clocked.

## 5. Modulation, Assets, and Profile Link

- [ ] 5.1 Define modulation references (stepped-CV tracks; sampler-LFO trigger + waveform + target), never DAW automation.
- [ ] 5.2 Define the reference to the reusable ES-9 rig profile (roles as the only cross-reference) and whether an embedded snapshot is allowed.
- [ ] 5.3 Define bundled asset entries and their linkage to modulation/sampler references.

## 6. Validation and Handoff

- [ ] 6.1 Define schema validation with pass/warn/fail semantics (reuse the agent-control-surface validator direction).
- [ ] 6.2 Feed the bundle-format decision into `decide-base-platform` and list follow-up spikes (importer, validator, editor) before implementation.
- [ ] 6.3 Mark runtime-dependent fields provisional pending `decide-base-platform` and the hardware bench.

## 7. Verification

- [ ] 7.1 Review the output against every `song-bundle-manifest` requirement.
- [ ] 7.2 Run OpenSpec validation or status checks for the completed change.
