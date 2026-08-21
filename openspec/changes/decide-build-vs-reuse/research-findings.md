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
| Assimil8or preset + sample card management | **A8Manager** | **unstated** (JUCE-based) | user-installed | **Reuse-with-gaps** — see below |
| File sync engine + safety | **rclone `copy`** | MIT (confirm) | separate binary | **Reuse-with-gaps** — see below |
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

## Outstanding

- Confirm A8Manager's license from the repository (task 3.1).
- Determine whether to contribute a Linux build to A8Manager upstream (task 3.2).
- Confirm rclone's license and decide on installed-vs-vendored (tasks 3.3, 3.4).
- Search prior art for the CV selection layer and the clock-following scheduler before accepting them as irreducible (tasks 2.6, 2.7).

**Sources:** [A8Manager repo](https://github.com/cpr2323/A8Manager) · [A8Manager site](https://cpr2323.github.io/a8manager/index.html) · [rclone copy](https://rclone.org/commands/rclone_copy/) · [rclone sync](https://rclone.org/commands/rclone_sync/) · [rclone docs](https://rclone.org/docs/) · [Chinenual-VCV](https://github.com/chinenual/Chinenual-VCV)
