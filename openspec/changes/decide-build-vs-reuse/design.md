## Context

Maybelle's actual job is narrow: follow the rack clock, read rack CV for selection, and emit pitch CV, gates, triggers, and modulation through the ES-9 from stored sequences. The Pi is not the rack voice and is not a sampler.

The research spikes have nevertheless accumulated a long tail of surrounding capabilities — MIDI parsing, multichannel DC audio I/O, schema validation, file synchronization, sampler card management, MIDI capture from VCV Rack, waveform generation, a web UI, a CLI. Each spike, reasoning locally, tends to specify something to build. Nothing was pushing back.

Two examples show the cost. `research-sampler-sample-sync` specifies content hashing, dry-run diffing, additive-by-default semantics, and post-write verification — a description of `rclone copy --checksum --dry-run` written from scratch, for a device that already has an open-source card manager. `research-vcv-rack-companion-module` weighs a C++ Rack plugin whose central dependency, MIDI capture, is already solved by an existing module.

`decide-base-platform` already made the right call implicitly by choosing Mido, sounddevice/PortAudio, NumPy, and FastAPI rather than writing any of them. This change makes that reasoning explicit and reusable.

## Goals / Non-Goals

**Goals:**
- State reuse-before-build as a principle with the burden of proof on building.
- Record what exists against each capability, limitations included.
- Name the irreducible core — the only code the project must own.
- Give future spikes a consistent test to apply before specifying implementation work.
- Re-scope the changes already drafted against what the register found.

**Non-Goals:**
- Add, vendor, pin, or integrate any dependency.
- Re-open `decide-base-platform`'s stack choices, which the register endorses.
- Choose the sampler sync implementation — that stays in `research-sampler-sample-sync`, re-scoped.
- Turn the register into a general software directory. It covers capabilities Maybelle actually needs.

## Decisions

### The burden of proof sits on building, not on reusing

A proposal that specifies building something is incomplete until it records what was considered for reuse and why it was rejected. This inverts the usual default, deliberately. The failure mode this guards against is not a bad decision to build — it is *never making the decision*, and arriving at an implementation plan without anyone having asked whether the work was necessary.

### Record limitations with equal prominence

A register that lists only what candidates can do is worse than no register, because it invites planning against tools that will not actually work here. **A8Manager** is the immediate case: genuinely open source, genuinely does sample *and* preset management, genuinely scans a card and offers fixes — and has **no Linux build**, with **no license stated in its README**. On a Fedora authoring machine, today, that is not a solution. It is a strong candidate with two blocking questions.

### Distinguish how a dependency is consumed

License consequences differ sharply between vendoring a library, invoking a separate binary, and pointing the user at something to install themselves. The project already relies on this distinction — VCV Rack and the GPL-3 Chinenual recorder are fine precisely because the user installs them and Maybelle sits downstream. The register records the consumption mode, not just the license, because that is what determines whether a GPL tool is a problem.

### `rclone copy`, not `rclone sync`

`sync` deletes destination files absent from the source — exactly the mirror-delete that `research-sampler-sample-sync` identified as unacceptable against a performer's card. `copy` is additive and never deletes, which is that change's stated default behaviour, already implemented. With `--checksum` for content comparison, `--dry-run` for the mandatory preview, and `--max-delete` and `--backup-dir` for the cases that do remove files, the safety model that change specified is largely off-the-shelf.

What remains Maybelle-specific is **reconciliation**: determining which samples a given song bundle requires, resolving them to concrete files, and reporting drift in terms the user understands. That is a small program that emits a file list, not a sync engine.

### Keep the core honest and short

The value of naming the irreducible core is that everything outside it becomes a candidate for reuse by default. The core should be defended, not padded: a capability belongs there only when the search genuinely came up empty, and "no exact match" is not the same as "nothing to reuse."

### The register is maintained, not written once

It is a living artifact. New spikes add rows; candidates change license, gain platforms, or go unmaintained. A row is a snapshot with a date, not a permanent verdict.

## Risks / Trade-offs

- **Reuse dogma producing a worse product** → The principle is reuse-*before*-build, not reuse-instead-of-build. Building is legitimate when the alternatives are worse; the requirement is to have compared, not to always defer.
- **Dependency sprawl** → Reusing everything is its own failure. Admission criteria and the consumption-mode distinction exist to keep dependencies deliberate; a small app with forty dependencies is not small.
- **Planning against unusable tools** → A8Manager's missing Linux build and unstated license are exactly this hazard. Mitigated by recording limitations with equal prominence and by refusing to mark a candidate "reuse" while a blocking question is open.
- **Stale register** → Verdicts rot. Date each row and treat it as a snapshot.
- **Upstream abandonment** → Reused projects can go unmaintained, and the ones here are small-team or single-maintainer efforts. Prefer standard file formats at the boundary so a dead dependency is replaceable rather than load-bearing.
- **Losing the plot on what the product is** → The mitigation is the register's most useful output: a short, explicit statement of the irreducible core.

## Open Questions

- What license is A8Manager actually under? The README does not say and it is JUCE-based, which usually implies GPLv3 for a free application.
- Will A8Manager gain a Linux build? Its author notes it "should be relatively simple" with JUCE, which makes contributing upstream a real option worth weighing against building.
- Is rclone's license confirmed MIT, and does invoking it as a separate binary raise any obligation at all?
- Should Maybelle depend on rclone being installed, vendor a static binary, or implement the small subset it needs directly?
- Does A8Manager's card validation overlap enough with the manifest's pass/warn/fail model to be invoked rather than reimplemented?
- Which core items are genuinely irreducible versus merely unsearched? The clock-following scheduler and the MIDI→CV engine look irreducible; the CV selection layer's quantize/debounce/hysteresis/latch behaviour may have prior art worth finding.
