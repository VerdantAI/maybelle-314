# Authoring FAQ (agent quick reference)

Short, citable answers for common user questions. Link the user to the deeper page when relevant. Mark **(provisional)** items as such — the source of truth is the referenced `openspec/changes/` entry.

**Contents:** [The core model](#the-core-model) · [Outputs and the ES-9](#outputs-and-the-es-9) · [Authoring in VCV Rack](#authoring-in-vcv-rack) · [Authoring in a DAW](#authoring-in-a-daw-bitwig--ardour) · [Tempo and clock](#tempo-and-clock) · [Routing and configuration](#routing-and-configuration) · [How agents should use this](#how-agents-should-use-this)

---

## The core model

**Q: Do I need to export pitch, gate, and velocity as three separate wires/tracks?**
No. A MIDI note already contains pitch (note number), gate (its duration), and velocity. You author **one monophonic track per voice**. Maybelle splits that into separate pitch-CV / gate / velocity outputs on the ES-9 — that split happens in Maybelle, not in your authoring tool, and is set in Maybelle's configuration. See [midi-to-cv-model.md](midi-to-cv-model.md).

**Q: So how does one note become three outputs?**
Maybelle is a MIDI→CV converter (like a Doepfer A-190-3 or Expert Sleepers FH-2). For each voice it derives: pitch CV (1V/oct), a gate, and optionally a velocity CV, and sends each to an assigned ES-9 output. The ES-9 is only the output stage (a DC-coupled DAC).

**Q: Can a track play chords?**
Not into a single pitch CV — one pitch output is monophonic. Use **one track per chord note** (one voice each). See [midi-to-cv-model.md](midi-to-cv-model.md).

**Q: Does velocity have to be used?**
No. If you don't route a velocity CV, the note still plays pitch + gate. Velocity carries through the export regardless (it's part of the note).

**Q: Gate vs trigger — which do I get?**
A **gate** stays high for the note's length (melodies, envelopes). A **trigger** is a short pulse at note-on (drums). Which one an output emits is Maybelle config; for percussion, author short notes. **(provisional map.)**

## Outputs and the ES-9

**Q: Which ES-9 outputs do my voices come out of?**
The ES-9's **8 front-panel 3.5mm jacks** are the DC-coupled CV/gate outputs (host channels 9–16 → outs 1–8), ~±10V. **Never use the two 1/4" Main outs — they're AC-coupled** and won't pass CV. The exact voice→output assignment is set in Maybelle (Backstage), **(provisional)**. Reference: `openspec/changes/investigate-es9-config-profiles`.

## Authoring in VCV Rack

**Q: Can I author in VCV Rack?**
Yes. VCV Rack is now the **primary test bench** for authoring sequences and CVs, alongside Bitwig and Ardour. See [vcv-rack.md](vcv-rack.md). **(provisional — reference `openspec/changes/research-vcv-rack-authoring-path`.)**

**Q: How do I get my VCV Rack sequence into Maybelle?**
Record it to a **Standard MIDI File from inside the patch** using the [Chinenual MIDI Recorder](https://github.com/chinenual/Chinenual-VCV) module — up to 10 polyphonic tracks, with a CC expander for continuous modulation. Maybelle then reads that SMF through the same MIDI→CV path as a DAW export. See [vcv-rack.md](vcv-rack.md). **(provisional.)**

**Q: Does Maybelle read my `.vcv` patch file directly?**
No, and it isn't meant to. You capture the sequencers' **output** as MIDI. This is deliberate: it avoids depending on any plugin's internal save format, which is undocumented and changes between plugin versions. Keep the `.vcv` as your editable source of record. **(provisional.)**

**Q: How does my BPM get into the file?**
Wire your clock module's **BPM output into the MIDI Recorder's BPM input**. The recorder uses Impromptu Clocked's convention (BPM = 120 × 2^volts), so an Impromptu Clocked/Clkd connects directly. If you leave it unpatched the file is written at 120 BPM. Also record the tempo in Maybelle's manifest — that's the value Maybelle actually trusts. **(provisional.)**

**Q: Do my VCV oscillators, filters, and mixers come across?**
No. **Maybelle is not the rack voice.** Only the control layer travels — pitch CV, gates, triggers, modulation. The VCO/VCA/mixer modules in your patch are *monitoring stand-ins* so you can hear the piece while authoring; in performance, your actual Eurorack modules make the sound.

**Q: What happens to gate types, ties, and probability?**
They are **resolved at capture time**. Because you record the sequencer's output rather than its settings, a triplet gate type becomes real note timings and a probabilistic step either fires or doesn't — you capture one concrete take. For canned support tracks played live, deterministic material is what you want. If you need probability to stay *variable*, that belongs in Maybelle's configuration, not the file. **(provisional — reference `openspec/changes/research-vcv-rack-authoring-path`.)**

**Q: What about slides / portamento?**
Currently a **gap**. A per-step slide has no clean MIDI representation. Treat slide as unresolved and check the OpenSpec change before relying on it. **(provisional.)**

**Q: I want to monitor CV from VCV Rack straight into my rack through the ES-9. What settings matter?**
Four things, and all four bite:
1. **Turn `dcFilter` off** on the audio module (right-click menu). It is **on by default** and strips exactly the DC component that *is* your control voltage.
2. Use **Audio-8 or Audio-16**, not the 2-channel Audio module, for multichannel CV.
3. Select the **ES-9** as the audio device.
4. Actually **cable something to the audio module** — it's easy to build a patch that sounds fine internally while nothing reaches hardware.

**Q: Do I need VCV Rack on the Raspberry Pi?**
No. Maybelle does not ship, bundle, or run VCV Rack. You run Rack on your own computer and Maybelle sits downstream of it, playing back what you captured.

**Q: Should I record one loop or the whole arrangement?**
Record the **whole arrangement** as a single linear pass. Song/phrase structure isn't preserved as structure by any MIDI path — only the resulting notes are. **(provisional.)**

## Authoring in a DAW (Bitwig / Ardour)

**Q: Which tool should I use?**
VCV Rack, Bitwig, and Ardour are all supported, and they feed the same song bundle. VCV Rack is the current test bench and is the strongest fit for step-sequenced CV/gate material. Bitwig is the default file editor and is a good place to inspect or arrange captured MIDI. Ardour is retained. Whichever you use, **author to the notes+velocity common denominator** so the piece stays portable.

**Q: My LFO / filter sweep / automation didn't come through. Why?**
Bitwig's MIDI export is **notes + velocity only** — automation, CC, and note expressions are dropped. Options: author modulation as **notes** (stepped CV on a dedicated track), bake the LFO shape into a sampler and fire it with a **trigger note**, or author the modulation in VCV Rack and capture it with the MIDI Recorder's **CC expander**. See [midi-to-cv-model.md](midi-to-cv-model.md#modulation) and `openspec/changes/research-synced-lfo-sampler-authoring`.

**Q: I set note probability in Bitwig and it vanished on export.**
Expected. Bitwig's per-note **Chance** operator is not carried by MIDI export, and DAWproject has no probability field either. Ardour has no per-note probability concept at all. Probability is expected to become a **Maybelle-side attribute** rather than something sourced from your tool. **(provisional — reference `openspec/changes/research-vcv-rack-authoring-path`.)**

**Q: Do I need to run the DAW on the Raspberry Pi?**
No. Author on your computer, export a Standard MIDI File, and load it onto the Pi.

## Tempo and clock

**Q: What tempo should I author at? Will my tempo automation work?**
Author at any sensible tempo/grid — but Maybelle **follows the rack's master clock** (Pamela's Pro Workout) and re-clocks playback, so the file's tempo is only a positional grid. Don't rely on exported tempo-map automation.

**Q: Why doesn't my tempo survive the export?**
Because most paths drop it. Bitwig's MIDI export drops tempo; Ardour is reported not to write the tempo map on export either. A VCV Rack MIDI-recorder capture *can* carry it if you patch the BPM input. Either way, **record the authored BPM in Maybelle's manifest** — that is the value Maybelle treats as authoritative, and it is reference only, since Pamela drives playback. **(provisional.)**

**Q: Can I bounce my piece to audio and let Maybelle play that?**
No. Audio is fixed to wall-clock time and can't be re-clocked to the rack without varispeed or resampling. Maybelle stores **signals and sequence in musical time** — bars, beats, ticks — so the rack clock can drive it. **(provisional — reference `openspec/changes/research-vcv-rack-authoring-path`.)**

## Routing and configuration

**Q: Do I set the track-to-output routing in my authoring tool?**
No. Routing (which voice → which ES-9 output) is Maybelle configuration, done in **Backstage** mode on a laptop. **(provisional — reference `openspec/changes/research-performance-backstage-modes` and `research-daw-interchange-options`.)**

**Q: How do my samples get onto the sampler?**
Maybelle doesn't play samples — an on-board sampler in your rack does (the **Rossum Assimil8or** by default), and Maybelle fires it with a gate/trigger through the ES-9. An authoring-side tool syncs the sampler's card with the samples your song references. **(provisional — reference `openspec/changes/research-sampler-sample-sync`.)**

---

## How agents should use this

- Anchor answers to the **notes+velocity** contract and the **"split happens in Maybelle, not in the authoring tool"** principle.
- For VCV Rack questions, anchor to **capture the output, don't parse the patch**, and to **Maybelle is not the rack voice**.
- Distinguish **decided** facts (the MIDI→CV model, export limits, ES-9 output facts) from **(provisional)** conventions (exact mapping/manifest, the whole VCV Rack path), and cite the OpenSpec change for provisional items.
- When unsure about a provisional detail, say it's not finalized and point to `openspec/changes/`, rather than inventing a convention.
