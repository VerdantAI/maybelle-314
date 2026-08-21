# Authoring in VCV Rack

> **Status: provisional.** VCV Rack is the current primary test bench, but the capture path is still being researched. The source of truth is `openspec/changes/research-vcv-rack-authoring-path`. Verified findings live in that change's `research-findings.md`.

## The short version

**Author your sequence in VCV Rack, then record its *output* to a Standard MIDI File from inside the patch.** Maybelle reads that SMF through the same MIDI→CV path as a DAW export. Your `.vcv` patch stays as your editable source of record; Maybelle never parses it.

## What travels, and what doesn't

**Maybelle is not the rack voice.** Only the control layer travels to the rack:

| Travels | Stays in VCV Rack |
| --- | --- |
| Pitch CV (1V/oct) | Oscillators |
| Gates and triggers | Filters, VCAs, effects |
| Continuous modulation (as CC) | Mixers |
| The authored BPM, as reference | Anything that makes sound |

The VCO/VCA/mixer modules in your patch are **monitoring stand-ins** so you can hear the piece while authoring. In performance your actual Eurorack modules make the sound, driven by the CV and gates Maybelle emits through the ES-9.

Design your patch with that split in mind: the sequencers and modulation sources are the deliverable, the voices are scaffolding.

## Why capture output instead of reading the patch

A `.vcv` file is a save format, not an interchange format. Inspecting a real patch showed:

- Per-step gate types, ties, probability, and slides are **bit-packed into single integers** with no published schema, inside a third-party plugin's own state.
- Tempo has **no patch-level field** — it lives as an unnamed numeric parameter on whichever clock module you happen to use.
- The patch embeds **machine-specific hardware bindings** (audio device name, driver index) that won't resolve on another computer.
- Module ports and parameters are referenced by **integer index**, resolvable only by the plugin that defines them.

All of that is version- and plugin-coupled and would break on a plugin update. Capturing the sequencers' output sidesteps every bit of it: a running plugin resolves its own gate types and ties into plain note timings.

## Setup

### 1. Install the MIDI Recorder

Install the [Chinenual MIDI Recorder](https://github.com/chinenual/Chinenual-VCV) plugin (GPL-3.0). It records a VCV performance to a Standard MIDI File — up to **10 polyphonic tracks**, with a **MIDI RecorderCC** expander for capturing CV as 7-bit or 14-bit CC.

You install this yourself, on your own machine. Maybelle ships nothing and runs nothing of VCV Rack's.

### 2. Patch the BPM

Wire your clock module's **BPM output → the MIDI Recorder's BPM input.**

The recorder uses Impromptu Clocked's convention (BPM = 120 × 2^volts), so an Impromptu **Clocked** or **Clkd** connects directly. Leave it unpatched and the file is written at 120 BPM regardless of what you authored.

Also record the tempo in Maybelle's manifest. That is the value Maybelle treats as authoritative — and it is **reference only**, because Pamela's Pro Workout drives playback tempo. See [the tempo section of the FAQ](faq.md#tempo-and-clock).

### 3. Route one voice per track

Each row of recorder inputs is one MIDI track. Feed each voice's **V/OCT** and **GATE** into its own row. Percussion and trigger patterns get their own rows too — a gate with no pitch is fine.

Keep it to one monophonic voice per track, matching the [MIDI→CV model](midi-to-cv-model.md). Chords need one track per note.

### 4. Route modulation through the CC expander

Continuous modulation goes through the **MIDI RecorderCC** expander as CC values. This is the one clear advantage over Bitwig, whose MIDI export drops CC entirely.

### 5. Record the whole arrangement

Run the **full arrangement** once and capture it as a single linear pass — not one loop of one sequence. No MIDI path preserves phrase/sequence structure *as structure*; only the resulting notes survive.

The recorder's **"Start at first note gate"** option aligns the captured events to the beginning of a bar.

## What gets resolved at capture time

Because you record output rather than settings, several things become concrete:

- **Gate types** — a triplet or subdivided gate becomes real note on/off timings. The *abstraction* is lost; the *timing* is preserved exactly.
- **Probability** — a probabilistic step either fires or it doesn't. You capture one take.
- **Ties** — become longer notes.

For canned support tracks played live this is usually what you want: deterministic material that behaves the same every night. If you need probability to stay **variable** at performance time, that belongs in Maybelle's configuration, not in the captured file.

## Known gaps

- **Slide / portamento** has no clean MIDI representation. Unresolved — check the OpenSpec change before relying on it.
- **Gate-type abstraction** is not preserved anywhere, only its resolved edges.
- **Track → ES-9 output binding** is not expressed by any authoring tool. It is Maybelle configuration, edited in Backstage.
- **Phrase/song structure** is flattened by the capture.

## If you also want to monitor CV straight into your rack

You can point VCV Rack at the ES-9 directly to hear the patch in the rack while authoring. Four settings matter, and all four commonly bite:

1. **Turn `dcFilter` off** on the audio module (right-click menu). It is **on by default** and strips exactly the DC component that *is* your control voltage.
2. Use **Audio-8** or **Audio-16** — the 2-channel Audio module won't carry multichannel CV.
3. Select the **ES-9** as the audio device.
4. **Cable something to the audio module.** It is easy to build a patch that sounds correct internally while nothing reaches hardware.

Remember the ES-9's **8 front-panel 3.5mm jacks** are the DC-coupled outputs. **Never use the two 1/4" Main outs** — they are AC-coupled and will not pass CV.

## Related

- [midi-to-cv-model.md](midi-to-cv-model.md) — the conceptual model, tool-agnostic
- [faq.md](faq.md#authoring-in-vcv-rack) — quick answers
- [bitwig.md](bitwig.md) — good for inspecting or arranging captured MIDI
- `openspec/changes/research-vcv-rack-authoring-path` — the source of truth for everything provisional here
