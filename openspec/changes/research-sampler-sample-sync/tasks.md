## 0. Reuse Evaluation (do this first)

- [x] 0.1 Evaluate **A8Manager** against this change's card-management, validation, and preset-binding requirements. (Functionally a strong match — manages presets and samples, scans a card, offers fixes.)
- [x] 0.2 Resolve A8Manager's blocking questions. (**No license at all** — no license field, no LICENSE/COPYING file. Cannot be a dependency, wrapped, vendored, or invoked as a shipped component. Point-at only. Linux build still absent.)
- [x] 0.3 Evaluate **`rclone copy`** against the sync and safety models below. (**MIT**, and `copy` verbatim "Doesn't delete files from the destination." `--checksum`, `--dry-run`, `--max-delete`, `--backup-dir` cover the specified model. Design validated.)
- [ ] 0.4 Decide the implementation: rclone-installed vs vendored vs **stdlib** (`hashlib` + `shutil`). Stdlib is a serious contender for copying a few dozen WAVs — a large Go binary may be less small than the code it replaces.
- [x] 0.5 Re-scope in light of 0.1–0.3. (Sections 1 and 4–5 **stay in scope** — A8Manager's missing license means the Assimil8or's card layout, filename rules, and WAV constraints must be sourced from Rossum documentation and the hardware. Sections 2–3 keep their *requirements* but defer their *implementation* to task 0.4.)
- [ ] 0.6 Decide whether to ask the A8Manager author to add a license, which is the prerequisite for any deeper reuse or upstream Linux-build contribution.
- [ ] 0.7 Re-check A8Manager's license periodically; the repository is actively maintained and one may appear.

## 1. Sampler Storage Inventory

- [ ] 1.1 Record the Assimil8or's card media type, required filesystem format, and capacity limits from **Rossum's own documentation and the hardware** — not from A8Manager's unlicensed source. Mark anything unconfirmed as unverified.
- [ ] 1.2 Record the on-card directory layout, where preset data lives relative to sample files, and how samples bind to banks, presets, and channels.
- [ ] 1.3 Record filename constraints: length, permitted characters, and case sensitivity.
- [ ] 1.4 Record accepted WAV encodings, bit depths, sample rates, channel counts, and length or size limits.
- [ ] 1.5 Record the module's behavior when given a file that violates a constraint. (expected blocked: needs the module on hand)
- [ ] 1.6 Confirm the inventory against hardware and mark each item verified or still assumed. (expected blocked: Assimil8or runtime spike is deferred)

## 2. Sync Model

- [ ] 2.1 Define the authoritative set as the sample references from the song bundle, with the card as the reconciliation target, and confirm one-directional sync is sufficient.
- [ ] 2.2 Define drift detection by content hash plus size rather than filename; record the cost of hashing referenced card files per sync.
- [ ] 2.3 Define the classification of each reference: present and matching, present but differing, missing from the card, or present on the card but unreferenced.
- [ ] 2.4 Define how the referenced sets of several songs combine into one target state for a shared card.
- [ ] 2.5 Define behavior for unreferenced files already on the card, defaulting to leaving them untouched and reporting them.
- [ ] 2.6 Define how capacity is checked against the full intended state before writing, and how exhaustion is reported.

## 3. Safety Model

- [ ] 3.1 Specify the dry-run diff: every intended create, overwrite, and deletion, produced before any write.
- [ ] 3.2 Specify which operations are destructive and require explicit confirmation, and confirm mirror-delete is never the default.
- [ ] 3.3 Specify post-write verification against expected content identity.
- [ ] 3.4 Define behavior when the card is removed or the tool is interrupted mid-write, and the resulting diagnosable state.
- [ ] 3.5 Define the recovery procedure from a partial sync.
- [ ] 3.6 Confirm source assets on the authoring machine are never mutated, and define where derived artifacts are written and how their relationship to the original is recorded.
- [ ] 3.7 Compare this safety model against the risk-tiered agent-control-surface model in `decide-base-platform` and record where it must be stricter.

## 4. Name and Slot Mapping

- [ ] 4.1 Define how a logical sample name becomes a filename satisfying the sampler's constraints, and how the mapping is recorded so it is reproducible.
- [ ] 4.2 Define how the destination bank, preset, and channel slot is chosen or read from the manifest.
- [ ] 4.3 Resolve whether the destination comes from the manifest, from the preset written by `research-synced-lfo-sampler-authoring`, or both, and which wins on disagreement.
- [ ] 4.4 Define collision and rename handling such that a song is never silently rebound to the wrong sample; treat a collision as an error.

## 5. Conversion and Validation

- [ ] 5.1 Decide, per constraint violation, whether the tool rejects, warns, or converts, and on what criteria.
- [ ] 5.2 Define what a conversion records so an audible change is never silent.
- [ ] 5.3 Align validation reporting with the pass/warn/fail model in `decide-song-bundle-manifest`.

## 6. Boundaries and Generalization

- [ ] 6.1 State the runtime boundary: offline authoring-side tooling only; Maybelle emits gates, triggers, and modulation CV through the ES-9 and does not read, serve, or play sample content.
- [ ] 6.2 State whether the Pi has any role in sample provisioning, including none.
- [ ] 6.3 Separate sampler-agnostic concerns (drift detection, safety, diffing, verification, validation reporting) from Assimil8or-specific concerns (layout, filename rules, format constraints, preset binding).
- [ ] 6.4 State what a second sampler target would require, and whether generalizing now is worth the cost or the seam is sufficient.
- [ ] 6.5 Define how generated LFO waveforms from `research-synced-lfo-sampler-authoring` travel through this same sync path.

## 7. Provenance and Licensing

- [ ] 7.1 Define the provenance and licensing metadata recorded per sample reference, consistent with the project's music-licensing review requirement.
- [ ] 7.2 Confirm the tool does not bundle or redistribute third-party sample content as part of the project.
- [ ] 7.3 Define how generated content is distinguished from user-supplied and third-party content.

## 8. Decision Handoff

- [ ] 8.1 Recommend a sync model, a safety model, and a mapping scheme.
- [ ] 8.2 List which requirements of `decide-song-bundle-manifest` must be extended to express sample references and sampler destinations.
- [ ] 8.3 Consume the sample-reference findings from `research-vcv-rack-authoring-path` and record any gaps.
- [ ] 8.4 Decide whether Backstage should surface a card-versus-songs verification check before a performance.
- [ ] 8.5 List the unresolved spikes and hardware checks that must complete before any card I/O or conversion code is written.

## 9. Verification

- [ ] 9.1 Review the research output against every `sampler-sample-sync-research` requirement.
- [x] 9.2 Run `openspec validate` for this change. (passes, 2026-08-20)
