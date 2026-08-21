## 1. Scope Against Existing Tooling

- [x] 1.1 Record what the Chinenual MIDI Recorder and other existing Rack modules already provide. (MIDI capture is solved — 10 poly tracks, CC expander, BPM input. See `research-findings.md`.)
- [x] 1.2 State specifically what a Maybelle module would add beyond that. (The manifest: voice→ES-9 binding, track naming, BPM as reference, validation, sample references, bundle packaging.)
- [ ] 1.3 Confirm no other existing plugin already writes structured project metadata alongside MIDI.
- [ ] 1.4 Determine whether the module should interoperate with the Chinenual recorder rather than capture MIDI itself, and whether inter-plugin cooperation is technically possible.

## 2. Level of Effort — Build

- [x] 2.1 Record the implementation language, API version, SDK, and toolchain. (C++11 against the Rack SDK; `helper.py` scaffolds from an SVG panel.)
- [x] 2.2 Record the non-code obligations: panel artwork, `plugin.json` manifest, module state serialization.
- [ ] 2.3 Estimate effort for a first working module — a panel, an export button, and a bundle written to disk.
- [ ] 2.4 Estimate effort for a releasable module — validation, error reporting, import, and a panel that is not embarrassing.
- [ ] 2.5 Confirm how a module performs file I/O and file-dialog interaction, citing existing modules as precedent.

## 3. Level of Effort — Distribution

- [x] 3.1 Record the platforms and architectures to build. (Windows x64, Mac x64, Mac ARM64, Linux x64. Linux ARM64 absent from the SDK and irrelevant — authoring-side only.)
- [x] 3.2 Confirm whether one machine can build all targets. (Yes — the VCV Rack Plugin Toolchain, natively on Linux or via Docker.)
- [x] 3.3 Record the VCV Library submission process and update mechanism. (One GitHub issue per plugin with a source URL; updates are a version bump plus a comment naming the commit hash.)
- [ ] 3.4 Record the Plugin Ethics Guidelines requirements the module must satisfy, including naming and panel-design constraints.
- [ ] 3.5 Determine whether the project wants VCV Library distribution at all, or whether direct download suffices.

## 4. Level of Effort — Maintenance

- [x] 4.1 Record the API/ABI compatibility policy across major and minor Rack releases. (Major releases break; minor releases only add symbols — backward but not forward compatible.)
- [x] 4.2 Record the historical migration cost as evidence. (v1→v2: roughly 90% of plugins needed only a version bump and recompile.)
- [ ] 4.3 State what the project commits to by publishing, and the consequence of abandoning a module users depend on.
- [ ] 4.4 Estimate the annual maintenance cost in concrete terms.

## 5. Licensing

- [x] 5.1 Record permitted plugin licenses and their conditions. (VCV's Non-Commercial Plugin License Exception permits MIT/BSD **provided the plugin is free**.)
- [x] 5.2 Record what forfeits the permissive position. (Charging for it requires a commercial royalty arrangement with VCV; incorporating substantial Rack source forces GPLv3.)
- [ ] 5.3 Confirm the exact exception text against the current VCV licensing page rather than a summary, and record it verbatim.
- [ ] 5.4 Confirm the project's MIT-compatible constraint is satisfiable, and record the standing obligations.
- [ ] 5.5 Re-examine the distribution boundary from `research-vcv-rack-authoring-path`: shipping a plugin is not shipping Rack, but it is new surface inside a third-party ecosystem. Record whether this is consistent with the project's downstream position.

## 6. Formats

- [ ] 6.1 Define what the module writes toward Maybelle: note/gate data, manifest, bundle packaging, and sample references — each tied to `decide-song-bundle-manifest` rather than invented.
- [ ] 6.2 Define what the module reads when importing a bundle, and what it can and cannot reconstruct as a patch given that bundles carry no voices.
- [ ] 6.3 Record the Rack-side obligations: `plugin.json` fields, panel asset requirements, and how module state serializes into a patch.
- [ ] 6.4 Determine whether the voice→ES-9-output binding can be inferred from patch structure, or must be declared by the user regardless.

## 7. Alternatives

- [ ] 7.1 Evaluate **doing nothing** — capture MIDI, author the manifest in Backstage.
- [ ] 7.2 Evaluate a **standalone Python converter** — captured SMF plus an authoring-side config, emitting a validated bundle, reusing the existing core and JSON Schema validation.
- [ ] 7.3 Evaluate **contributing to an existing plugin**, noting that Chinenual is GPL-3.0 and that it is someone else's roadmap.
- [ ] 7.4 Evaluate **building the module**.
- [ ] 7.5 Score all four for effort, maintenance, licensing, user experience, and how much of the manifest gap each actually closes.
- [ ] 7.6 State which alternatives are complementary rather than exclusive, particularly whether a converter now and a module later share one contract.
- [ ] 7.7 Establish what the module gives that the converter cannot, and whether that difference justifies a C++ artifact.

## 8. Sequencing and Recommendation

- [ ] 8.1 State the risk of building against an unsettled song-bundle manifest format.
- [ ] 8.2 State what must be true before any module implementation starts.
- [ ] 8.3 Recommend build, defer, or decline, with reasons.
- [ ] 8.4 State the conditions that would change the recommendation.
- [ ] 8.5 If deferring, name what should be done instead in the meantime.
- [ ] 8.6 Determine whether bundle export belongs in a Rack module at all, or in Backstage, which already owns the mapping UI and validation.

## 9. Verification

- [ ] 9.1 Review the research output against every `vcv-rack-companion-module-research` requirement.
- [x] 9.2 Run `openspec validate` for this change. (passes, 2026-08-20)
