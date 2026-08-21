## 1. Principle

- [x] 1.1 State reuse-before-build as a project principle with the burden of proof on building.
- [x] 1.2 Add the principle to `README.md` so it is visible at the project's front door.
- [ ] 1.3 Decide whether proposals should carry a standing "considered for reuse" section, and if so add it to the OpenSpec proposal rules in `openspec/config.yaml`.

## 2. Register — Compile

- [x] 2.1 Record the capabilities already covered by `decide-base-platform`'s stack: Mido, sounddevice/PortAudio, NumPy, FastAPI/Uvicorn. (See `research-findings.md`.)
- [x] 2.2 Record **A8Manager** for Assimil8or preset and sample card management, including its missing Linux build and unstated license.
- [x] 2.3 Record **rclone** for the file-sync engine, noting `copy` rather than `sync`, with `--checksum`, `--dry-run`, `--max-delete`, and `--backup-dir`.
- [x] 2.4 Record the **Chinenual MIDI Recorder** for VCV Rack capture.
- [x] 2.5 Record **AKWF** for single-cycle waveforms and **DAWproject** as an evaluated-and-rejected interchange format.
- [ ] 2.6 Search for prior art on the CV selection layer — quantize-to-N with debounce, hysteresis, and latching — which is currently assumed to be build.
- [ ] 2.7 Search for prior art on clock-following schedulers driven by external pulse input.
- [ ] 2.8 Record candidates for WAV read/write, JSON Schema validation, and the CLI façade.

## 3. Register — Resolve Open Questions

- [ ] 3.1 Determine A8Manager's actual license by inspecting the repository rather than the README.
- [ ] 3.2 Determine whether A8Manager can be built for Linux, and whether contributing a Linux build upstream is preferable to building anything ourselves.
- [ ] 3.3 Confirm rclone's license and whether invoking it as a separate binary carries any obligation.
- [ ] 3.4 Decide whether to depend on rclone being installed, vendor a binary, or implement the small subset needed.
- [ ] 3.5 Determine whether A8Manager's card validation overlaps the manifest's pass/warn/fail model enough to be invoked rather than reimplemented.

## 4. Irreducible Core

- [x] 4.1 Draft the list of capabilities with no credible existing substitute.
- [ ] 4.2 Challenge each core item against task 2.6–2.8 findings and remove anything that turns out to be merely unsearched.
- [ ] 4.3 Record the final core in the register as the definition of what the project builds.

## 5. Admission Criteria

- [x] 5.1 Define the criteria: license compatibility, platform coverage, maintenance health, and the size of the remaining gap.
- [x] 5.2 Define the consumption-mode distinction — vendored, invoked as a separate tool, or user-installed — and its licensing consequences.
- [ ] 5.3 Define the close-but-not-sufficient decision procedure: wrap, contribute upstream, or build.
- [ ] 5.4 Define how and when register rows are re-checked, given that verdicts go stale.

## 6. Apply to Existing Changes

- [ ] 6.1 Re-scope `research-sampler-sample-sync` against rclone: remove the sync-engine mechanics, keep bundle-to-card reconciliation.
- [ ] 6.2 Re-scope `research-sampler-sample-sync` against A8Manager: remove card-layout and preset-binding work it already covers, subject to the Linux and license questions.
- [ ] 6.3 Record the register's effect on `research-vcv-rack-companion-module`'s recommendation.
- [ ] 6.4 Review the remaining open changes for capabilities the register already covers.

## 7. Verification

- [ ] 7.1 Review the output against every `build-vs-reuse-register` requirement.
- [x] 7.2 Run `openspec validate` for this change. (passes, 2026-08-20)
