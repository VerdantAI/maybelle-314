# DAW Interchange Research — Findings

**Type:** Web-research output (WebSearch/WebFetch).
**Retrieval date:** 2026-07-22.
**Scope:** Web-doable portions of `tasks.md`. Bench work that needs a running DAW or hardware is
explicitly flagged in the "Open / needs non-web follow-up" section and is **not** covered here.

This document builds on the initial research already recorded in `design.md` (including the Bitwig
`File > Export MIDI` = notes+velocity-only finding). It verifies, extends, and sources those claims
and maps findings onto each Requirement in `specs/daw-interchange-research/spec.md`.

A note on source quality: several precise export-behavior facts (e.g. "tempo map not written to SMF")
are best documented in vendor forums/trackers rather than manuals. Where a manual does not state the
behavior, the forum/tracker link is given and the claim is marked accordingly. Anything that could
only be confirmed by exporting a file is deferred to the bench spike, not asserted here.

---

## Requirement: Ardour-on-Pi assessment

**Verdict: Ardour is installable and usable on a Raspberry Pi 5, but as an authoring/export host, not
as the Maybelle runtime. For Maybelle it is best treated as "authoring/export, off-Pi" with
"runs-on-Pi" as a proven-but-unnecessary fallback.**

- Ardour installs on Pi 5 via the distro package manager (`sudo apt install ardour` on Raspberry Pi
  OS / Debian). Community reports confirm real multitrack recording works once an external audio
  interface is attached ([Ardour discourse: Install Ardour on a Pi5](https://discourse.ardour.org/t/install-ardour-on-a-pi5/113040)).
- ARM binaries come from **distro packages**, not the paid ardour.org bundle. Debian/Raspberry Pi OS
  ship Ardour, and Arch Linux ARM publishes an `aarch64` package
  ([Arch Linux ARM: ardour aarch64](https://archlinuxarm.org/packages/aarch64/ardour)). The official
  ardour.org "ready-to-run" installer is the funding path and is x86_64/macOS-oriented; Ardour itself
  documents source builds as challenging and unsupported for end users
  ([Building Ardour on Linux](https://ardour.org/building_linux.html)). Net: on a Pi you rely on the
  distro's ARM package, which exists and is maintained.
- Pi 5 has no onboard audio codec; audio needs an external interface
  ([Ardour discourse thread](https://discourse.ardour.org/t/install-ardour-on-a-pi5/113040)). For
  Maybelle this limitation is moot — the **ES-9 is the USB audio interface** — but it reinforces that
  the Pi's role here is I/O host, not a self-contained studio.
- Relevance to Maybelle: the runtime is a clock-following CV/gate emitter, not a DAW. Running a full
  DAW on the runtime Pi would add a large GUI/JACK/session stack Maybelle does not need (consistent
  with the `design.md` decision to treat Ardour sessions as research input, not the runtime contract).
  **Recommendation: do not run Ardour on the runtime Pi.** Author off-Pi; the Pi consumes a normalized
  bundle. "Ardour-on-Pi works" is a useful de-risking fact (a Pi could double as an authoring box in a
  pinch) but is not the intended architecture.

---

## Requirement: Ardour export artifact inventory

**Web-verifiable summary (the definitive per-event inventory needs the bench spike — see Open section):**

| Ardour export path | Preserves | Loses (per docs/forum) |
|---|---|---|
| **Stem export** (`Session > Export > Stem Export`) | audio or MIDI, one file per track/bus, kept in sync (silence included) | "All data will be lost except the actual audio/MIDI" — automation, routing, plugin state, session metadata ([Ardour Manual: Stem Exports](https://manual.ardour.org/working-with-sessions/interchange-with-other-daws/stem-exports/)) |
| **MIDI export** (SMF) | note data and CC events that live inside the MIDI region | **Tempo map / time-signature not written into the exported MIDI** as of Ardour 7.4 per user reports ([discourse](https://discourse.ardour.org/t/is-it-possible-for-ardour-to-pass-tempo-map-tempo-ts-to-exported-midi-file-according-to-session-settings/108813); [tracker 8457](https://tracker.ardour.org/view.php?id=8457)) |
| **Audio export** | BWAV 24-bit / 32-float, FLAC, WAV, etc. | n/a (rendered audio) ([Ardour Manual: Exporting](https://manual.ardour.org/exporting/)) |

Key nuances that separate Ardour from Bitwig:

- Ardour has **full in-application MIDI CC automation** and treats CC as first-class track data
  ([Ardour Manual: MIDI Editing](https://manual.ardour.org/working-with-midi/)). This is richer than
  Bitwig's export path. **However**, whether that CC/automation actually lands in an exported `.mid`
  file depends on whether the data lives as MIDI events inside the region versus in a separate
  automation lane — this is exactly the ambiguity the bench spike must resolve. Do not assume Ardour
  SMF export carries automation lanes; assume it carries region-embedded notes+CC and verify the rest.
- Ardour's tempo map not travelling in the SMF is a concrete gap for a clock-following system. Because
  Maybelle **follows Pamela's external clock**, the exported file's internal tempo map is
  lower-stakes than for a DAW-to-DAW transfer (beat positions matter more than absolute BPM), but
  time-signature/bar structure still matters for bank/section metadata and must be supplied out-of-band
  in the manifest if the SMF omits it. A community helper (`ArdourMIDIExport`) exists to merge MIDI
  tracks, indicating the stock export is per-region/per-track oriented
  ([dbolton/ArdourMIDIExport](https://github.com/dbolton/ArdourMIDIExport)).
- **Ardour session file** is XML on disk. It is inspectable but carries full DAW state (editing,
  routing, plugins) Maybelle does not need; `design.md` already decides against making `.ardour` the
  runtime contract. Web research does not surface a stable, documented, versioned schema intended for
  third-party consumption — treat direct session parsing as brittle/optional (consistent with the
  design's risk note).

**Missing-from-export vs Maybelle needs:** rack/channel→ES-9 output mapping, bank structure, CV
selection behavior, calibration, and quantization/debounce metadata are **not expressible in any DAW
export** and must live in a Maybelle manifest regardless of source DAW.

---

## Requirement: Interchange format comparison

| Candidate | Data coverage | License | Stability | DAW support | Runtime fit for clock-following CV/gate |
|---|---|---|---|---|---|
| **Standard MIDI File (Type-1)** | notes, velocity, CC, pitch bend, program/bank, meta events; tempo/TS *if the exporter writes them* | open/ubiquitous | very stable, decades | universal export | **Best baseline.** Beat-positioned note/gate events map directly to CV/gate. Simple parsers everywhere. |
| **Ardour stem export (audio/MIDI)** | audio or MIDI only, per track, synced | GPL tool | stable | Ardour (and any DAW's audio bounce) | Good for *baked* modulation/waveforms (audio stems → sampler), not for live control events. |
| **Ardour session (`.ardour` XML)** | full DAW state | GPL app | schema not published as an interchange contract | Ardour only | Poor: over-rich, DAW-coupled, brittle to parse across versions. |
| **DAWproject (ZIP+XML)** | notes+expressions, automation (incl. MIDI messages, tempo, time sig), audio clips/fades, plugin state, track structure, clip launcher | **MIT** ([bitwig/dawproject](https://github.com/bitwig/dawproject)) | v1.0, declared stable | Bitwig, Studio One, Cubase/Cubasis/VST Live, n-Track; Reaper via 3rd-party tool; **not** Ableton/Logic/Ardour natively (see below) | High *coverage*, but heavier XML/ZIP parse; overkill for a notes+gate runtime; value is authoring-side richness, not runtime. |
| **Maybelle song bundle (SMF + JSON/TOML manifest)** | whatever the manifest defines: bundle of SMF(s) + rack mappings, banks, ES-9 output map, calibration, CV-selection metadata | project-owned (MIT) | project-controlled | any DAW that can emit SMF (+ optional stems) | **Best overall fit.** Runtime parses one normalized contract; DAW differences absorbed at authoring time. |

DAWproject specifics (verified):

- MIT-licensed, ZIP container of XML (`project.xml` + `metadata.xml`), covers notes/automation/audio/
  tempo/time-sig/plugin state/track structure/clip launcher; v1.0 stable
  ([bitwig/dawproject README](https://github.com/bitwig/dawproject);
  [Bitwig DAWproject FAQ](https://www.bitwig.com/support/technical_support/dawproject-file-format-faqs-62/)).
- Native support as of the v1.0 announcements: **Bitwig Studio 5.0.9, PreSonus Studio One 6.5,
  Steinberg Cubase 14 / Cubasis 3.7.1 / VST Live 2.2, n-Track**. Reaper via a third-party tool, not
  native. Ableton, Logic, and Ardour have **no native DAWproject export**
  ([Bitwig: transferring projects](https://www.bitwig.com/stories/transferring-projects-between-bitwig-studio-and-other-daws-333/);
  [PreSonus: Introducing DAW Project](https://support.presonus.com/hc/en-us/articles/19743606863629-Introducing-DAW-Project);
  [Synthtopia announcement](https://www.synthtopia.com/content/2023/09/26/bitwig-and-presonus-introduce-open-dawproject-format-for-sharing-audio-projects-between-daws/)).

**Takeaway:** DAWproject is the right *open standard to watch and to keep as an optional richer import
path*, but it is not present in one of Maybelle's two current authoring tools (Ardour) and is
heavier than a clock-following gate engine needs. SMF-plus-manifest remains the pragmatic runtime
contract; DAWproject is a future authoring-side on-ramp for the DAWs that speak it (notably Bitwig,
which is the current test bench).

---

## Requirement: Cross-DAW abstraction assessment

What each DAW can hand Maybelle through **standard exports** (not counting a bespoke exporter):

| DAW | SMF export | Automation/CC in that SMF | Tempo map in SMF | Stems/audio | DAWproject | Path to normalized bundle |
|---|---|---|---|---|---|---|
| **Ardour** | yes | region CC yes; automation lanes **TBD (bench)** | **no** (per forum/tracker) | yes (stem/audio) | no (native) | SMF + stems + manifest |
| **Bitwig** | yes (Arrangement) | **no** — notes+velocity only | no | yes (audio) + **DAWproject** | **yes (native)** | SMF+manifest today; DAWproject later |
| **Studio One** | yes | richer than Ableton | (varies) | yes | **yes (native)** | SMF+manifest or DAWproject |
| **Ableton Live** | yes | "no extra options for exporting automation" | **no** (defaults to 120 BPM in target) ([Ableton: Extracting a Tempo Map](https://help.ableton.com/hc/en-us/articles/360000046760-Extracting-a-Tempo-Map-from-Live)) | yes | no | SMF+manifest (+ tempo-track workaround) |
| **Logic Pro** | yes | partial; markers importable via MIDI | (varies) | yes | no | SMF+manifest |
| **Reaper** | yes, incl. **"Embed tempo map"** option in Export Project MIDI ([Cockos forum](https://forums.cockos.com/showthread.php?t=217388)) | via MIDI items | **yes (opt-in)** | yes | 3rd-party tool | SMF+manifest (strongest stock SMF) |

Cross-DAW observations:

- **The common denominator across every DAW is notes + velocity in a Type-1 SMF.** Everything richer
  (CC, pitch bend, tempo map) is exporter-specific and frequently requires workarounds (Ableton's
  "print a click track" trick; Reaper's opt-in embed; Bitwig drops it entirely). This validates
  `design.md`'s "notes as the reliable carrier" stance.
- Tempo-map portability is unreliable in general (Ableton drops it, Ardour drops it, Reaper opts in).
  Maybelle sidesteps this because it **follows Pamela as master clock** — beat positions, not embedded
  BPM, are what the runtime consumes. Absolute tempo/section structure that matters for banks should
  be authored into the manifest, not trusted from the SMF.
- DAWproject cleanly covers automation/tempo but only unifies the Bitwig/Studio One/Cubase family —
  not Ardour, Ableton, or Logic — so it cannot be the single normalizing contract across Maybelle's
  stated DAW set.

---

## Requirement: Runtime importer boundary (normalized bundle)

The normalized Maybelle **song bundle** = one or more Standard MIDI Files + a manifest. Field
ownership:

**Runtime-native fields (belong in the normalized bundle; DAW-agnostic):**
- Beat-positioned note/gate/trigger events (from SMF) — the universal common denominator.
- Per-track/channel → ES-9 output routing map.
- Song banks / song selection / channel banks structure.
- ES-9 output calibration (V/oct scaling, offsets) and gate/trigger shape.
- CV-selection behavior: quantization to N choices, debounce/hysteresis/latching.
- Optional: baked-modulation references (audio stems / single-cycle waveforms) and the trigger note
  that fires them, per the synced-LFO-sampler-authoring approach.
- Optional: time-signature / section / bar metadata carried in the manifest (because SMF tempo/TS is
  not reliably exported).

**Authoring-source-specific (stays on the authoring side; never in the runtime):**
- Which DAW produced it (Ardour vs Bitwig vs other) and any DAW session/plugin/routing state.
- How modulation was realized upstream (Bitwig automation lane, Ardour CC lane, baked waveform) — the
  runtime only ever sees notes/gates + optional baked-audio references.
- DAWproject/session XML, stem-render settings, export scripts, and any DAW-specific conversion.

This preserves the `design.md` two-layer model: **DAW-specific producers emit the bundle; the Pi
runtime parses only the normalized bundle.** No DAW parser (Ardour, DAWproject, Bitwig) belongs in
the runtime.

**Ardour↔Bitwig divergence, resolved:** the two authoring tools disagree on how much rides in the
export (Bitwig = notes+velocity only; Ardour = notes + potentially CC). The normalizing rule is to
**target the intersection**: notes/gates are the contract; modulation is carried either as (a) baked
sampler waveforms triggered by a plain note, or (b) explicit manifest metadata — **not** as
export-embedded automation, since that does not survive Bitwig at all and is uncertain in Ardour SMF.

---

## Requirement: Existing project and module survey

| Precedent | Pattern relevant to Maybelle | Adopt / avoid |
|---|---|---|
| **Squarp Hapax** | Imports/exports **Type-0 and Type-1 SMF with notes + CC**, from a `MIDI/` folder at SD-card root; projects saved to SD ([Hapax manual: MIDI import/export](https://squarp.net/hapax/manual/modetrack/); [Hapax manual p84 mirror](https://www.manualslib.com/manual/3392916/Squarp-Instruments-Hapax.html?page=84)) | **Adopt:** SD-card `MIDI/` convention, per-track import selection, CC-in-SMF as a proven interchange baseline. |
| **Squarp Hermod+** | Eurorack module: 8 CV/Gate + 8 MIDI tracks, records CV/Gate + MIDI notes + modulation, **sync-safe project load from SD** without interrupting playback ([Hermod+](https://squarp.net/hermodplus/)) | **Adopt:** in-sync/glitch-free bank/project switching (matches Maybelle's live CV-driven selection). |
| **OXI One MkII** | Firmware 2.0 **MIDI Player** imports/plays MIDI files from microSD; 8×CV + 8×gate outputs configurable for pitch/LFO/env/clock ([Sound on Sound review](https://www.soundonsound.com/reviews/oxi-one-mkii); [firmware 2.0](https://weraveyou.com/2026/05/oxi-one-mkii-firmware-2-0-sequencer-modes-2026/)) | **Adopt:** "play authored MIDI files from SD → CV/gate" is essentially Maybelle's core loop, validated in a shipping product. |
| **Patchbox OS (Blokas)** | Raspberry Pi OS Lite–based audio image, JACK auto-config wizard, module system for swapping projects ([Patchbox OS](https://blokas.io/patchbox-os/); [docs](https://blokas.io/patchbox-os/docs/)) | **Reference:** precedent that a Pi can be a low-latency appliance-style audio/MIDI host; the module/wizard pattern is a nice-to-have, but Maybelle needs a narrower stack. |
| **Zynthian** | Fully open Pi 5 synth platform, DSI 5" touch + encoders + step sequencer ([zynthian.org](https://zynthian.org/); [specs](https://zynthian.org/technical-specifications)) | **Reference:** strong precedent for Pi 5 + 5" DSI touch UI (mirrors Maybelle's Touch Display 2). Avoid its scope (it is a full instrument/groovebox). |

Takeaway: three shipping hardware sequencers (Hapax, Hermod+, OXI One MkII) already do "authored
MIDI on SD card → CV/gate out," and two Pi audio platforms (Patchbox OS, Zynthian) prove the Pi-5 +
touchscreen appliance form factor. Maybelle's design sits squarely inside proven territory.

---

## RECOMMENDATION

**Interchange contract:** A **Maybelle song bundle = Standard MIDI File(s) (Type-1) + a project-owned
manifest** (JSON/TOML) carrying rack mappings, banks, ES-9 output/calibration, CV-selection behavior,
and optional baked-modulation references. Notes/gates are the DAW-agnostic common denominator across
every candidate DAW; the manifest owns everything no DAW export can express. Modulation is carried as
baked sampler waveforms (triggered by plain notes) or as manifest metadata — never as
export-embedded automation.

**First-class Maybelle inputs (recommended):**
1. **Type-1 SMF** (notes/velocity, plus CC where the DAW provides it) — primary.
2. **Maybelle manifest** — required companion; the normalizing layer.
3. **Audio stems / single-cycle waveforms** — optional, for baked modulation into the sampler.
   Ardour session XML, DAWproject, and any DAW-native session file are **not** first-class runtime
   inputs; they stay authoring-side.

**v1 scope: Ardour-first with a DAW-agnostic bundle.** Not Ardour-only (Bitwig is already the active
test bench and must be validated in parallel), and not full multi-DAW from day one (the long tail of
export quirks — Ableton tempo drops, Reaper opt-in embeds, DAWproject's partial DAW coverage — is not
worth absorbing before the runtime exists). Build the normalized SMF+manifest contract, prove it end
to end against **both Ardour and Bitwig exports** (the notes+velocity intersection), and keep the
producer layer pluggable so Studio One / Reaper / DAWproject sources can be added later without
touching the runtime.

**DAWproject verdict:** Genuinely attractive as an open, MIT, richly-scoped standard — keep it as a
**future optional authoring-side import path**, especially for Bitwig (native) — but **do not adopt it
as the v1 runtime contract**: it is absent from Ardour natively, absent from Ableton/Logic, heavier
than a gate engine needs, and does not unify Maybelle's actual DAW set.

**Ardour-on-Pi verdict:** Feasible (installs via distro ARM package, real recording works) but
**unnecessary and out of scope for the runtime** — author off-Pi, feed the Pi a normalized bundle.
The ES-9 supplies the audio I/O the Pi lacks.

---

## Open / needs non-web follow-up

1. **DAW export bench spike — BLOCKED: needs a running DAW host.** Web sources cannot definitively
   confirm which event types land in an actual exported file. Must export controlled sessions from
   **Ardour and Bitwig** (notes, velocity, CC automation, pitch bend, markers, tempo/TS changes, named
   tracks) and inspect the resulting `.mid`/stem bytes. Specifically unresolved by web research:
   - Does **Ardour SMF export write MIDI CC / automation-lane data**, or only region-embedded notes+CC?
   - Confirm **Bitwig notes+velocity-only** against the installed version and check newer versions for
     restored CC export.
   - Confirm Ardour's **tempo/time-signature omission** from SMF on the installed version.
2. **ES-9 timing spike — BLOCKED: needs hardware.** Which ES-9 I/O API on Raspberry Pi OS gives stable
   gate/trigger timing (feeds `decide-base-platform`). Not a DAW-interchange question but gates the bundle→output path.
3. **Manifest schema definition — non-web design work.** Formalize the runtime-native field list above
   into a concrete manifest schema (banks, channel→ES-9 map, calibration, CV-selection/debounce). This
   is the natural next OpenSpec change (song-bundle-format).
4. **Ardour session XML stability — optional, low priority.** If direct session inspection is ever
   wanted for authoring-time conversion, its cross-version stability must be validated against multiple
   real sessions; web research found no third-party-interchange schema guarantee.
5. **DAWproject converter viability — deferred.** Only worth a spike if/when a DAWproject-native DAW
   (Bitwig, Studio One) becomes a first-class producer; would need a bench test of a real
   round-tripped `.dawproject` against the Maybelle bundle fields.
