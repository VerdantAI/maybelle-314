# ES-9 Config-Profile Research Findings

**Type:** Web-research output (docs + reverse-reading the official config tool). No physical ES-9 or Pi was exercised.
**Retrieval date:** 2026-07-22
**Primary firmware/tool target:** ES-9 firmware **1.3.1** (16/3/2023) with **config tool 1.3.0**. This is the latest published line as of retrieval; the config-tool version tracks firmware minor version (1.3.x firmware ↔ tool 1.3.0).

Confidence note: Everything below is grounded in the official ES-9 firmware v1.3 User Manual and the product page. The manual documents the full SysEx surface but does **not** publish the byte-level field layout *inside* the configuration dump payload. Where this document infers structure from SysEx message shapes, it is flagged as **inferred** and must be confirmed against a real `.syx` dump on the bench. The ES-9's internal config encoding is otherwise sparsely documented — do not treat inferred byte offsets as ground truth.

---

## Requirement 1 — ES-9 configuration tool assessment

**Official tool.** A GUI "Configuration Tool" is provided per firmware version. Two delivery forms:
- **Standalone app** for macOS and Windows.
- **Platform-independent browser tool** (`webapps/es9_config_tool_1.3.html`) using the **Web MIDI API**. It works **only in Chromium-based browsers** (Chrome). Chrome may block SysEx when the page is served from a website, so the documented workaround is to **download the HTML and open it locally**. The tool shows `Web MIDI status: OK` when usable.

**Version ↔ firmware map** (from the firmware page):

| Firmware | Date | Config tool |
|---|---|---|
| 1.3.1 | 16/3/2023 | 1.3.0 |
| 1.3.0 | 14/12/2022 | 1.3.0 |
| 1.2.x | 2021–22 | 1.2.0 |
| 1.1.1 | 17/3/2020 | 1.1.0 |
| 1.0.1 | 16/9/2019 | 1.0.0 |

Tool download URLs: `https://www.expert-sleepers.co.uk/webapps/es9_config_tool_{1.0,1.1,1.2,1.3}.html`. **Config save/load was introduced in firmware 1.3.0**; earlier tools cannot round-trip a `.syx` config. A Maybelle validator therefore has a hard floor of **firmware ≥ 1.3.0**.

**Save/load & connection behavior.** The tool connects over MIDI, most conveniently the ES-9's own USB-C class-compliant MIDI port ("ES-9 MIDI Out/In"), or indirectly through the DIN breakout. Changes made in the tool are reflected in hardware **immediately** (live SysEx), independent of the flash slots. Config download is a browser link/button that saves a `.syx`; upload is `Choose file` → `Upload`.

**Configurable areas relevant to Maybelle:** input DC-blocking (per pair), routing (three DSP blocks), internal 8×8 mixer + optional second mixer, stereo links, input EQ, mix smoothing, per-output DC offsets, MIDI-channel assignment for mixer CC control, S/PDIF-vs-second-mixer option, MIDI-thru option, and read-only sample rate.

**Licensing / redistribution constraints.** The manual carries a standard proprietary notice: *"furnished under licence … copyright © 2022 Expert Sleepers Ltd. All rights reserved."* The downloaded HTML tool contains **no open-source license, no copyright grant, and no MIT/GPL notice** (verified by scanning the served file). Default copyright therefore applies: **treat the tool as a behavioral reference only.** Do not fork, wrap, redistribute, or vendor its code without written permission from Expert Sleepers. The SysEx protocol itself is documented in the manual and is fair to reimplement.

**SysEx observed / documented.** See Requirement 2.

Sources: [ES-9 firmware & config tool page](https://www.expert-sleepers.co.uk/es9firmware.html), [ES-9 firmware v1.3 User Manual (PDF)](https://www.expert-sleepers.co.uk/downloads/manuals/es9_user_manual_1.3.pdf), [config tool 1.3](https://www.expert-sleepers.co.uk/webapps/es9_config_tool_1.3.html).

---

## Requirement 2 — Configuration dump format inventory

**SysEx header (authoritative, manual §MIDI SysEx).** Every ES-9 message is:

```
F0  00 21 27   19   <command>   [payload…]   F7
```

- `00 21 27` = Expert Sleepers manufacturer ID.
- `19` = ES-9 device/family byte.
- 7-bit MIDI data; multi-byte numeric fields are sent as **3 bytes = 21-bit** values (see DC offset, mix value, filter params).

**Command map (documented):**

*Host → ES-9 (received):*
| Cmd | Meaning | Notes |
|---|---|---|
| `09` | Configuration dump (write) | `… 09 <chunk> 0 <configuration data> F7` — uploads full config to module |
| `22` | Request version string | replies via `32` Message |
| `23` | Request configuration dump | module replies with its config (see `08`) |
| `24` | Request save `<n>` | save current state to flash; `<n>`=0 standalone, 1 hosted; replies "OK" |
| `25` | Request restore `<n>` | load state from flash slot `<n>` |
| `26` | Request reset | reset **current** state to defaults (does not touch flash) |
| `2A`/`2B`/`2C` | Request mix / usage / sample rate | |
| `31` | Set HPF `<hpf>` | DC-blocking filter state (per input pair) |
| `32` | Set Options `<options>` | bit0 = use mixer-2 (not S/PDIF); bit1 = MIDI thru |
| `33` | Set Links `<link> <state>` | stereo link enable |
| `34` | Set Virtual Mix `<mix> <level>` | macro-mix fader |
| `35` | Set MIDI Channels `<USB ch> <DIN ch>` | mixer CC control |
| `36` | Set DC Offset `<channel> <3 bytes>` | per-output DC offset |
| `39` | Set Filter `<mixer><filter><type><3B freq><3B Q><3B gain>` | EQ band |
| `3A` | Set Smoothing `<mix> <state>` | |
| `40`–`43` | Set Inputs `<40+dspID 0-3> <8 bytes input routing>` | input routing of the 4 DSP blocks |
| `50`–`53` | Set Outputs `<50+dspID 0-3> <8 bytes output routing>` | output routing of the 4 DSP blocks |
| `60`–`6F` | Set Mix `<60+mixID 0-15> <channel 0-7> <3 bytes value>` | raw-mix fader |

*ES-9 → host (sent):*
| Cmd | Meaning |
|---|---|
| `08` | Configuration dump: `… 08 00 00 <configuration data> F7` — the module's complete state, used to init the tool |
| `11` | Mix: `<128 × 3-byte mix> <128 × virtual mix/pan>` |
| `12` | Usage (DSP load) |
| `14` | Sample rate `<3 bytes>` (read-only; set by host/ALSA) |
| `32` | Message: `<NULL-terminated ASCII>` (version string, "OK", error text) |

**Dump size.** The manual's own screenshots show a full config dump arriving as **`received sysex (267 bytes)`** and the version reply as **13 bytes**. So one complete `.syx` config ≈ **267 bytes** including `F0…F7`. The exact internal field layout of the `<configuration data>` blob is **not published**; it must be recovered from a captured dump (**bench task**).

**Checksum / terminator.** No checksum is documented; messages are delimited by `F0`/`F7` only. The module returns a `32` Message ("OK") after a save, which is the practical success signal. (**inferred:** validation should rely on total length + header match + successful "OK" echo rather than a checksum.)

**Hosted vs standalone scope.** Two flash slots hold two complete configurations: **standalone** (slot 0, no USB host — mixer inputs default to the analogue inputs, second mixer enabled) and **hosted** (slot 1, USB host present — mixer inputs default to USB channels, S/PDIF enabled). The panel knob toggles slots; the hosted config auto-loads **only the first time** USB connects (for crash/cable-pull recovery).

**Fields required for Maybelle patch safety** (subset that can silently misroute CV):
1. **Input DC-blocking / HPF (`31`)** — MUST be OFF on any input carrying CV.
2. **Output routing (`50`–`53`)** — which signal reaches 3.5mm outputs 1-8 (DAW channels 9-16).
3. **Input routing (`40`–`43`)** — which hardware input feeds each USB (DAW) input channel.
4. **Per-output DC offset (`36`)** — must match Maybelle's calibration, not fight it.
5. **Options (`32`)** — S/PDIF-vs-mixer-2 changes the third DSP block and thus available routing.
6. **Active slot (hosted vs standalone)** — the running config, not just the stored one.
7. **Sample rate (`14`, read-only)** — must match the ALSA/Pi config Maybelle opens.

Fields lower-priority for CV safety but worth surfacing: stereo links, EQ, smoothing, mixer faders, MIDI-CC channels.

Sources: [ES-9 v1.3 manual — MIDI SysEx section](https://www.expert-sleepers.co.uk/downloads/manuals/es9_user_manual_1.3.pdf).

---

## Requirement 3 — Maybelle patch profile model

**Fixed hardware channel map (authoritative — manual "Channel numbering").** These are the *default* DAW channel assignments; routing is user-reconfigurable, which is exactly why a profile must pin them.

Outputs (DAW/USB out → destination):
- DAW **1/2** → Main outputs (¼" balanced, **AC-coupled — unusable for CV**)
- DAW **1/2 & 3/4** → Headphone mix (DC-coupled but a mix, not for CV)
- DAW **5/6** → S/PDIF out
- DAW **7/8** → ES-5 expansion out
- DAW **9-16** → **3.5mm DC-coupled outputs 1-8** ← **Maybelle CV/gate/trigger/mod outputs live here**

Inputs (source → DAW/USB in):
- 3.5mm DC-coupled inputs **1-14** → DAW in **1-14** ← **Maybelle reads Pamela clock/reset/run and runtime-selection CV here**
- S/PDIF in → DAW in **15/16**

Electrical scaling (authoritative): **≈ ±10V at a 3.5mm jack ↔ 0 dBFS** (full scale) in the DAW, both directions. So nominal software scale is **1.0 FS ≈ 10 V ⇒ 1 V ≈ 0.1 FS**, i.e. one 1V/oct octave ≈ 0.1 FS *before* calibration. The "approximately" is why per-system calibration is mandatory.

**Proposed semantic profile (roles → hardware):** Maybelle should keep a semantic profile separate from the raw `.syx`, mapping each rack role to `{direction, es9_daw_channel, es9_phys_jack, coupling_required, voltage_range, scale/offset calibration, safety}`. Example roles:

| Role | Dir | DAW ch | Phys jack | Notes |
|---|---|---|---|---|
| `pamela_clock_in` | in | 1 | 3.5mm in 1 | DC-block MUST be off; gate/trigger threshold |
| `reset_in` | in | 2 | 3.5mm in 2 | edge-detect |
| `run_start_in` | in | 3 | 3.5mm in 3 | level/latch |
| `song_bank_cv_in` | in | 4 | 3.5mm in 4 | quantized-to-N selection, hysteresis |
| `song_select_cv_in` | in | 5 | 3.5mm in 5 | quantized selection |
| `channel_bank_cv_in` | in | 6 | 3.5mm in 6 | quantized selection |
| `pitch_cv_out_1` | out | 9 | 3.5mm out 1 | 1V/oct, needs scale+offset calib |
| `gate_out_1` | out | 10 | 3.5mm out 2 | 0/+5–8V, fixed level |
| `trigger_out_1` | out | 11 | 3.5mm out 3 | fixed-width pulse |
| `mod_cv_out_1` | out | 12 | 3.5mm out 4 | stepped modulation |

(Channel assignments above are Maybelle design choices within the DAW 1-14 in / 9-16 out envelope, not ES-9 mandates.)

**Song-bundle / channel-bank linkage:** bundles and channel banks should reference the profile **by role name / profile ID**, not by raw ES-9 channel numbers, so low-level routing lives in one place and a single profile swap re-targets all songs.

Sources: [ES-9 v1.3 manual — Inputs and Outputs / Channel numbering](https://www.expert-sleepers.co.uk/downloads/manuals/es9_user_manual_1.3.pdf), [ES-9 product page](https://www.expert-sleepers.co.uk/es9.html).

---

## Requirement 4 — ES-9 profile validation rules

A validator compares a downloaded `.syx` (parsed) against the semantic profile. Proposed severities:

**ERROR (block arming playback):**
- Input **DC-blocking / HPF ON** on any input carrying CV/clock (`31` bits) — the manual states the filter *must* be off for CV; this silently corrupts DC-level reads.
- A **CV/gate output role's DAW channel is not routed to its 3.5mm jack** (`50`–`53` output routing mismatch) — signal goes nowhere or to the wrong jack.
- Maybelle output role mapped to **DAW 1/2** (Main outs are AC-coupled — cannot pass CV).
- **Active flash slot ≠ expected** (running standalone when Maybelle expects hosted) — mixer inputs and S/PDIF differ.
- **Sample rate mismatch** between ES-9 (`14`) and the rate Maybelle opens on ALSA.

**WARN (require explicit confirmation):**
- **Per-output DC offset (`36`) non-zero** on a Maybelle CV output — may conflict with Maybelle's own software calibration; must be reconciled (pick one place to apply offset).
- **Options bit0 (mixer-2 vs S/PDIF)** differs from profile assumption — changes available routing.
- **Stereo link** enabled on a channel pair Maybelle drives independently (pan/level coupling surprises).
- **Mixer routing** injecting extra signal onto a CV output (summing — the manual warns multiple outputs to one destination are summed).
- **MIDI-CC mixer channels** enabled on a channel Maybelle also drives (external CC could move a fader mid-set).

**INFO:** EQ enabled, smoothing enabled, DSP-usage headroom.

**Cannot be validated from `.syx` (needs the physical checklist):** whether cables are actually patched, whether the right rack module is on the other end, output polarity/expected voltage at the jack, and real V/oct scaling. The profile must carry a human-readable **patch checklist** for these, and the runtime should treat config validation as necessary-but-not-sufficient.

**Arming policy recommendation:** default to **refuse-to-arm on any ERROR**, **confirm-to-proceed on WARN**, log INFO. Provide an operator override for bench/dev, but the safe live default blocks.

---

## Requirement 5 — Profile utility recommendation

**Recommended first path: validator-only, read-only.** Parse a user-downloaded `.syx` (firmware ≥ 1.3.0) + a Maybelle semantic profile, emit a human-readable pass/warn/fail report and a printable patch checklist. No SysEx is sent to hardware in phase 1.

Rationale: lowest hardware risk, immediately prevents the common silent-misroute failures, and needs only the (documented) header + total-length checks plus a decoded routing/HPF/offset view. Generation and especially **upload** (`09` write, `24` save-to-flash) should wait until the payload layout is confirmed by round-trip on real hardware and a version-compatibility policy exists.

Path comparison:
- **Validator-only (downloaded `.syx`)** — ✅ recommend first. Safe, useful, feasible from docs.
- **`.syx` generator** — later; needs confirmed payload layout + round-trip proof.
- **SysEx upload / flash-write** — latest; highest risk; requires bench + explicit slot targeting.
- **Wrap/fork official HTML tool** — ❌ blocked on licensing (all rights reserved); behavioral reference only.
- **Manual checklist** — ship alongside every phase; covers what config cannot observe.

**Unresolved spikes before any utility writes config to an ES-9:** (1) recover the `<configuration data>` byte layout from a captured dump; (2) confirm round-trip stability (download → upload → re-download is byte-stable or semantically stable); (3) get written clarification on tool-code reuse rights if a wrapper is ever desired; (4) decide version-pinned parsers keyed on the version string (`22`/`32`).

---

## RECOMMENDATION

### ES-9 profile schema (what a Maybelle "ES-9 profile" must capture)

```yaml
es9_profile:
  profile_id: string
  firmware_min: "1.3.0"          # validator floor
  config_tool_version: "1.3.0"
  target_slot: hosted            # hosted | standalone (must match running slot)
  sample_rate: 48000             # must match ALSA open + ES-9 report (14H)
  buffer_frames: 128             # Pi/ALSA side; not in ES-9 config, but part of the profile
  usb_channels: { in: 16, out: 16 }
  input_dc_blocking:             # per pair 1/2..13/14; false = required for CV
    "1/2": false
    "3/4": false
    # …
  options:
    third_block: spdif           # spdif | mixer2  (Options bit0)
    midi_thru: false             # Options bit1
  output_dc_offset:              # per 3.5mm out 1-8; where offset calibration is applied
    out_1: 0
    # …
  roles:
    - name: pamela_clock_in
      direction: in
      daw_channel: 1             # DAW/USB channel (in 1-14 / out 9-16)
      phys_jack: "in_1"          # 3.5mm jack label
      coupling_required: dc      # dc for CV; error if HPF on
      voltage_range: [0, 8]
      signal: gate
      safety: edge_detect
    - name: pitch_cv_out_1
      direction: out
      daw_channel: 9
      phys_jack: "out_1"
      coupling_required: dc
      voltage_range: [-5, 5]
      calibration: { volts_per_oct: 1.0, scale_fs_per_volt: 0.1, offset_fs: 0.0, ref: c0 }
      signal: pitch_cv
  patch_checklist:               # things config cannot verify
    - "Confirm 3.5mm out 1 patched to VCO 1 V/oct in"
    - "Confirm Pamela clock patched to ES-9 in 1"
```

Load-bearing invariants the schema enforces: CV inputs have DC-blocking OFF; CV outputs land on DAW 9-16 / 3.5mm 1-8 (never Main 1/2); running slot == `target_slot`; ES-9 sample rate == profile sample rate; DC offsets reconciled with software calibration.

### Default / hosted baseline

Adopt the ES-9's own **"Reset to defaults suitable for hosted"** as the Maybelle baseline: mixer inputs set to USB channels, S/PDIF enabled, and the default channel map (DAW in 1-14 ← 3.5mm in 1-14; DAW out 9-16 → 3.5mm out 1-8; Main 1/2, S/PDIF 5/6, ES-5 7/8). Then Maybelle's profile overlays two required deltas: (a) **turn OFF input DC-blocking** on every input pair used for clock/CV, and (b) **zero the per-output DC offsets** (or set them to measured calibration) on the 3.5mm outputs Maybelle drives. This yields a known-good starting config saved to the **hosted** slot, with a matching standalone slot as fallback.

### Calibration approach

The ES-9 provides **offset compensation only** (per-output DC offset, `36`); it has **no V/oct gain calibration**. So Maybelle must own **pitch-CV scaling in software**, à la Silent Way's Voice Controller:
- **Model:** per pitch-CV output store `{scale_fs_per_volt, offset_fs}` (optionally a small multi-point correction table). Start from nominal 0.1 FS/V and refine by measurement.
- **Method (bench):** output a rising voltage ramp, feed the target VCO's audio back into an ES-9 input (DC-block ON for that audio input, per manual), detect pitch, and fit voltage→pitch to derive true FS/V and offset. This mirrors Silent Way calibration and requires the physical loop.
- **Offset placement:** apply DC offset in **one** place — prefer Maybelle software (transparent, versioned) and keep ES-9 `36` offsets at zero — to avoid the two fighting. Flag non-zero ES-9 offsets as a WARN.
- **Temperature:** the ES-9 output DAC is stable; 1V/oct temperature drift is a property of the **downstream analog VCO**, not the ES-9. Any thermal compensation belongs to the module/quantizer, not the ES-9 profile. Maybelle can optionally support periodic recalibration but should not model ES-9 thermal drift.

Sources: [Silent Way overview](https://www.expert-sleepers.co.uk/silentway.html), [Sound on Sound — Silent Way review (calibration)](https://www.soundonsound.com/reviews/expert-sleepers-silent-way).

---

## Open / needs bench hardware (ES-9 + Pi)

These cannot be closed from docs and are **blocked on the physical ES-9 on a Pi bench**:

1. **Config-dump payload layout** — capture a real `08`/`.syx` dump (~267 bytes) and decode the `<configuration data>` field offsets for HPF, routing (`40`–`43`/`50`–`53`), options, links, DC offsets. Docs give message shapes, not the dump's internal packing.
2. **Round-trip stability** — verify download → upload → re-download is byte- or semantically stable across the official tool before trusting generation/upload.
3. **Real channel discovery on Linux/ALSA** — confirm the class-compliant device enumerates as 16-in/16-out on Raspberry Pi OS, which ALSA/PipeWire/JACK path gives stable DC output, and that DAW channels 9-16 truly map to 3.5mm out 1-8 as documented.
4. **V/oct calibration measurement** — derive true FS-per-volt and offset per pitch output via the audio-feedback loop; nominal 0.1 FS/V is unverified per unit.
5. **Latency / timing** — clock-in jitter and output timing for gates/triggers under the chosen Pi audio stack (relevant to `decide-base-platform`).
6. **Slot/active-config detection at runtime** — whether Maybelle can reliably read the *running* slot (via `23`/`08` dump) to enforce hosted-vs-standalone before arming.
7. **DC-offset units** — the `36` payload is 3 bytes (21-bit); the volts-per-LSB mapping is undocumented and must be measured.

Sources: [ES-9 firmware page](https://www.expert-sleepers.co.uk/es9firmware.html), [ES-9 v1.3 manual](https://www.expert-sleepers.co.uk/downloads/manuals/es9_user_manual_1.3.pdf), [ES-9 product page](https://www.expert-sleepers.co.uk/es9.html), [Thomann ES-9 (specs)](https://www.thomannmusic.com/expert_sleepers_es_9.htm), [Silent Way](https://www.expert-sleepers.co.uk/silentway.html).
