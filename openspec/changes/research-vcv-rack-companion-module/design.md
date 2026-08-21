## Context

`research-vcv-rack-authoring-path` established that VCV Rack material reaches Maybelle as a Standard MIDI File, captured from inside the patch by the Chinenual MIDI Recorder. That path works and needs nothing from this project.

What it does not produce is the **manifest**. Maybelle's song bundle binds voices to ES-9 outputs, names tracks, carries the authored BPM as reference, configures rack-CV selection, and — per `research-sampler-sample-sync` — references sample assets. None of that is expressible in an SMF, so today the user captures MIDI in Rack and hand-assembles the rest in Backstage, with nothing checking that the two agree until the material is in the rack.

A Maybelle-authored Rack module could close that: export a complete, validated bundle from inside the patch, where the voice-to-output intent actually lives, and import one back for editing.

The counterweight is what it costs. The project's runtime is Python; the module would be **C++ against a third-party API**, built for four platform targets, distributed through someone else's library, and maintained across someone else's release cycle. That is a new kind of obligation, not just new code.

Research findings supporting this design are recorded in `research-findings.md`. Summarized:

- **Licensing is permissive-compatible.** VCV's Non-Commercial Plugin License Exception explicitly permits MIT or BSD licensing "provided plugins are free." Charging for it would require a commercial royalty arrangement with VCV, and incorporating substantial Rack source would force GPLv3.
- **Build effort is moderate and well-trodden.** C++11 against the Rack SDK, a `helper.py` scaffolder that generates boilerplate from an SVG panel, a `plugin.json` manifest, and a `process()` function per audio frame. The Plugin Toolchain builds all architectures from one command via Docker.
- **Platform targets are Windows x64, Mac x64, Mac ARM64, and Linux x64.** Linux ARM64 is absent from the SDK, which does not matter here — this is authoring-side only and never runs on the Pi.
- **Distribution is lightweight.** One GitHub issue per plugin with a source URL; updates are a version bump plus a comment naming the commit hash. Plugins must satisfy VCV's Plugin Ethics Guidelines.
- **Maintenance is real but historically small.** Major Rack releases make incompatible API/ABI changes; minor releases only add symbols. The v1→v2 migration required, for roughly 90% of plugins, a version bump and a recompile.
- **File I/O from a module is proven**, by the Chinenual recorder and VCV Recorder among others.

## Goals / Non-Goals

**Goals:**
- State honestly what a module would add beyond existing tooling, before estimating anything.
- Establish effort across build, distribution, and maintenance, separating a first working module from a releasable one.
- Establish the licensing position and its conditions.
- Define the formats on both sides, tied to the existing manifest contract rather than invented.
- Compare the module against cheaper alternatives on equal terms.
- Produce a build / defer / decline recommendation with the conditions that would change it.

**Non-Goals:**
- Implement a module, panel, converter, or bundle writer.
- Re-solve MIDI capture, which the Chinenual recorder already handles.
- Settle the song-bundle manifest format — that is `decide-song-bundle-manifest`.
- Run anything of this on the Pi. The module is authoring-side by definition.
- Commit the project to publishing and supporting a plugin. That is the decision this research informs.

## Decisions

### Frame this as "what closes the manifest gap," not "should we write a Rack module"

The gap is that authoring-side intent — which voice drives which ES-9 output — has no home in the captured artifact. A Rack module is one way to close it. Asking the narrow question first keeps the comparison honest, because most of the value on offer can be delivered without any C++ at all.

### The module's value is the manifest and the bundle, not the MIDI

MIDI capture is solved and re-solving it would be waste. Anything proposed here must be justified by what it does *around* the notes: output binding, track naming, BPM recorded as reference, validation at authoring time, sample references, and bundle packaging. A proposal that reduces to "our own MIDI recorder" should be rejected on those grounds.

### Do not build against an unsettled format

`decide-song-bundle-manifest` is not final, and this research will likely extend it further. Writing a C++ exporter against a moving target means churning a cross-platform binary artifact — with a library submission per release — every time the format shifts. **The manifest format should be settled before any module implementation starts.** This is a sequencing constraint, not an argument against the module.

### Treat the standalone Python converter as the leading alternative

The manifest gap can be closed entirely outside Rack: take the captured SMF plus a small authoring-side config, and emit a validated bundle. That approach reuses the project's existing Python core and the manifest's JSON Schema validation, requires no C++, no SDK, no four-target build matrix, no library submission, and no Rack API tracking. It is worse only in user experience — two steps instead of one, and the voice-to-output binding is stated in a config rather than read from the patch.

The research should therefore work hard to establish what the module gives that the converter cannot, rather than assuming the in-patch experience justifies the cost. The two are also **complementary**: a converter can exist now and a module can wrap the same contract later.

### Publishing a plugin is a support commitment

A module in the VCV Library becomes something users depend on and expect to keep working across Rack releases. The maintenance cost is historically small, but it is ongoing and it is public. The recommendation must state what the project is committing to, including the consequence of abandoning a published module — which is worse for users than never publishing one.

## Risks / Trade-offs

- **Building against an unsettled manifest** → Churn in a binary artifact distributed through a third party. Mitigate by sequencing: settle the format first.
- **New artifact class for the project** → First C++ code, first cross-platform binary matrix, first third-party distribution channel. Each carries setup and review cost the project has not paid before. Weigh honestly against a Python converter that adds none of them.
- **Abandonment risk** → A published module that stops working after a Rack release is a visible failure. Do not publish without an intent to maintain.
- **Licensing forfeiture** → The permissive position depends on the plugin being free and on not incorporating Rack source. Both are easy to lose accidentally. Record the conditions explicitly and keep original code original.
- **Scope creep into a sequencer** → A module with a panel invites feature requests. The scope is import and export of a bundle; sequencing, editing, and playback belong to Rack's own ecosystem and to Maybelle respectively.
- **Duplicating the Chinenual recorder** → Wasteful and slightly rude to a working upstream tool. If MIDI capture is needed inside the module, prefer interoperating with what exists over reimplementing it.
- **Third-party plugin dependency, again** → Recommending a workflow that requires a specific third-party module is a dependency the project does not control, the same class of risk identified for Impromptu's serialization. Record it even though it is milder here, since the recorder's output is a standard file format.

## Open Questions

- What can a module read from the patch that a user could not just as easily state in a config — and is that difference worth a C++ artifact?
- Can the voice-to-ES-9-output binding be inferred from patch structure at all, or must the user declare it regardless of where they declare it?
- On import, how much of a bundle can meaningfully be reconstructed as a patch, given that Maybelle bundles carry no oscillators, filters, or voices?
- Should the module interoperate with the Chinenual recorder rather than capture MIDI itself, and is that technically possible between separate plugins?
- Does the export belong in a module at all, or in Backstage — which already owns the mapping UI and already validates?
- If a converter ships first, does the module ever become worth building, or does the converter simply absorb the need?
