# VCV Rack Authoring Path — Research Findings

**Status:** In progress. Section 1 executed against a real patch on 2026-08-20; sections 2–7 outstanding.
**Sample under test:** `examples/vcv-rack/prog-riff-v1.vcv` (2,656 bytes), authored in VCV Rack **2.6.6**.

> Note: the file is at `examples/vcv-rack/`, not `docs/examples/vcvrack/`.

## 1. Patch File Inventory — executed

### 1.1 Container format — **confirmed**

`.vcv` is a **zstd-compressed POSIX tar archive**. This confirms the hypothesis recorded in `design.md`; it is no longer an assumption.

```
prog-riff-v1.vcv  →  zstd  →  tar  →  patch.json
```

The 2,656-byte file expands to a 60 KB tar holding a 56 KB `patch.json`. No binary blobs, no per-module asset directories in this patch — but the archive is a plain tar and can carry arbitrary additional files (see 1.6).

### 1.2 `patch.json` structure — **confirmed**

Top-level keys: `version`, `unsaved`, `zoom`, `gridOffset`, `modules`, `cables`, `masterModuleId`.

**There is no patch-level tempo, time signature, or transport field.** This is the single most consequential structural finding — see 1.5.

Each module carries `id`, `plugin`, `model`, `version`, `params`, `pos`, and optionally `data`:

| plugin | model | version | has `data` |
| --- | --- | --- | --- |
| Fundamental | VCO | 2.6.4 | no |
| Fundamental | VCMixer | 2.6.4 | yes |
| Core | AudioInterface2 | 2.6.6 | yes |
| ImpromptuModular | Phrase-Seq-16 | 2.5.0 | yes |
| ImpromptuModular | Clocked-Clkd | 2.5.0 | yes |
| ImpromptuModular | Gate-Seq-64 | 2.5.0 | yes |
| NANOModules | OCTA | 2.3.9 | no |

Module identity is `plugin` + `model` + `version` — a usable three-part key. Two of the four plugins are third-party (**ImpromptuModular**, **NANOModules**); `Fundamental` and `Core` ship with Rack.

`cables` is a flat list of `outputModuleId`/`outputId` → `inputModuleId`/`inputId`. Ports are referenced by **integer index**, not name, so interpreting any cable requires knowing the module's port ordering — which lives in the plugin, not the patch.

`params` are `{id, value}` pairs. **Parameter identity is a bare integer index with no name and no unit.**

### 1.3 Where sequence data lives — **located, schema not decoded**

Per-step data lives inside each module's own `data` object:

**Phrase-Seq-16**
- `cv` — flat array of **256 floats** (16 sequences × 16 steps), directly readable as 1V/oct volts.
- `attributes` — flat array of **256 ints**, **bit-packed**, undocumented.
- `sequences` — per-sequence lengths; `phrase`/`phrases` — song-mode arrangement.
- `pulsesPerStep: 6` — advanced gate mode is active.

**Gate-Seq-64**
- `attributes2` — flat array of **2048 ints** (32 sequences × 64 steps), bit-packed.
- `sequences`, `phrase2`, `phrases`, `pulsesPerStep: 6`.

Decoded sequence 0 of the Phrase-Seq-16 (length **7**):

| step | cv (V) | semitones | attributes |
| --- | --- | --- | --- |
| 0 | +0.0000 | +0 | 677 |
| 1 | +0.2500 | +3 | 4261 |
| 2 | +0.4167 | +5 | 685 |
| 3 | +0.5000 | +6 | 4261 |
| 4 | +0.2500 | +3 | 677 |
| 5 | +0.4167 | +5 | 1605 |
| 6 | +0.0000 | +0 | 516 |

**Pitch CV extracts cleanly and unambiguously.** It is plain 1V/oct floats; no plugin knowledge is needed.

**Gate data does not.** Gate on/off, gate type, probability, tie, and slide are all bit-packed into one integer per step with no published schema. Decoding requires reverse-engineering against the Impromptu Modular source, and the layout can change between plugin versions. This is the concrete form of the plugin-coupling risk `design.md` flagged as a hypothesis — it is now measured.

**Positive finding for the musical-time requirement:** the sequencer data is inherently **tempo-relative**. Steps are positions on a grid advanced by clock pulses (`pulsesPerStep: 6`), not timestamps. There is no wall-clock or sample position anywhere in the sequence data. A transcription path would produce musical-time material *by construction*, satisfying the precondition without conversion.

### 1.4 Third-party plugin dependency — **confirmed as material**

All musical content in this patch lives in third-party plugins. `Fundamental`/`Core` modules (VCO, VCMixer, AudioInterface2) carry no sequence data at all. Any transcription path is therefore **entirely** dependent on ImpromptuModular's serialization for this patch — there is no vendor-neutral fallback within the file.

Missing-plugin behavior is **not yet tested** (task 1.4 needs a Rack install lacking the plugins).

### 1.5 Tempo representation — **confirmed, and it is a problem**

The authored BPM the project wants to carry as reference metadata is **137**. It is stored as:

```
modules[Clocked-Clkd].params[3].value = 137.0
```

That is: an **unnamed integer-indexed parameter on a third-party clock module.** There is no patch-level tempo field. Consequences:

- Extracting "the original BPM as reference" requires knowing that Impromptu's Clocked-Clkd exposes BPM at param index 3 — plugin-specific, version-specific, undocumented knowledge.
- A patch using a different clock module stores its tempo somewhere else entirely, under a different index.
- A patch with no clock module has no tempo at all.

**Recommendation for the manifest:** the authored BPM should be recorded as an explicit manifest field supplied at capture time, not scraped from patch internals. Scraping is a per-clock-module special case that will not generalize, and it fails silently — a wrong param index yields a plausible number, not an error.

Related clock state on the same module: `ppqn: 4`, `bpmDetectionMode: false`, `clockMaster: -1`, `running: true`. The patch's clock is internal master and is **not** configured to follow an external source — expected for authoring, and precisely the authority that transfers to Pamela's Pro Workout at playback.

### 1.6 Portability hazards — **four found, all machine-specific**

The `Core AudioInterface2` module serializes a **hardware binding into the patch**:

```json
{"audio": {"driver": 2, "deviceName": "Razer Leviathan V2 X",
           "sampleRate": 48000.0, "blockSize": 256,
           "inputOffset": 0, "outputOffset": 0},
 "dcFilter": true}
```

1. **`deviceName` is a literal device string** for the authoring machine's hardware. It will not resolve elsewhere.
2. **`driver` is a bare integer index** into Rack's driver list. The archive contains a stray backup (`patch.json.codex-alsa-backup-20260706084114`) showing the same patch previously at `driver: 1` with device `"Razer Leviathan V2 X (USB Audio)"` — i.e. **the index and the device string both shifted** across a driver change on the same machine. Driver indices are not stable identifiers.
3. **A stray non-patch file shipped inside the `.vcv` archive.** The tar carries that backup file alongside `patch.json`. Whatever produced it, the practical finding is that a `.vcv` can contain arbitrary extra files, so any importer must read `patch.json` by name and ignore the rest rather than assuming a single-entry archive.
4. **`dcFilter: true`** — see 1.7.

### 1.7 The patch is not currently configured for CV output — **four blockers**

Relevant to the bench setup (task group 2), the example patch cannot drive the ES-9 with CV as saved:

- **`dcFilter: true`.** The DC-blocking filter is **on**, which strips exactly the DC component that *is* the control voltage. This must be off for any CV use.
- **`AudioInterface2`** is the 2-channel (stereo) audio module. Multichannel CV to the ES-9 needs `Audio-8`/`Audio-16`.
- **Device is a consumer soundbar**, not the ES-9.
- **Nothing is cabled to the audio interface at all.** Tracing the 11 cables: `Clocked → PhraseSeq16/GateSeq64` (clock+reset), `PhraseSeq16 out0 → VCO` (pitch CV), `PhraseSeq16 out1 → VCMixer`, `GateSeq64 out0/out1 → OCTA`, `VCO/OCTA → VCMixer`. **The VCMixer's output is not patched to `AudioInterface2`.** No signal reaches hardware.

This is an internal audio-making patch (sequencers → VCO → mixer), not a CV-export patch. That is a reasonable first test file, but the bench setup in task group 2 starts from a different patch topology, and the reference bench configuration (2.5) will need to be built rather than derived from this one.

**Follow-up (task 2.6):** the patch is to be rebuilt as `prog-riff-v2.vcv` with the Chinenual MIDI Recorder wired in, keeping the musical content identical so the two remain comparable. The rebuild checklist lives in `examples/vcv-rack/README.md`. This blocks the bench work in task groups 2 and 3.

## Implications for the capture-path comparison

- **Transcription is more tractable than expected for pitch, and less for gates.** Pitch CV is plain 1V/oct floats requiring no plugin knowledge. Gate type, probability, tie, and slide are bit-packed integers requiring reverse-engineering per plugin per version. A path that captures pitch structurally and gates behaviorally may beat either extreme.
- **The musical-time precondition is satisfied natively by the sequencer data.** Steps and pulses-per-step, no timestamps. This meaningfully strengthens transcription and weakens any argument for wall-clock capture.
- **BPM must be captured explicitly, not scraped.** See 1.5.
- **`.vcv` is confirmed as a version-and-machine-coupled save format, not an interchange format.** It embeds a hardware device binding, unstable driver indices, integer port and param references resolvable only by the plugin, and undocumented per-plugin state. It is fine as a *source of record* for provenance; it is a poor *contract*. This supports the `design.md` decision to treat the patch as source and the captured artifact as the runtime input.
- **The MIDI path (task 3.2) remains the highest-leverage unknown** and is untouched by these findings.

## Outstanding in section 1

- **1.4** — open the patch with plugins missing; record failure behavior and surviving data. Needs a Rack install without ImpromptuModular/NANOModules.
- **1.5 (stability)** — whether Impromptu's `attributes` bit layout is stable across plugin versions. Needs comparison across two plugin releases.
- **1.6** — whether `.vcv` is documented anywhere as an interchange format, and what pinning or vendoring would be required.

## 2. Do the same data problems affect Ardour and Bitwig? — executed 2026-08-20

**No — and the distinction matters more than the answer.** The three sources fail in two fundamentally different ways.

- **VCV Rack has an *extraction* problem.** The data exists in the file; the format is undocumented, bit-packed, and coupled to a specific plugin version. Solvable with engineering effort, brittle across upgrades.
- **The DAWs have a *representation* problem — but only for probability.** For gate and tie there is no problem at all; MIDI expresses both natively and, in some respects, better than VCV does. For probability the data does not exist in any interchange format the DAWs can emit. No amount of parsing effort fixes that.

Extraction problems are engineering. Representation problems require a different format or a project-side convention.

### Per-attribute comparison

| Attribute | VCV Rack (`.vcv`) | Ardour (SMF) | Bitwig (SMF) |
| --- | --- | --- | --- |
| Pitch | **Easy** — plain 1V/oct floats | **Easy** — MIDI note number | **Easy** |
| Gate on/off + length | **Hard** — bit-packed | **Easy** — note on/off duration | **Easy** |
| Tie | **Hard** — bit-packed flag | **Easy** — a longer note | **Easy** |
| Gate type / subdivision | **Hard** — 12 symbolic types, bit-packed | **Easy** as notes, abstraction lost | **Easy** as notes, abstraction lost |
| Velocity | n/a (CV domain) | **Easy** | **Easy** |
| **Probability** | **Hard but present** — bit-packed | **Absent** — not an Ardour concept | **Absent from export** — authored, then dropped |
| Continuous modulation | Native | CC (Ardour exports more than Bitwig) | **Dropped on export** |
| Tempo / BPM | **Hard** — unnamed plugin param | **Dropped on export** (corrected in §3) | **Dropped on export** |
| Machine bindings | **Present** — device name + driver index | **None** | **None** |
| Format stability | Version + plugin coupled | Documented standard | Documented standard |

### Gate and tie are *easier* from the DAWs than from VCV Rack

This inverts the expected direction. In PhraseSeq16, gate presence, gate type, and tie are three bit-fields inside an undocumented integer. In MIDI they are the note itself: **duration is the gate**, and **a tie is just a longer note**. Both come free from Ardour and Bitwig with no plugin-specific decoding.

What is lost is the *abstraction*, not the *information*. PhraseSeq16's "this step is a triplet gate type" becomes, in MIDI, three short notes inside one step. For Maybelle that is an acceptable trade — Maybelle emits gates into a rack; it has no need for the step-and-gate-type model, only for the resulting edges. The one thing to pin down is a convention for overlapping/legato notes and whether Maybelle retriggers the gate, which is a Maybelle-side decision for the manifest rather than a source limitation.

### Probability is the one real casualty, and the DAWs are *worse* than VCV Rack

Ranking, best to worst:

1. **VCV Rack** — probability is in the file. Hard to decode, but recoverable.
2. **Bitwig** — probability exists in the editor as the per-note **Chance** operator, but `File > Export MIDI` carries notes and velocity only, so it is dropped. Authored but unexportable.
3. **Ardour** — no per-note probability concept at all. Nothing to lose.

**DAWproject does not rescue this.** It was the obvious escape hatch for Bitwig — MIT-licensed, XML, and far richer than SMF. But its `Note` class defines only `time`, `duration`, `channel`, `key`, `velocity`, `releaseVelocity`, and `content` (per-note expression timelines). **There is no probability or chance field.** Bitwig's own interchange format does not carry Bitwig's own probability data.

*(Worth noting on the positive side: DAWproject's `time` and `duration` are positions on the parent timeline — musical time — which satisfies the project's musical-time precondition natively.)*

### Recommendation: the manifest should own probability

Since no authoring source can deliver probability through an interchange format — VCV Rack only via brittle reverse-engineering, Bitwig not at all on export, Ardour not at all — probability should be a **Maybelle-side per-step attribute in the manifest**, authored or edited in Backstage rather than sourced from the authoring tool.

This is the only approach that works across all three sources, it is the only approach that works for Ardour at all, and it removes the strongest single motivation for decoding Impromptu's bit-packed `attributes` field. If probability is the main thing transcription would have bought, transcription gets substantially less attractive.

### Two inversions worth carrying forward

- **Tempo is weak everywhere.** ~~BPM is a standard SMF tempo meta-event in Ardour.~~ **Corrected in §3:** Ardour is reported not to write the tempo map on MIDI export either, so all three DAW-side paths lose tempo. Only VCV Rack with a MIDI-recorder module writes the authored BPM automatically. This reinforces §1.5: capture BPM as an explicit manifest field regardless of source.
- **Portability hazards vanish for the DAW path.** An SMF embeds no device name, no driver index, and no integer port references resolvable only by a plugin. Every portability hazard in §1.6 is VCV-specific.

### Outstanding

- Confirm Ardour's exact MIDI export event coverage on a bench — still listed as uninventoried in the README open questions and in `research-daw-interchange-options` task 2.1.
- Confirm the Bitwig version in use still drops probability and tempo on export, and whether any newer release changes this.
- Decide whether the gate-type abstraction is worth preserving anywhere, or whether resolved gate edges are sufficient for every source.

**Sources:** [DAWproject `Note.java`](https://github.com/bitwig/dawproject/blob/main/src/main/java/com/bitwig/dawproject/timeline/Note.java) · [DAWproject FAQ](https://www.bitwig.com/support/technical_support/dawproject-file-format-faqs-62/) · [Bitwig user guide — Operators/Chance](https://www.bitwig.com/userguide/latest/note_fx/) · [Ardour manual — MIDI](https://manual.ardour.org/working-with-midi/)

## 3. Best authoring tool to get the sample piece into the rack intact — executed 2026-08-20

### First: what "intact" can and cannot mean

`prog-riff-v1.vcv` contains seven modules, but **only the control layer travels to the rack.** Maybelle is not the rack voice. The Fundamental VCO, NANOModules OCTA, and VCMixer in this patch are *monitoring stand-ins* for the user's actual Eurorack voices — they exist so the piece can be heard while authoring. They are not part of the deliverable and no authoring tool needs to export them.

What must survive, derived from the patch:

| # | Element | Source in patch |
| --- | --- | --- |
| 1 | 137 BPM as reference | `Clocked-Clkd.params[3]` |
| 2 | Pitch sequence, 7 steps | `Phrase-Seq-16.cv` (1V/oct) |
| 3 | Gate 1, with advanced gate types | `Phrase-Seq-16.attributes`, `pulsesPerStep: 6` |
| 4 | Gate 2 | same |
| 5 | Two gate channels → OCTA | `Gate-Seq-64.attributes2` |
| 6 | 4-phrase song arrangement | `phrases: 4` on both sequencers |
| 7 | Ties, slides, probability | bit-packed in `attributes` |

Judging tools against *that* list, not against "reproduce the patch," changes the answer.

### The finding that reframes everything: VCV Rack can write MIDI files directly

Task 3.2 asked whether MIDI can be captured out of VCV Rack. **Yes — and more cleanly than assumed.** The [Chinenual MIDI Recorder](https://github.com/chinenual/Chinenual-VCV) is a VCV Rack module that records a performance straight to a Standard MIDI File from inside the patch:

- **Up to 10 polyphonic tracks**, one per row of inputs (V/OCT, GATE, VEL, AFT, PW poly; MW mono).
- Converts CV **"in the same way the VCV core CV-MIDI module does."**
- A **MIDI RecorderCC expander** captures CV as CC values, **7-bit and 14-bit**.
- An option to **"Start at first note gate,"** aligning events to the beginning of a bar.
- Licensed **GPL-3.0**.

**The BPM problem from §1.5 is solved by a patch cable.** The recorder's BPM input "uses same conventions as Impromptu's CLOCKED BPM output (BPM = 120 × 2^voltage)" — and this patch already runs on Impromptu's Clocked-Clkd. Wiring Clocked's BPM output to the recorder's BPM input writes the authored 137 BPM into the SMF tempo meta-event. No param-index scraping, no plugin-specific special case.

**The bit-packed `attributes` problem from §1.3 evaporates.** Capturing the *output* of the sequencers means gate types, ties, and probability are resolved by the running plugin into concrete note on/off timings. Nothing needs decoding, and nothing is coupled to Impromptu's serialization or its version.

**Probability freezing is a feature here, not a loss.** A probabilistic step either fires or does not during capture, so one concrete take is recorded. For *canned support tracks* played back in performance, deterministic material is what the project wants. This also removes the strongest remaining reason to build transcription at all (see §2's probability recommendation).

The GPL-3.0 license is not a problem under the decided boundary: it is a Rack plugin the *user* installs on their own authoring machine, exactly like Rack itself. Maybelle ships nothing and sits downstream.

### Correction to §2: Ardour also drops tempo on MIDI export

§2's table listed Ardour tempo as "Easy — SMF tempo meta-event." That was wrong. Ardour users report on 7.4.0 that **the tempo map is not written to exported MIDI**, and it is a long-standing open request on the Ardour tracker. Ardour *does* export track and instrument names, and it reads tempo maps on import — but export is the weak direction.

So **all three DAW-side paths lose tempo**, and the VCV Rack + MIDI Recorder path is the only one of the four that puts the authored BPM into the file automatically. This strengthens rather than weakens §1.5's recommendation to carry BPM as an explicit manifest field: it is the one thing that works regardless of source.

### Ranked comparison for this piece

| Path | Verdict | Gaps to fill |
| --- | --- | --- |
| **A. VCV Rack + Chinenual MIDI Recorder → SMF** | **Recommended** | Verify capture timing accuracy vs. the 6 PPS grid; confirm 10 tracks covers the voice count; record the full 4-phrase arrangement rather than one loop pass; slide/portamento has no MIDI representation; confirm track naming for the ES-9 channel map |
| **B. VCV Rack → CV-MIDI → virtual port → DAW → SMF** | Viable fallback | Everything in A, plus MIDI port routing, Rack↔DAW clock sync, and then the receiving DAW's own export gaps |
| **C. Bitwig as primary author** | Not for this piece | Re-enter the riff by hand; no gate-type or step model; probability authored but dropped on export; tempo dropped; CC dropped. Better used as the *editor* for captured MIDI |
| **D. Ardour as primary author** | Not for this piece | No per-note probability concept at all; tempo map reportedly not exported; CC export unconfirmed. Strength: exports track/instrument names, which the ES-9 channel map can use |
| **E. Direct SMF generation (Python/mido)** | Niche but real | No authoring UI and not musician-facing, but deterministic and lossless. Worth keeping for fixtures, tests, and generated material, given the Python core |

### Recommended workflow for this piece

1. Author in **VCV Rack** — it is the only one of the tools with the step/gate-type/probability model the piece actually uses, and it is already the bench.
2. Add the **Chinenual MIDI Recorder**; wire **Clocked's BPM output to its BPM input** so 137 lands in the file.
3. Route each voice's V/OCT and GATE into its own recorder track — PhraseSeq16 pitch + gate 1 + gate 2, and the two GateSeq64 channels.
4. Use the **CC expander** for any continuous modulation.
5. Run the **full 4-phrase arrangement** once and capture it, rather than looping one sequence.
6. Optionally open the resulting SMF in **Bitwig** to inspect, trim, or arrange — Bitwig's export limits do not matter if it is not the export stage.
7. Maybelle consumes the SMF through the **existing MIDI→CV engine and manifest**, unchanged.

The strategic point: this path makes VCV Rack **just another producer of the artifact the DAWs already produce**. It needs no `.vcv` parser, no engine on the Pi, no Impromptu decoding, and no new runtime contract — and it satisfies the musical-time precondition natively, because an SMF is tempo-relative by construction.

### Gaps that remain for *every* path

- **Slide / portamento has no clean MIDI representation.** PhraseSeq16 carries it as a per-step attribute; MIDI has only pitch bend or CC. Confirm whether the piece uses slides, and decide whether Maybelle implements slide as a manifest-side per-step attribute (the same answer §2 reached for probability).
- **Gate-type abstraction is not preserved** anywhere — only its resolved edges. Confirm that is acceptable before committing.
- **Track → ES-9 output binding** is not expressed by any source. It lives in the manifest and is edited in Backstage, as `docs/authoring/` already states.
- **Song/phrase arrangement** must be captured as a linear pass; no MIDI path preserves the phrase/sequence structure as structure.

### Outstanding

- Bench the Chinenual MIDI Recorder against `prog-riff-v1.vcv`: timing accuracy, tempo write-through, track count, CC fidelity.
- Confirm whether the piece uses slides or ties, which requires either decoding `attributes` or inspecting the patch in Rack.
- Confirm Ardour's tempo-map export behavior on the installed version rather than relying on forum reports.

**Sources:** [Chinenual-VCV](https://github.com/chinenual/Chinenual-VCV) · [VCV CV to MIDI](https://library.vcvrack.com/Core/CV-MIDI) · [VCV Gate to MIDI](https://library.vcvrack.com/Core/CV-Gate) · [VCV Core manual](https://vcvrack.com/manual/Core) · [Entrian Sequencers](http://entrian.com/audio/entrian-sequencers.html) · [Ardour tempo-map export discussion](https://discourse.ardour.org/t/export-tempo-mapped-midi/104866) · [Ardour tracker #8457](https://tracker.ardour.org/view.php?id=8457)
