# VCV Rack Companion Module — Research Findings

**Status:** Effort, licensing, and distribution researched 2026-08-20 from VCV's own documentation. Format definition, alternatives scoring, and the recommendation remain open.

## Summary

Building a Rack module is **cheaper than expected on every axis except one**: it is C++ in a project that is otherwise Python, and it is distributed and maintained inside someone else's ecosystem. Licensing, tooling, build matrix, submission, and migration cost all come out favourably. The open question is not *can we* but *should we*, and the honest comparison is against a standalone Python converter that closes most of the same gap for none of the new surface.

## Licensing — permissive is explicitly allowed

VCV grants a **Non-Commercial Plugin License Exception**. Plugins may use "open-source licenses like BSD 3-clause or MIT," closed-source freeware, or donationware — **provided the plugin is free**. GPLv3+ is also available and is what Rack itself uses.

Two conditions attach, and both are easy to lose by accident:

1. **Charging forfeits it.** For-profit plugins under non-GPLv3 terms "require contacting VCV at support@vcvrack.com for commercial royalty licensing."
2. **Incorporating Rack source forfeits it.** "If you incorporate substantial Rack code into your plugin, you must license it under GPLv3."

So an **MIT-licensed, free Maybelle module is permitted** and satisfies the project's MIT-compatible constraint, as long as it stays free and the code stays original. Task 5.3 should record the exception's exact wording rather than this summary before anything is committed.

VCV Library distribution additionally requires compliance with the **Plugin Ethics Guidelines**, which prohibit cloning "the brand name, model name, logo, panel design, or layout of components…of an existing hardware or software product without permission." Not a constraint for an original Maybelle panel, but it is binding.

### Boundary note

`research-vcv-rack-authoring-path` settled that the project does not ship, bundle, host, or redistribute VCV Rack. **Shipping a plugin does not violate that** — the user still installs and runs their own Rack. But it is genuinely new surface: the project would be distributing a binary inside a third-party ecosystem and tracking a third-party release cycle. Worth deciding rather than drifting into.

## Build effort — moderate, well-trodden

| | |
| --- | --- |
| Language | C++11 |
| Against | The Rack SDK (download, build from any folder) |
| Scaffolding | `helper.py` generates a template and C++ boilerplate **from an SVG panel**, auto-detecting params, inputs, outputs, lights, and custom widgets |
| Manifest | `plugin.json` — slug, version, license, author, URLs |
| Core code | A `process()` function "called every audio frame" |
| Registration | Each model registered in `plugin.hpp` and `plugin.cpp` |
| Build | `make`, `make dist`, `make install` |

Notably, the panel SVG is not decoration — it is **input to the code generator**, so panel design is on the critical path rather than a finishing step.

**File I/O from a module is proven**, not speculative: the Chinenual MIDI Recorder writes Standard MIDI Files and VCV Recorder writes audio and video. An export module is squarely within what modules already do.

VCV's own framing is that "creating Rack plugins is a great way to learn programming and C++," which suggests the API is approachable. For an import/export module the DSP burden is near zero — the hard part of most plugins does not apply here.

## Distribution — lighter than expected

**Platform targets:** Windows x64, Mac x64, **Mac ARM64**, Linux x64.

Linux ARM64 is **absent from the SDK**. That does not matter here — this is authoring-side tooling that never runs on the Pi. It is the same gap noted in `research-vcv-rack-authoring-path`, and it is equally irrelevant for the same reason.

**One machine builds them all.** The **VCV Rack Plugin Toolchain** builds "for all architectures with one command," natively on Linux or via Docker. No per-platform CI matrix is required.

**Submission is a GitHub issue.** For open-source plugins: create one issue titled with the plugin slug and provide the source URL. To release an update, bump the version in the manifest, commit, and comment in the thread with the new version and **commit hash** (not a branch name); a maintainer processes it. The issue thread is "your permanent communication channel with VCV Library maintainers."

Closed-source or commercial plugins go through email instead. Not applicable if the module is free and MIT.

## Maintenance — real but historically small

- **Major Rack releases make incompatible API and ABI changes.** Plugins must be updated and recompiled.
- **Minor releases only add symbols** — backward compatible with plugins built against older minor versions, but not forward compatible.
- **Historical evidence:** the v1→v2 migration required, for roughly **90% of plugins, only a version bump and a recompile** — described as a one-line edit. The remaining 10%, using advanced or unstable API, needed a few search-and-replace steps.

So the recurring cost is small and concentrated at major releases. The real commitment is not effort — it is that a **published module becomes something users depend on**, and letting it break after a Rack release is a visible failure. Publishing should imply intent to maintain.

## The scoping question this all hangs on

MIDI capture is **already solved** by the Chinenual recorder. A Maybelle module must justify itself on what it does *around* the notes:

- voice → ES-9 output binding
- track naming that makes the output map legible
- authored BPM recorded as reference
- validation at authoring time, before material reaches the rack
- sample references for `research-sampler-sample-sync`
- bundle packaging rather than a bare SMF

That is the **manifest gap**, and it is genuine — today it is hand-assembled in Backstage with nothing checking it against the captured MIDI.

## The comparison that actually decides it

Every item in that list can be produced **outside Rack**, by a standalone converter that takes the captured SMF plus a small authoring-side config and emits a validated bundle. That approach:

- reuses the project's existing **Python core** and the manifest's JSON Schema validation
- adds **no C++**, no SDK, no four-target build matrix, no library submission, no Rack API tracking
- is worse only in **user experience** — two steps instead of one, and the output binding is declared in a config rather than read from the patch

The decisive question is therefore narrow: **what can a module read from the patch that a user would otherwise have to declare anyway?** If the voice→output binding has to be stated by hand regardless of where it is stated, the module's advantage shrinks to convenience, and convenience is not obviously worth a cross-platform binary and a public support commitment.

There is also a **sequencing constraint**: `decide-song-bundle-manifest` is not settled, and this research will likely extend it. Building a C++ exporter against a moving format means re-releasing a binary through a third-party library every time the format shifts. The manifest should be settled first.

The two options are **complementary, not exclusive** — a converter can ship now against the same contract a module would later wrap.

## Preliminary lean

**Defer the module; build the converter first.** Not because the module is hard — the research says it is not — but because it is the more expensive way to close a gap that is currently better closed in Python, against a format that is not yet stable. Revisit once the manifest is settled and the converter has shown what the in-patch experience would actually add.

This is a lean, not the recommendation; tasks 7.5–7.7 and 8.3 should score the alternatives properly before it is recorded as a decision.

**Sources:** [Plugin Licensing](https://vcvrack.com/manual/PluginLicensing) · [Building](https://vcvrack.com/manual/Building) · [Plugin Development Tutorial](https://vcvrack.com/manual/PluginDevelopmentTutorial) · [VCV Library submission](https://github.com/VCVRack/library) · [Versioning](https://vcvrack.com/manual/Version) · [Migrating v1 plugins to v2](https://vcvrack.com/manual/Migrate2) · [Chinenual-VCV](https://github.com/chinenual/Chinenual-VCV)
