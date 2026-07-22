## Context

Maybelle 314 reads authored music and emits CV/gate/trigger/modulation through the ES-9, following the rack's master clock. Authoring happens in a DAW (Bitwig or Ardour); the portable export is a Type-1 Standard MIDI File carrying **notes + velocity only**. Everything else Maybelle needs — which voice drives which ES-9 output, gate vs trigger, banks/songs, how rack CV selects them, calibration, and modulation — lives in a Maybelle-owned **manifest** that travels with the SMF as a **song bundle**.

This change decides the v1 bundle structure and manifest format. It synthesizes prior decisions:
- **Bundle = SMF(s) + manifest** (`research-daw-interchange-options`); the manifest owns channel→ES-9 map, banks, calibration, CV-selection/debounce.
- **MIDI→CV voice model** (`docs/authoring/`): one note carries pitch+gate+velocity; one monophonic voice per track; Maybelle fans a voice out to separate outputs.
- **ES-9 rig facts and profile** (`investigate-es9-config-profiles`): 8 DC outs (DAW ch 9–16 → outs 1–8), 14 DC ins, calibration/`{scale_fs_per_volt, offset_fs}` owned in software; a reusable rig **profile** already has its own schema.
- **Selection model** (README): rack CV quantized to the number of choices with debounce/hysteresis/latching.
- **Modulation** (`research-synced-lfo-sampler-authoring`): stepped CV as notes; synced LFOs baked into sampler waveforms and fired by trigger notes.
- **Consumers**: the router screen reads the track↔output map; Backstage is the manifest editor; all surfaces share one manifest model.

## Goals / Non-Goals

**Goals:**
- Decide bundle packaging, manifest serialization, and versioning.
- Define the voice/output mapping, selection, modulation, and asset fields.
- Define the two-layer split between the reusable ES-9 rig profile and the per-song manifest.
- Define validation and keep the format permissively licensed.
- Give `decide-base-platform` a concrete bundle-format input.

**Non-Goals:**
- Implement a parser, validator, importer, runtime, or Backstage editor.
- Finalize the ES-9 profile internals (its own change) or the visual editor (Backstage change).
- Lock every field; some remain provisional pending the runtime/hardware bench.

## Decisions

### Bundle = a directory (zippable) of manifest + SMF(s) + assets

A song bundle is a **directory** (optionally zipped for transport) containing a `manifest.json`, one or more `.mid` files, and an optional `assets/` folder (waveforms, samples, optional audio stems). A directory keeps large/binary assets out of the manifest and is trivial to inspect, diff, and version.

Rationale: separates human-authored config (manifest) from binary/media (assets) and from the DAW export (SMF); zip is only a transport wrapper.

Alternatives considered:
- Single embedded file (base64 assets in JSON): self-contained, but bloated and un-diffable.
- DAWproject-style zip-only: fine, but a plain directory is simpler for a Pi appliance and Git.

### Canonical serialization: JSON with a published JSON Schema 2020-12

The manifest is **JSON**, validated by a **published JSON Schema (2020-12)**. This aligns with the agent-control-surface decision (its CLI `--json` contract uses JSON Schema 2020-12), is the most tooling/agent-friendly, and is trivially validatable.

Rationale: one schema serves validation, the CLI, the Backstage editor, and agent assistance; JSON/JSON-Schema licensing is permissive.

Alternatives considered:
- TOML/YAML canonical: friendlier to hand-edit, but weaker tooling and a second parser; may be offered later as an **authoring front-end that compiles to the canonical JSON**, not the source of truth.

### Two layers: reusable ES-9 rig profile + per-song manifest

The **ES-9 rig profile** (physical outputs/inputs, coupling, calibration constants, sample rate/buffer — `investigate-es9-config-profiles`) is a **separate, reusable document** describing the rig across songs. The **song manifest references a profile by id** and binds this song's voices to the profile's output roles. Calibration lives in the profile; per-song *assignment* lives in the manifest.

Rationale: a performer's rig is stable across many songs; duplicating calibration per song invites drift. Referencing keeps one source of rig truth.

Alternatives considered:
- Embed the full profile in every manifest: self-contained bundles, but duplicated calibration and drift risk. (A bundle MAY embed a profile snapshot for portability, but the referenced profile is authoritative.)

### Voice model mirrors the MIDI→CV docs

The manifest binds each SMF **track (or MIDI channel)** to a **voice**, and each voice to ES-9 **output roles**: `pitch_cv`, `gate` (mode gate|trigger), and optional `velocity`. Percussion is a note→output map (each note → a trigger output). This is the exact model in `docs/authoring/midi-to-cv-model.md`, so the manifest is the machine form of the authoring guidance.

Rationale: consumers (runtime, router screen, Backstage help) and the authoring docs then share one model.

### Selection config encodes the README CV-selection model

Runtime selection (song bank, song, channel bank, transport, performance params) is configured per control as `{ cv_in, choices, debounce_ms, hysteresis, latch }`, matching the README's quantize-to-N-choices Assimil8or-style behavior.

### Modulation references, not embedded automation

Modulation is expressed as **references**, never as DAW automation (which doesn't survive export): a `stepped` entry naming a track that encodes CV steps as notes, and a `sampler_lfo` entry naming a trigger track + a bundled waveform asset + its sampler target. This matches `research-synced-lfo-sampler-authoring`.

### Versioning and determinism

A top-level `schema_version` (SemVer) governs the format; the runtime rejects unknown **major** versions and tolerates unknown minor/patch additively. Timestamps (`created`) are authored/passed in, never generated implicitly, so bundles are reproducible.

### Illustrative shape (non-normative)

```json
{
  "schema_version": "1.0.0",
  "bundle": { "id": "set-a", "name": "Opening Set", "authored_with": "bitwig", "created": "2026-07-22" },
  "es9_profile": "studio-rig-v1",
  "sources": [ { "type": "smf", "file": "opening.mid" } ],
  "voices": [
    { "id": "bass", "source": { "track": "Bass" }, "mono": true,
      "outputs": { "pitch_cv": { "role": "out-1" },
                   "gate": { "role": "out-2", "mode": "gate" },
                   "velocity": { "role": "out-3", "enabled": true } } }
  ],
  "percussion": [ { "note": 36, "role": "out-4", "mode": "trigger" } ],
  "modulation": [
    { "type": "stepped", "track": "ModA", "role": "out-5" },
    { "type": "sampler_lfo", "trigger_track": "LfoTrig", "waveform": "assets/lfo1.wav", "target": "assimil8or" }
  ],
  "banks": { "song_banks": [ { "songs": [ "opening" ] } ], "channel_banks": [] },
  "selection": {
    "song_bank":    { "cv_in": "in-1", "choices": 4, "debounce_ms": 30, "hysteresis": 0.05, "latch": true },
    "song":         { "cv_in": "in-2", "choices": 8, "debounce_ms": 30, "hysteresis": 0.05, "latch": true },
    "channel_bank": { "cv_in": "in-3", "choices": 4, "debounce_ms": 30, "hysteresis": 0.05, "latch": true },
    "transport":    { "cv_in": "in-4", "mode": "gate" }
  },
  "clock": { "source": "external", "clock_in": "in-5", "start_in": "in-6", "reset_in": "in-7" },
  "assets": [ { "file": "assets/lfo1.wav", "kind": "waveform" } ]
}
```

Output roles (`out-1`…`out-8`, `in-1`…`in-14`) are the ES-9 profile's role ids, so the manifest never hard-codes physical jacks — the profile maps roles to jacks/calibration.

## Risks / Trade-offs

- Over-specifying v1 before the runtime exists -> Mitigation: mark runtime-dependent fields provisional; keep the schema additively versioned.
- JSON is verbose to hand-edit -> Mitigation: Backstage is the primary editor; an optional TOML/YAML front-end can compile to JSON later.
- Profile-vs-manifest split confuses authors -> Mitigation: roles are the only cross-reference; document clearly and allow an embedded profile snapshot for portability.
- Track-vs-channel ambiguity for voice identity -> Mitigation: support both `track` and `channel` selectors; decide a default in the open questions.
- Selection/bank model may not fit multi-SMF sets -> Mitigation: treat `sources` as a list and validate bank references against it.

## Migration Plan

No runtime migration is required; this change adds planning artifacts and a proposed schema. Output feeds `decide-base-platform` (bundle format), the future importer/validator (built on the JSON Schema), the Backstage editor, and the router screen.

Rollback is limited to removing this OpenSpec change before implementation.

## Open Questions

- Is a voice identified by **track name**, **MIDI channel**, or either — and what is the default when both are present?
- One SMF per song or one multi-song SMF per bundle; how do banks/songs reference `sources`?
- What is the exact **stepped-CV note encoding** (note→value mapping, resolution)?
- Is the ES-9 profile **referenced only**, or may a bundle **embed a snapshot** for portability (and which wins)?
- Zip vs plain directory as the canonical on-disk form for the Pi, and how are asset paths resolved/sandboxed?
- How are calibration and voltage ranges surfaced to authors (read-only from the profile) versus overridable per song?
- Does `selection` need per-choice labels/targets (e.g. which songs a bank contains), and where do those live?
