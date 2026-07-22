## Why

Maybelle 314 authors modulation in a DAW, but the supported DAWs cannot carry continuous modulation through MIDI export in a portable way (Bitwig's MIDI export is notes + velocity only). A tempo-synced LFO therefore needs a different path: author its shape in Ardour/Bitwig, bake that shape into a waveform stored in a sampler such as the Rossum Assimil8or, and let Maybelle emit only the *trigger timing* — which does survive MIDI export as a plain note. Before committing to this workflow or building any tooling, we need evidence about single-cycle/multi-bar waveform formats, Assimil8or preset structure and sync behavior, and the licensing boundary around existing Assimil8or configurator tools.

## What Changes

- Add a research spike for authoring tempo-synced LFO/modulation waveforms and preparing sampler presets, as offline authoring-side tooling separate from the Pi runtime.
- Research single-cycle (Adventure Kid AKWF-style) and multi-bar "bounce the automation curve" waveform formats, including WAV specifications (mono, 16-bit, 44.1 kHz, single-cycle length conventions) and their licensing.
- Research how the Assimil8or plays a stored waveform as an LFO: loop vs one-shot modes, gate/trigger-in retriggering, CV mapping (Pitch, Sample Start, Loop Start/Length), and how loop length plus clock division keep the waveform phase-locked to an external clock.
- Establish the division of labor: the LFO *shape* lives in the sampler; the LFO *trigger timing* is authored as a dedicated MIDI note track that Maybelle converts to a gate/trigger through the ES-9 into the sampler's trig-in.
- Determine the licensing boundary for wrapping vs pointing to existing Assimil8or configurators, and define a clean-room preset-writer approach for generated content.
- Do not implement waveform generation, WAV export, preset writing, or sampler integration in this change.

## Capabilities

### New Capabilities

- `synced-lfo-sampler-authoring-research`: Defines research requirements for authoring tempo-synced LFO waveforms, storing them in a sampler, triggering them from Maybelle, and handling configurator tooling and licensing.

### Modified Capabilities

- None.

## Impact

- Adds OpenSpec planning artifacts for the synced-LFO / sampler-content-tooling investigation.
- Informs the DAW interchange research (modulation that cannot ride through MIDI export), the song-bundle metadata design, ES-9 output/trigger allocation, and the deferred Assimil8or runtime spike.
- No production code, runtime dependencies, hardware integration, or bundled third-party code or media are introduced by this proposal.
