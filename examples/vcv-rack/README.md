# VCV Rack example patches

Test fixtures for the authoring-path research. Their inventory and analysis live in
`openspec/changes/research-vcv-rack-authoring-path/research-findings.md`.

## `prog-riff-v1.vcv`

A 7-step riff at 137 BPM. VCV Rack 2.6.6.

| | |
| --- | --- |
| Sequencers | Impromptu Phrase-Seq-16, Gate-Seq-64 (both `pulsesPerStep: 6`) |
| Clock | Impromptu Clocked-Clkd, 137 BPM, `ppqn: 4` |
| Voices | Fundamental VCO, VCMixer; NANOModules OCTA |
| Output | Core AudioInterface2 |
| Pitch (seq 0) | +0, +3, +5, +6, +3, +5, +0 semitones |

**This patch cannot drive CV or be captured as-is.** Four blockers, all documented in
`research-findings.md` §1.7:

1. `dcFilter: true` on the audio module — strips the DC component that *is* the control voltage.
2. `AudioInterface2` is the 2-channel module; multichannel CV needs `Audio-8`/`Audio-16`.
3. The audio device is a consumer soundbar, not the ES-9.
4. **Nothing is cabled to the audio module at all** — the VCMixer output goes nowhere.

It is a valid fixture for *file-format* work (container, `patch.json`, module identity,
bit-packed attributes, tempo location) and it has served that purpose. It is not a valid
fixture for signal-path or capture work.

## TODO: rebuild as `prog-riff-v2.vcv`

Needed before the bench tasks in `research-vcv-rack-authoring-path` task groups 2 and 3
can run. Keep the musical content identical so v1 and v2 stay comparable.

Add the [Chinenual MIDI Recorder](https://github.com/chinenual/Chinenual-VCV) (now installed) and:

- [ ] Wire **Clocked BPM out → MIDI Recorder BPM in**, so the authored 137 BPM lands in the
      SMF tempo meta-event. The recorder uses Impromptu Clocked's convention
      (BPM = 120 × 2^volts), so this connects directly.
- [ ] Route **one voice per recorder track**: PhraseSeq16 V/OCT + gate 1, PhraseSeq16 gate 2,
      and the two GateSeq64 channels.
- [ ] Add the **MIDI RecorderCC expander** if any continuous modulation is in play.
- [ ] Enable **"Start at first note gate"** so events align to a bar boundary.
- [ ] Record the **full 4-phrase arrangement** as one linear pass, not a single loop.

Optionally, for monitoring CV straight into the rack through the ES-9 (a separate concern
from MIDI capture — either may be built as its own patch):

- [ ] Replace `AudioInterface2` with **`Audio-8`** or **`Audio-16`**.
- [ ] Turn **`dcFilter` off** (right-click menu).
- [ ] Select the **ES-9** as the audio device.
- [ ] Actually cable the CV/gate outputs to it.

Note that the VCO, OCTA, and VCMixer are **monitoring stand-ins** — they exist so the piece
can be heard while authoring and are not part of what reaches the rack. See
`docs/authoring/vcv-rack.md`.
