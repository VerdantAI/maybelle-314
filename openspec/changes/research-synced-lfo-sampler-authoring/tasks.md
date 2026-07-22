## 1. Waveform Format Research

- [ ] 1.1 Research single-cycle waveform formats (Adventure Kid AKWF), including WAV specifications, single-cycle length conventions, and sampler compatibility.
- [ ] 1.2 Research multi-bar "bounce the automation curve" waveforms: how Ardour/Bitwig render a synced modulation curve to audio, and the resulting file characteristics.
- [ ] 1.3 Record the licensing of candidate waveform sources (AKWF CC-BY 3.0 vs CC0) and the attribution obligations, and confirm self-generated waveforms carry none.

## 2. Assimil8or Playback and Sync Research

- [ ] 2.1 Research Assimil8or playback modes (one-shot vs gated/loop), envelope behavior, and how a stored waveform functions as an LFO.
- [ ] 2.2 Research the channel gate/trigger-in and CV-in mappings (Pitch, Sample Start, Loop Start/Length, etc.) relevant to LFO playback.
- [ ] 2.3 Research the clock-sync approach: loop-length-to-clock-division mapping, trig-in retriggering to correct drift, and Pitch-CV tempo tracking.
- [ ] 2.4 Define how Maybelle's trigger note maps to an ES-9 gate/trigger output feeding the sampler's trig-in, phase-locked to Pamela's clock.

## 3. Trigger-Authoring Model

- [ ] 3.1 Define the dedicated MIDI trigger-note track convention (which survives Ardour/Bitwig MIDI export) for LFO (re)trigger timing.
- [ ] 3.2 Compare against the alternative of encoding modulation as notes or as song-bundle metadata, and record when each is preferred.
- [ ] 3.3 Identify the song-bundle metadata fields needed to associate a trigger track, a stored waveform, and a sampler channel/output.

## 4. Configurator Tooling and Licensing

- [ ] 4.1 Inventory existing Assimil8or configurators (A8Manager and alternatives) and confirm their license status.
- [ ] 4.2 Decide the boundary: which tools are pointed-to/launched vs what Maybelle must implement itself.
- [ ] 4.3 Identify a clean-room reference (Rossum manual, documented format, or A8Manager-readable sample files) for a Maybelle-owned preset writer, with no reuse of reserved source.
- [ ] 4.4 Record required attribution/credits for pointed-to tools and any bundled waveform content.

## 5. Decision Handoff

- [ ] 5.1 Recommend the supported waveform flavors and the trigger-authoring convention.
- [ ] 5.2 Recommend the point-to-vs-clean-room split for Assimil8or preset preparation.
- [ ] 5.3 List follow-up spikes required before implementing waveform generation, preset writing, or ES-9 trigger emission.
- [ ] 5.4 Feed relevant findings back into the DAW interchange research, song-bundle metadata design, and the deferred Assimil8or runtime spike.

## 6. Verification

- [ ] 6.1 Review the research output against every `synced-lfo-sampler-authoring-research` requirement.
- [ ] 6.2 Run OpenSpec validation or status checks for the completed change.
