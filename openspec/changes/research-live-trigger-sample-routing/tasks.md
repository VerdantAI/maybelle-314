## 1. Trigger Source Research

- [ ] 1.1 Inventory rack-level sources including gates, triggers, CV, audio pulses, Pamela's Pro Workout outputs, and ES-9 input paths.
- [ ] 1.2 Inventory controller sources including USB MIDI devices, DIN MIDI adapters, BeatStep/Keystep-style controllers, MIDI notes, CC, program changes, and transport messages.
- [ ] 1.3 Inventory venue and software sources including OSC, network cues, DAW cues, MIDI Show Control, MIDI Time Code, LTC/SMPTE, and application-specific cue outputs.
- [ ] 1.4 Record connection path, signal/protocol constraints, timing expectations, and relevance for each source.

## 2. Routing Target Research

- [ ] 2.1 Compare ES-9 output routing for CV, gate, trigger, and patch-selection actions.
- [ ] 2.2 Compare external sampler control options, including future Assimil8or workflows and non-MIDI trigger paths.
- [ ] 2.3 Compare sending MIDI, OSC, or network cue messages to DAWs, cue players, visual tools, and show-control systems.
- [ ] 2.4 Evaluate whether Pi-based sample playback belongs in scope or should remain deferred.

## 3. Live Software Surface Research

- [ ] 3.1 Research audio cue playback and theater/show-control tools and their integration protocols.
- [ ] 3.2 Research VJ, visual effects, interactive media, and lighting/show-control tools and their integration protocols.
- [ ] 3.3 Research modular-friendly protocol bridges and patch-control utilities relevant to CV/MIDI/OSC workflows.
- [ ] 3.4 Summarize common protocols and formats by software category rather than recommending a product prematurely.

## 4. Safety and Configuration Review

- [ ] 4.1 Define trigger qualification needs: edge detection, debounce, hysteresis, latching, arming, safe mode, and rate limits.
- [ ] 4.2 Identify live failure modes: duplicate triggers, missed triggers, unstable selection, wrong cue/sample, unavailable targets, and disconnected outputs.
- [ ] 4.3 Define mapping metadata needed for live trigger workflows, including labels, input calibration, output assignment, target identity, sample/cue identity, and dry-run behavior.
- [ ] 4.4 Identify how live trigger mappings should be validated against ES-9 profiles, song bundles, channel banks, and operator display state.

## 5. Recommendation

- [ ] 5.1 Recommend which live trigger workflows should be supported first, deferred, or excluded from Maybelle's core scope.
- [ ] 5.2 Recommend whether Maybelle should trigger external samplers, emit cues to other software, play/route samples directly, or combine these models.
- [ ] 5.3 Feed findings into `decide-base-platform`, `investigate-es9-config-profiles`, `investigate-pi-port-topology`, `investigate-agent-control-surface`, and future Assimil8or research.

## 6. Verification

- [ ] 6.1 Review the research output against every `live-trigger-sample-routing-research` requirement.
- [ ] 6.2 Run OpenSpec validation or status checks for the completed change.
