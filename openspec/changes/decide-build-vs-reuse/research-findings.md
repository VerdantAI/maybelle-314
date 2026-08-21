# Build-vs-Reuse Register

**Status:** First compilation, 2026-08-20. Rows are dated snapshots, not permanent verdicts.

**Principle:** reuse before build. A proposal specifying implementation work is incomplete until it records what was considered for reuse and why it was rejected.

## Register

| Capability | Candidate | License | Consumption | Verdict |
| --- | --- | --- | --- | --- |
| MIDI file parse/write | Mido | MIT | vendored | **Reuse** — chosen in `decide-base-platform` |
| MIDI ports | python-rtmidi | MIT | vendored | **Reuse** |
| ES-9 duplex audio I/O | sounddevice / PortAudio | MIT | vendored | **Reuse** — gated on the ES-9 timing bench |
| Buffers / numerics | NumPy | BSD | vendored | **Reuse** |
| Local server + SSE | FastAPI + Uvicorn | MIT / BSD | vendored | **Reuse** |
| Kiosk display | Chromium + labwc | permissive | system | **Reuse** |
| VCV Rack → MIDI capture | Chinenual MIDI Recorder | GPL-3.0 | **user-installed** | **Reuse** — GPL is fine at this boundary |
| Assimil8or preset + sample card management | **A8Manager** | **none — all rights reserved** | user-installed only | **Point-at only** — cannot be a project dependency |
| File sync engine + safety | **rclone `copy`** | **MIT (confirmed)** | separate binary | **Reuse-eligible** — consumption mode still open |
| Single-cycle waveforms | AKWF | CC0 | content | **Reuse** |
| DAW interchange format | DAWproject | MIT | — | **Rejected** — no note-probability field |
| WAV read/write | soundfile / libsndfile | BSD / LGPL | vendored | **Candidate** — not yet evaluated |
| JSON Schema validation | jsonschema | MIT | vendored | **Candidate** — implied by the 2020-12 decision |
| CLI façade | Typer / Click | MIT / BSD | vendored | **Candidate** |

## The two significant finds

### A8Manager — an open-source Assimil8or card manager already exists

[A8Manager](https://github.com/cpr2323/A8Manager) by Chris Roberts is free and open source, written in C++/JUCE, and manages **presets *and* sample files** on the Assimil8or's SD card. It edits presets on the card, "checks for common issues when preparing samples and presets," and scans a card offering to fix what it finds.

This overlaps `research-sampler-sample-sync` substantially. That change should not respecify card layout, filename rules, or preset binding that A8Manager already handles.

**Two blocking questions, recorded with equal prominence:**

1. **No Linux build.** Windows and macOS only. The author notes "there is no Linux version yet, but since we are using JUCE it should be relatively simple." On a Fedora authoring machine this is currently unusable — which makes **contributing a Linux build upstream** a genuine option to weigh against building anything ourselves.
2. **License unstated in the README.** JUCE is dual-licensed, and a free JUCE application is usually GPLv3, but this must be confirmed from the repository rather than assumed.

Until both are resolved, this is a strong candidate, not a solution.

The project already points at A8Manager in `research-synced-lfo-sampler-authoring` for preset editing. The new information is that it also handles **samples and card validation**.

### rclone — the sync engine and its safety model already exist

`research-sampler-sample-sync` specified content hashing, dry-run diffing, additive-by-default semantics, no mirror-delete, and post-write verification. That is a from-scratch description of a tool that exists.

**Use `rclone copy`, not `rclone sync`.** `sync` deletes destination files absent from the source — precisely the mirror-delete identified as unacceptable against a performer's card. `copy` is additive and never deletes, which is that change's stated default.

| Requirement specified | rclone flag |
| --- | --- |
| Drift by content, not filename | `--checksum` |
| Mandatory preview before writing | `--dry-run` |
| Never mirror-delete | `copy` (deletes nothing by default) |
| Guard on destructive operations | `--max-delete N` |
| Recoverable removals | `--backup-dir` |

**What remains genuinely Maybelle-specific: reconciliation.** Determining which samples a given song bundle requires, resolving those references to concrete files, and reporting drift in the user's terms. That is a small program producing a file list — not a sync engine.

*Confirm rclone's license from its repository before relying on it. It is understood to be MIT, but invoking it as a separate binary would carry minimal obligation in any case.*

## Irreducible core — draft

Capabilities with no credible existing substitute. This is what the project builds:

1. **Clock-following scheduler** — driven by Pamela's pulses arriving on ES-9 input channels, edge-detected in the audio callback.
2. **MIDI → multichannel DC CV/gate engine** — writing pitch CV, gates, triggers, and modulation to ES-9 output channels.
3. **Rack-CV selection layer** — quantize-to-N with debounce, hysteresis, and latching. *Flagged: assumed irreducible but not yet searched for prior art (task 2.6).*
4. **Song bundle + manifest contract** — the project-specific data model binding authored material to ES-9 outputs, banks, selection, and samples.
5. **Performance / Backstage surfaces** — the two role-specific views over one core.
6. **Bundle-to-card reconciliation** — which samples this song needs, above rclone and A8Manager.

Six items, three of which are data-model and UI rather than algorithmic. The genuinely novel engineering is items 1 and 2 — which is also where `decide-base-platform` already identified the only real technical risk, the ES-9 timing bench.

That is a good sign for smallness, and it is the register's most useful output.

## Consumption modes matter as much as licenses

License consequences differ by how a dependency is consumed:

- **Vendored / linked** — the license binds the project directly. Permissive only.
- **Invoked as a separate binary** — minimal obligation; a GPL tool is generally fine.
- **User-installed, project merely points at it** — no obligation at all.

The project already relies on this: VCV Rack and the GPL-3 Chinenual recorder are acceptable precisely because the user installs them and Maybelle sits downstream. A8Manager would sit in the same category. rclone would be invoked, not linked.

## Resolved 2026-08-21

### A8Manager has no license — this is decisive

Checked the repository directly rather than trusting secondary sources. **GitHub reports the license field as `null`, and there is no `LICENSE`, `LICENCE`, or `COPYING` file anywhere in the repository root** (contents: `.github`, `Source`, `test_data`, `.gitignore`, `A8Manager.jucer`, `README.md`).

Secondary coverage describes A8Manager as "free and open source." **The repository does not support that.** Source being publicly visible is not a license. With no license stated, default copyright applies — all rights reserved — and there is no grant to modify, redistribute, vendor, or build upon it.

Consequences, in order of impact:

1. **It cannot be a project dependency.** Not vendored, not wrapped, not invoked as a shipped component.
2. **Contributing a Linux build upstream is not the clean option it appeared to be.** A pull request to a repository with no license leaves the contribution's terms undefined for both sides. This would need the author to add a license first — a reasonable thing to ask, and worth asking, but it is a prerequisite rather than a plan.
3. **Pointing users at it remains completely fine**, which is what `research-synced-lfo-sampler-authoring` and the README already do. A user downloading a freeware tool for their own hardware is their business.

So the Assimil8or card-management gap is **not** closed for Maybelle's toolchain. The still-missing Linux build is now the second problem rather than the first.

The repository is actively maintained (last push April 2026, updated July 2026, 33 open issues), so a license may appear. Re-check before relying on this verdict.

**Practical note:** the Assimil8or's *constraints* — card layout, filename rules, accepted WAV formats — are facts about the hardware, not A8Manager's intellectual property. Source them from Rossum's own documentation and from the module, not by reading an unlicensed codebase.

### rclone is MIT and does exactly what was specified

Confirmed **MIT** via the repository's license field. Extremely healthy: ~59k stars, last pushed 2026-08-20.

`rclone copy` states it verbatim: **"Doesn't delete files from the destination. If you want to also delete files from destination, to make it match source, use the sync command instead."** That is precisely the additive-by-default semantics `research-sampler-sample-sync` specified. `--checksum` checks "for changes with size & checksum"; `--dry-run` performs a trial run; local filesystem paths are supported.

So the design that change specified is standard practice, independently arrived at — which is useful validation regardless of whether rclone is what ultimately implements it.

### But rclone may still be the wrong size for this job

Reuse-before-build is not reuse-at-any-size. The register's own risk section warns that a small app with forty dependencies is not small.

The actual task is copying a few dozen WAV files onto an SD card with content verification. In Python that is roughly a walk, a `hashlib` pass, a compare, a `shutil.copy2`, and a re-hash — stdlib only, no dependency at all. rclone is a cloud-storage tool; requiring users to install a large Go binary, or vendoring one per platform, to copy thirty samples locally is plausibly *less* small than the code it replaces.

The honest verdict is therefore split:

- **The design is validated** — rclone proves the specified model is the standard one, and `copy`-not-`sync` is the correct primitive.
- **The implementation is still open.** Task 3.4 should weigh rclone-installed against vendored against stdlib, with stdlib a serious contender precisely because it adds nothing.

This is the register working as intended: the research changed the plan, and it did not change it into "add a dependency."

## Outstanding

- ~~Confirm A8Manager's license~~ — **done: none.** Re-check periodically in case one is added.
- Decide whether to **ask the author to add a license**, which is the prerequisite for any deeper reuse or upstream contribution (task 3.2).
- ~~Confirm rclone's license~~ — **done: MIT.**
- Decide rclone-installed vs vendored vs stdlib for the reconciliation copy step (task 3.4). Stdlib is a serious contender.
- Source the Assimil8or's card and WAV constraints from Rossum documentation and the hardware, not from A8Manager's unlicensed source.
- Search prior art for the CV selection layer and the clock-following scheduler before accepting them as irreducible (tasks 2.6, 2.7).

**Sources:** [A8Manager repo](https://github.com/cpr2323/A8Manager) · [A8Manager site](https://cpr2323.github.io/a8manager/index.html) · [rclone copy](https://rclone.org/commands/rclone_copy/) · [rclone sync](https://rclone.org/commands/rclone_sync/) · [rclone docs](https://rclone.org/docs/) · [Chinenual-VCV](https://github.com/chinenual/Chinenual-VCV)
