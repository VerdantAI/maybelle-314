## Why

Authored material depends on sample assets — mostly `.wav` — that are played by an **on-board sampler in the rack, not by Maybelle**. The default sampler is the **Rossum Assimil8or**, which reads its samples from its own removable SD card. Nothing currently keeps that card in agreement with what a song expects: a song can reference a sample that is not on the card, or is on the card under a different name, in a different slot, or in a different revision, and the failure surfaces as silence or a wrong sound in the rack at performance time.

The authoring side is where the samples live and where the song is assembled, so that is where the sync belongs. `research-vcv-rack-authoring-path` establishes what sample references are captured from authored material; this change decides how those references become a correctly provisioned card. `research-synced-lfo-sampler-authoring` already covers generating LFO waveforms and writing Assimil8or presets — it does not cover managing sample assets or the card itself, which is the gap here.

This is provisioning, not playback. It runs offline on the authoring machine, it writes to removable media the user owns, and it must be safe: a tool that can silently overwrite a performer's sample card is worse than no tool.

## What Changes

- Add a research spike for **syncing an on-board sampler's storage with the sample assets a song bundle references**, as offline authoring-side tooling separate from the Pi runtime.
- Inventory the **Assimil8or's storage and sample constraints**: card type, format, and capacity; directory and preset layout; filename constraints; supported WAV encodings, bit depths, sample rates, channel counts, and length limits; and how samples bind to banks, presets, and channels.
- Define the **sync model**: what a sync compares, what it treats as authoritative, and how it detects drift — content hashing, size, and modification metadata — rather than trusting filenames.
- Define the **safety model**, treated as a first-class requirement: dry-run and diff before any write, explicit confirmation for destructive operations, never a blind mirror-delete, verification after write, and safe behavior on interruption or removal mid-write.
- Define **conversion and validation**: what happens when a referenced sample does not meet the sampler's constraints — reject, warn, or convert — and where any converted artifact is stored so the original is never mutated.
- Define **name and slot mapping**: how a logical sample reference in the bundle becomes a concrete filename and destination slot, and how collisions and renames are resolved deterministically.
- Define **multi-song and shared-library behavior**: how a card serving several songs is reconciled, and what happens to samples already present that no song references.
- Determine the **boundary against the runtime**: Maybelle emits gates/triggers and modulation CV to the sampler through the ES-9 and does not read, serve, or play samples.
- Assess **generalizing beyond the Assimil8or**, so a second sampler does not require rewriting the tool.
- Record **licensing and provenance** handling for sample content, consistent with the project's music-licensing review requirement.
- Do not implement the sync tool, a WAV converter, a preset writer, or any card I/O in this change.

## Capabilities

### New Capabilities

- `sampler-sample-sync-research`: Defines how the project researches and records the authoring-side synchronization of an on-board sampler's storage with the sample assets referenced by a song bundle, covering the Assimil8or's constraints, the sync and safety models, name/slot mapping, conversion, and generalization to other samplers.

### Modified Capabilities

- None.

## Impact

- Adds OpenSpec planning artifacts for the sampler sample-sync investigation.
- Consumes the sample-reference findings from `research-vcv-rack-authoring-path` — logical name, source location, format, and content identity.
- Complements `research-synced-lfo-sampler-authoring`, which covers waveform generation and preset writing. Generated LFO waveforms are themselves sample assets and should travel through the same sync path rather than a second one.
- Extends `decide-song-bundle-manifest`: the manifest must express sample references and their sampler destinations.
- Does not affect `decide-base-platform`. This is authoring-side tooling; the Pi runtime is unchanged and still does not play samples.
- Touches the deferred Assimil8or runtime spike by pinning down the module's storage and sample constraints ahead of it.
- No production code, runtime dependencies, hardware integration, or bundled third-party media are introduced by this proposal.
