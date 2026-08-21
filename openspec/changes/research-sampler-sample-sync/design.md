## Context

Maybelle 314 is explicitly **not** a sampler. The README states the Pi is not the rack voice and not a sampler, and the authoring docs put the sample playback in the rack: Maybelle emits a gate or trigger through the ES-9 into the sampler's trig-in, and the sampler plays its own stored content. The default sampler is the **Rossum Assimil8or**, which reads from its own removable SD card.

That leaves an unowned gap. A song bundle references sample assets — mostly `.wav` — that must physically exist on the sampler's card, under the right name, in the right slot, in a format the module accepts. Nothing currently keeps the card and the songs in agreement. The failure mode is bad: a mismatch is invisible on the authoring machine and shows up as silence or the wrong sound in the rack, most likely at the worst time.

Adjacent work does not cover this. `research-synced-lfo-sampler-authoring` generates LFO waveforms and writes presets — it produces content but does not manage it. `research-vcv-rack-authoring-path` captures sample *references* from authored material — name, location, format, content identity — but stops at the bundle. This change owns turning those references into a correctly provisioned card.

Two properties shape the whole design. First, this is **provisioning, not playback** — offline, authoring-side, on the user's machine, with the Pi runtime uninvolved. Second, it **writes to removable media the user owns**, which is a performer's sample card that may hold material this project knows nothing about. A tool that can silently overwrite or mirror-delete that card is worse than no tool at all.

Unverified and needing hardware or documentation confirmation: the Assimil8or's card type and filesystem, its directory and preset layout, its filename constraints, and its accepted WAV encodings, bit depths, sample rates, channel counts, and length limits. The Assimil8or runtime spike is deferred, so this research pins down the storage and format facts ahead of it.

### Reuse evaluation outcome (2026-08-21)

`decide-build-vs-reuse` evaluated the two obvious candidates. The results pull in opposite directions and both change this design.

**A8Manager cannot be used.** It is functionally close — it manages Assimil8or presets and sample files, scans a card, and offers to fix what it finds. But the repository carries **no license**: no license field, no `LICENSE` or `COPYING` file. Secondary coverage calling it "open source" is not supported by the repository itself, and public source is not a grant. Default copyright applies, so it cannot be depended on, wrapped, vendored, or invoked as a shipped component. Pointing users at it for their own editing is unaffected and remains fine.

Two consequences for this design: the Assimil8or's card layout, filename rules, and WAV constraints **stay in scope here**, and they must be sourced from Rossum's documentation and the hardware rather than by reading an unlicensed codebase. Those constraints are facts about the module, not anyone's intellectual property.

**rclone validates the design but may be the wrong size.** It is MIT, and `rclone copy` states verbatim that it "Doesn't delete files from the destination" — the additive-by-default semantics specified below, arrived at independently. `--checksum`, `--dry-run`, `--max-delete`, and `--backup-dir` cover the rest of the safety model.

That is useful confirmation that this design is standard rather than idiosyncratic. It is not automatically the implementation. The job here is copying a few dozen WAV files to an SD card with verification — a walk, a `hashlib` pass, a compare, `shutil.copy2`, and a re-hash, all stdlib. Requiring users to install a large Go binary, or vendoring one per platform, may be *less* small than the code it replaces. The decision is deferred to task 0.4 with stdlib a serious contender.

The requirements below are therefore unchanged in substance; what changed is that their implementation is explicitly open, and that A8Manager is not a way out of the card-constraint work.

## Goals / Non-Goals

**Goals:**
- Pin down the Assimil8or's storage layout and sample format constraints.
- Define a sync model that detects drift by content identity rather than filename.
- Define a safety model strong enough to point at a performer's card without hesitation.
- Define deterministic logical-name-to-file-and-slot mapping.
- Decide how non-conforming samples are handled, and keep originals immutable.
- Keep sampler-agnostic concerns separable from Assimil8or-specific ones.
- Produce evidence sufficient to specify the tool before writing card I/O.

**Non-Goals:**
- Implement the sync tool, a WAV converter, a preset writer, or any card I/O.
- Play, stream, or serve sample content from the Pi.
- Bundle or redistribute third-party sample content as part of the project.
- Decide the deferred Assimil8or runtime behavior beyond the storage and format facts needed here.
- Manage the sampler's firmware or non-sample configuration.

## Decisions

### The bundle is authoritative; the card is reconciled toward it

Sync is one-directional by default. The set of samples a song bundle references defines the intended state, and the card is brought toward it. Reading the card back into the bundle is not a goal — that direction invites a card of unknown provenance becoming the source of truth for a song.

*Alternative rejected:* bidirectional sync. It requires conflict resolution for a problem the workflow does not actually have, and it makes the failure modes much harder to reason about.

### Drift is detected by content identity, never by filename

Filenames on removable media are unreliable — case folding, truncation to the module's constraints, and manual edits on the card all break name-based comparison. Comparing a content hash plus size means a renamed file is recognized as the same content and a same-named file with different content is recognized as different. This costs a full read of the card's referenced files per sync, which is acceptable for the data sizes involved.

*Alternative rejected:* trusting filename and modification time. Cheaper, and wrong precisely in the cases that cause silent performance failures.

### Never mirror-delete; unreferenced files are left alone by default

A card may hold a performer's own material that no Maybelle song references. Treating "not referenced by this bundle" as "should be deleted" would destroy it. The default is therefore additive and corrective: create what is missing, fix what differs, and report what is unreferenced without touching it. Removal is available only as an explicit, separately confirmed operation.

*Alternative rejected:* mirror semantics as the default, with an opt-out. The safe direction has to be the default one; an opt-out protects only the users who already knew to look for it.

### Dry-run and diff precede every write

Every sync produces a complete diff of intended creates, overwrites, and deletions before anything is written, and destructive operations need explicit confirmation rather than being implied by having run the sync. This mirrors the risk-tiered safety already adopted for the agent control surface in `decide-base-platform`, where file writes are dry-run-plus-diff — the same reasoning applies, more strongly, to removable media.

### Originals are immutable; conversion produces derived artifacts

If a sample must be converted to satisfy the module's constraints, the source file on the authoring machine is never modified. The derived artifact is written elsewhere with its relationship to the original recorded, and any conversion that changes audible content is reported rather than performed silently.

### Split sampler-agnostic from sampler-specific behind a seam, but build the Assimil8or path first

Drift detection, diffing, safety, verification, and validation reporting are sampler-agnostic. Layout, filename rules, format constraints, and preset binding are not. Keep the seam clean so a second sampler is additive, but do not pay for a generalized abstraction against a single known target and a deferred runtime spike.

### Generated waveforms are ordinary sample assets

The LFO waveforms produced by `research-synced-lfo-sampler-authoring` are `.wav` files on a card like any other. They travel through this same sync path rather than a parallel one, and are distinguished only by their recorded provenance.

## Risks / Trade-offs

- **Destroying a performer's sample card** → The dominant risk. Mitigated by additive-by-default semantics, no mirror-delete, mandatory dry-run diff, explicit confirmation for destructive operations, and post-write verification. This risk justifies its own requirement rather than a footnote.
- **Interrupted or partial writes** → Card removal or a killed process mid-write leaves an ambiguous card. Mitigate with post-write verification and a defined, diagnosable partial state; specify how the user recovers.
- **Unverified Assimil8or constraints** → Layout, filename rules, and accepted formats are assumptions until confirmed against the module or its documentation. Mark them unverified and confirm on hardware before any implementation; a wrong filename rule produces a card that mounts fine and does not play.
- **Silent audible change from conversion** → Sample-rate or bit-depth conversion can alter the sound. Always report conversions; prefer rejecting with a clear message over converting quietly.
- **Collision and rename rebinding the wrong sample** → Two logical references mapping to one constrained filename could silently bind a song to the wrong audio. The mapping must be deterministic and collision-detecting, and a collision is an error, not something resolved by a suffix and a shrug.
- **Capacity exhaustion mid-sync** → A card that fills partway through leaves a partly provisioned song. Check capacity against the full intended state before writing, not per file.
- **Scope creep toward a sample manager** → The tool syncs what bundles reference. Browsing, auditioning, tagging, and library curation are a different product; keep them out.
- **Sample licensing** → The project must not bundle or redistribute third-party sample content. Record provenance per reference and keep the tool a mover of the user's own assets.

## Open Questions

- What card media and filesystem does the Assimil8or actually require, and what are its real directory, preset, and filename rules?
- What WAV encodings, bit depths, sample rates, channel counts, and length limits does it accept, and how does it behave when given something outside them?
- Does the destination bank/preset/channel slot come from the manifest, from the Assimil8or preset that `research-synced-lfo-sampler-authoring` writes, or from both — and which wins on disagreement?
- Do samples live inside the song bundle, in a shared library outside it, or both? This is asked in `research-vcv-rack-authoring-path` and answered for provisioning here.
- Should the tool ever read the card to help author a bundle, or is one-directional sync sufficient in every workflow the project expects?
- Is there a non-destructive way to identify card contents the user cares about, so unreferenced files can be reported usefully rather than just counted?
- How does a user verify, at the rack and before a performance, that a card matches the songs loaded on Maybelle — and should Maybelle surface that check in Backstage?
