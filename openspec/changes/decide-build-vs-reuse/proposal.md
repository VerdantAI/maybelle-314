## Why

Maybelle 314 should be **as small as possible**. Its distinctive job is narrow — follow the rack clock, read rack CV, and emit pitch CV, gates, triggers, and modulation through the ES-9 from stored sequences — but the research spikes have accumulated a much longer list of surrounding capabilities: MIDI parsing, audio I/O, schema validation, file synchronization, sampler card management, MIDI capture from VCV Rack, waveform generation, a web UI, and a CLI. Almost none of that is Maybelle's actual product.

Without an explicit position, each spike independently drifts toward specifying something to build. Two recent examples make the cost concrete: `research-sampler-sample-sync` specifies a sync engine with content hashing, dry-run diffing, and safety guarantees that `rclone` already provides and has hardened over years, and it does so for a device that already has an open-source card manager in **A8Manager**. `research-vcv-rack-companion-module` weighs writing a C++ Rack plugin when the MIDI capture it depends on is already solved by an existing module.

This change establishes the principle, records what already exists against each need, and — more usefully — names the short list of things that genuinely have no substitute and therefore *are* the product.

## What Changes

- Establish **"reuse before build"** as a stated project principle, with the burden of proof on building.
- Create a **build-vs-reuse register**: for each capability Maybelle needs, the candidate project, its license, its platform support, and a verdict of reuse, reuse-with-gaps, or build.
- Record the **existing projects already identified**, including their limitations rather than only their strengths:
  - **A8Manager** — open-source Assimil8or preset *and sample* manager that scans an SD card and offers to fix issues. Caveats: **no Linux build yet**, and its license is not stated in the repository README.
  - **rclone** — `copy` with `--checksum` and `--dry-run` supplies content-hash comparison, non-destructive preview, and additive-by-default semantics; `--max-delete` and `--backup-dir` cover the destructive cases.
  - **Chinenual MIDI Recorder** — already recommended for VCV Rack capture.
  - The Python stack already chosen in `decide-base-platform`: Mido, sounddevice/PortAudio, NumPy, FastAPI/Uvicorn.
- Name the **irreducible core** — the capabilities with no credible existing substitute, which is the only code the project must own.
- Define **evaluation criteria** for admitting a dependency: license compatibility, platform coverage including Linux and the Pi, maintenance health, and the cost of the gap left behind.
- Define what happens when a candidate is **close but not sufficient**: wrap it, contribute upstream, or build — and how that choice is made rather than defaulted.
- Re-scope `research-sampler-sample-sync` in light of A8Manager and rclone, so it specifies only the reconciliation Maybelle uniquely needs.
- Do not add, vendor, or integrate any dependency in this change.

## Capabilities

### New Capabilities

- `build-vs-reuse-register`: Defines the project's reuse-before-build principle, the register of candidate projects against each capability Maybelle needs, the criteria for admitting a dependency, and the irreducible core the project must own.

### Modified Capabilities

- None.

## Impact

- Adds OpenSpec planning artifacts establishing a cross-cutting project principle and its supporting register.
- **Re-scopes `research-sampler-sample-sync`**, whose sync-engine and card-management requirements substantially overlap rclone and A8Manager. What remains Maybelle-specific is bundle-to-card *reconciliation* — determining which samples a song needs — not the sync mechanics or the sampler's own card conventions.
- **Informs `research-vcv-rack-companion-module`**, reinforcing its preliminary lean away from building a C++ plugin.
- Consumes `decide-base-platform`, which already chose a permissive Python stack; this change makes the reasoning behind that stack explicit and reusable for future decisions.
- Applies to every future spike: proposals should state what was considered for reuse before specifying something to build.
- No production code, runtime dependencies, or hardware integration are introduced by this proposal.
