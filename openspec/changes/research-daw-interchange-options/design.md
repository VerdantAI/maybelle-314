## Context

Maybelle 314 is planned as a Raspberry Pi 5 stored-sequence MIDI/CV controller. It follows Pamela's Pro Workout as the master clock, receives rack-provided CV runtime parameters, and outputs CV/gates/triggers/modulation through the ES-9. The Pi is not intended to generate rack voices or act as a sampler.

The authoring workflow is expected to happen on a separate computer, with tracks authored first in Ardour and loaded onto the Pi. The unresolved question is whether the runtime should consume Ardour session files directly, consume exported MIDI/stems, consume an open DAW interchange format such as DAWproject, or use a Maybelle-specific song bundle generated from DAW exports. The research must also determine whether that bundle contract can be DAW-agnostic enough to later support Ableton Live, Bitwig Studio, Studio One, Logic Pro, Reaper, and other major DAWs.

Initial research suggests:
- Ardour distributes ready-to-run builds for Linux, macOS, and Windows, but source builds are explicitly described by Ardour as challenging and unsupported by the project for end users. Running Ardour itself on the Pi should therefore be treated as a spike, not an assumption.
- Ardour stem export is an intentional interchange path. Ardour documents stem export as exporting each track individually while preserving sync, but losing all data except actual audio/MIDI.
- Ardour audio export supports formats such as BWAV 24-bit, BWAV 32-float, FLAC, MP3, Ogg/Vorbis, and WAV-tagged outputs.
- DAWproject is an MIT-licensed open exchange format based on ZIP and XML. It covers note data, automation, audio data, tempo, time signature, plug-in state, and track/timeline structure, but it is not currently listed as an Ardour-supported export format.
- DAWproject is especially relevant to cross-DAW abstraction because Bitwig and Studio One are associated with the format, but DAW support coverage must be verified instead of assumed.
- Existing hardware sequencers provide useful precedents. Squarp Hapax advertises easy MIDI file import/export, OXI One MKII advertises MIDI pattern import and long MIDI file playback from SD card, and Squarp Hermod+ records/playbacks CV/Gate and MIDI patterns with SD-card project recall.
- Pi audio distributions such as Patchbox OS and Zynthian show that Raspberry Pi can support serious audio/MIDI workflows, but they solve broader instrument/DAW problems than Maybelle needs.

## Goals / Non-Goals

**Goals:**
- Define a repeatable research process for DAW/session interchange options.
- Compare direct Ardour-on-Pi usage, Ardour export bundles, DAWproject, Standard MIDI Files, and Maybelle-specific manifests.
- Determine whether Maybelle can define a DAW-agnostic song-bundle contract, with Ardour as the first producer and other DAWs as future producers.
- Compare major DAWs by the artifacts they can export for Maybelle: Standard MIDI Files, stems, markers, tempo maps, automation data, DAWproject, project XML, or other machine-readable formats.
- Identify existing modules, sequencers, or Pi audio projects that demonstrate relevant workflow patterns.
- Produce decision evidence for the base-platform and song-bundle decisions.
- Keep the runtime contract focused on clock-following MIDI/CV event playback unless evidence justifies broader DAW/session support.

**Non-Goals:**
- Implement an Ardour parser, DAWproject parser, or song bundle converter.
- Implement importers for Ableton, Bitwig, Studio One, Logic, Reaper, or other DAWs.
- Require Ardour to run on the Pi.
- Support full DAW session playback in the runtime.
- Choose an audio/CV sample workflow for Assimil8or or other samplers.

## Decisions

### Treat Ardour Session Files as Research Input, Not the Runtime Contract

The research will investigate Ardour session structure, but the runtime will not assume native `.ardour` session files are the primary interchange format unless the spike proves the format is stable, practical, and sufficiently documented for Maybelle's needs.

Rationale: Ardour sessions carry DAW state that Maybelle does not need, including editing, routing, plug-ins, and audio workflow details. The runtime needs deterministic beat-positioned control events and channel mappings.

Alternatives considered:
- Directly parse Ardour sessions: may preserve rich authoring context, but risks coupling Maybelle to a DAW-native format.
- Run Ardour headless or embedded on the Pi: powerful, but likely excessive for a stored-sequence CV controller.

### Treat Exported MIDI and Metadata as the Baseline Candidate

The first interchange candidate will be a Maybelle song bundle built from Ardour-exported Standard MIDI Files plus a manifest for track/channel/output mappings, bank structure, quantization behavior, and runtime metadata.

Rationale: MIDI is the closest match for clock-following pitch/gate/controller events, and a Maybelle manifest can express rack-specific concepts that DAW formats do not model directly.

Alternatives considered:
- MIDI files only: simple, but does not encode rack mappings, bank selection, ES-9 output calibration, or runtime CV parameter behavior.
- Audio/CV stems only: useful for fixed rendered modulation, but not suitable as the primary contract for tempo-following control events.

### Separate the Runtime Bundle from DAW-Specific Producers

The research will evaluate a two-layer model: DAW-specific exporters or conventions produce a Maybelle song bundle, and the Pi runtime consumes only the normalized bundle.

Rationale: the runtime should not need to know whether a sequence was authored in Ardour, Ableton, Bitwig, or another DAW. DAW-specific differences belong in authoring conventions, export scripts, or companion tools.

Alternatives considered:
- Build the runtime around Ardour-specific files: fastest for the first workflow, but makes later DAW support more expensive.
- Support every DAW format directly in the runtime: flexible on paper, but increases parsing risk and runtime complexity.

### Evaluate DAWproject as an Optional Authoring Import Path

The research will evaluate DAWproject as an optional source format or future converter target, not as the required first format.

Rationale: DAWproject is open, MIT licensed, XML/ZIP based, and explicitly designed to preserve richer DAW timeline data than Standard MIDI Files. However, Ardour support is currently unproven, so it may be useful for future DAW compatibility rather than the first Ardour workflow.

Alternatives considered:
- Ignore DAWproject: simpler, but misses a relevant open standard.
- Make DAWproject mandatory: premature without Ardour support and converter validation.

### Use Existing Sequencers as Workflow References

The research will document patterns from existing hardware/software sequencers rather than trying to clone them.

Rationale: Hapax, OXI One, Hermod+, Zynthian, and Patchbox OS demonstrate useful decisions around SD-card projects, MIDI import, CV/Gate recording, clock synchronization, and Pi audio setup. These are precedents for Maybelle's workflow and constraints.

Alternatives considered:
- Treat Maybelle as a greenfield invention: flexible, but likely to miss known UX and timing patterns.
- Require feature parity with an existing sequencer: too broad for the current product.

## Risks / Trade-offs

- Ardour export behavior does not preserve enough event data -> Mitigation: create controlled Ardour sessions and inspect exported MIDI/stem artifacts.
- DAW-agnostic abstraction becomes too vague -> Mitigation: define the normalized Maybelle bundle fields first, then test each DAW export path against those fields.
- Direct Ardour session parsing looks easy but proves brittle -> Mitigation: treat it as optional research until validated against multiple sessions.
- DAWproject looks attractive but has incomplete DAW coverage -> Mitigation: record it as a candidate producer format only where support is verified.
- Hardware sequencer research over-expands scope -> Mitigation: limit findings to import/export, clocking, SD-card workflow, and CV/Gate sequencing patterns.
- Pi DAW projects bias us toward running a full DAW -> Mitigation: compare only the pieces relevant to Maybelle's runtime: file interchange, timing, and I/O.

## Migration Plan

No runtime migration is required because this change adds planning artifacts only. Research output should feed back into `decide-base-platform` and a later song-bundle-format proposal.

Rollback is limited to removing this OpenSpec change before implementation.

## Open Questions

- Can Ardour export the exact MIDI event types Maybelle needs: notes, CC automation, pitch bend, markers, tempo map, time signatures, track names, and channel metadata?
- Which major DAWs can export enough data to produce the normalized Maybelle song bundle without manual reconstruction?
- Can Ableton Live, Bitwig Studio, Studio One, Logic Pro, Reaper, or other DAWs export markers, tempo maps, track names, MIDI automation, and stems in machine-readable ways?
- Does Ardour expose enough stable XML/session structure to make direct session inspection useful for authoring-time conversion?
- Is there a practical Ardour-to-DAWproject path, or would DAWproject only help if the authoring DAW changes later?
- Which existing hardware sequencer workflow is closest to Maybelle's intended SD-card song-bank/channel-bank model?
- Should Maybelle eventually provide a companion authoring/export app on the Ardour host?
