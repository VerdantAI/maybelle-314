## 1. Source Research

- [ ] 1.1 Research Ardour support on Raspberry Pi or ARM Linux, including official builds, distro packages, source-build feasibility, and user reports.
- [ ] 1.2 Research Raspberry Pi audio/DAW distributions and projects such as Patchbox OS and Zynthian for relevant audio/MIDI/runtime patterns.
- [ ] 1.3 Research DAW interchange formats, including DAWproject, Standard MIDI Files, Ardour stems, and Ardour session files.
- [ ] 1.4 Research major DAW export capabilities for Ableton Live, Bitwig Studio, Studio One, Logic Pro, Reaper, and other relevant authoring tools.
- [ ] 1.5 Research hardware sequencers/modules that import MIDI files, play SD-card projects, record CV/gate, or bridge DAW workflows into Eurorack.

## 2. DAW Export Spike

- [ ] 2.1 Create minimal Ardour and Bitwig test sessions with notes, velocity, CC automation, pitch bend, markers, tempo changes, time-signature changes, and named tracks.
- [ ] 2.2 Export MIDI (and, for Ardour, stem) artifacts from each session.
- [ ] 2.3 Inspect exported artifacts and record which event types and metadata each DAW preserves; confirm the Bitwig notes+velocity-only behavior against the installed version and check whether any newer version restores CC export.
- [ ] 2.4 Compare exported artifacts against Maybelle runtime needs for clock-following MIDI/CV playback, and note where the two DAWs diverge (the common denominator is notes + velocity).

## 3. Format Comparison

- [ ] 3.1 Compare Ardour session files, Standard MIDI Files, stem exports, DAWproject, and a Maybelle-specific manifest bundle.
- [ ] 3.2 Record implementation complexity, license status, stability, DAW support, and runtime fit for each format.
- [ ] 3.3 Identify the minimum song-bundle contract needed for song banks, channel banks, ES-9 mappings, calibration, and runtime CV selection.
- [ ] 3.4 Define which bundle fields are runtime-native and which fields are authoring-source-specific.
- [ ] 3.5 Compare whether Ardour, Ableton Live, Bitwig Studio, Studio One, Logic Pro, and Reaper can produce those bundle fields through standard exports, DAWproject, or custom conversion.

## 4. Decision Handoff

- [ ] 4.1 Recommend which DAW/export artifacts should be first-class Maybelle inputs.
- [ ] 4.2 Recommend whether first implementation should be Ardour-only, Ardour-first with a DAW-agnostic bundle, or multi-DAW from the start.
- [ ] 4.3 List follow-up spikes required before implementing import or conversion code.
- [ ] 4.4 Feed relevant findings back into `decide-base-platform` and the future song-bundle-format proposal.

## 5. Verification

- [ ] 5.1 Review the research output against every `daw-interchange-research` requirement.
- [ ] 5.2 Run OpenSpec validation or status checks for the completed change.
