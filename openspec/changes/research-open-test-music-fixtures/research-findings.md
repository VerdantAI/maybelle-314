# Research Findings: Open Test Music Fixtures

> **This is web-research output.** Retrieval date: **2026-07-22**. All license
> claims below reflect what public pages stated on that date and must be
> re-confirmed per-file at intake time. This is not legal advice. Where a page
> could not be retrieved (HTTP 403) or a license was ambiguous, the item is
> flagged as **unverified** and pushed to the "Open / needs non-web follow-up"
> section rather than asserted.

## Summary of the licensing bar

The project design requires MIT-compatible / permissive fixture rights and
**explicitly excludes** non-commercial (NC), no-derivatives (ND), and
share-alike (SA) terms for anything committed or auto-downloaded
([design.md](./design.md), Non-Goals + "Prefer Permissive Fixtures"). In
practice that means:

- **Safe to commit / redistribute in-repo:** CC0 1.0, public-domain dedication,
  and project-authored content. (MIT/0BSD/BSD/Apache rarely appear on music
  files but are equally fine.)
- **CC-BY 4.0:** commercial + derivative use allowed, attribution required. Not
  strictly MIT-equivalent, but compatible with the project's attribution-manifest
  requirement. Usable if the attribution obligation is honored — treat as a
  second tier behind CC0.
- **Reference-only (do NOT commit / auto-download):** CC-BY-SA (share-alike),
  CC-*-NC, CC-*-ND, and proprietary/EULA content. These may be documented as
  manual, external test candidates only.

---

## Requirement 1 — Candidate fixture source research

Inventory of candidate sources, each with source URL, author, artifact types,
expected pipeline coverage, and availability.

| # | Source | Author/owner | Artifacts | License (as stated) | Commit-safe? |
|---|--------|-------------|-----------|---------------------|--------------|
| A | [m-malandro/CC0-midis](https://github.com/m-malandro/CC0-midis) | m-malandro | Standard MIDI files | **CC0 1.0** | **Yes** |
| B | [TiMauzi/imslp-midi-cc0-1.0](https://huggingface.co/datasets/TiMauzi/imslp-midi-cc0-1.0) | TiMauzi (crawl of IMSLP) | 1,113 MIDI, 406 composers | **CC0 1.0 / public domain** | **Yes** (per-file check) |
| C | [Mutopia Project](https://www.mutopiaproject.org/) | many contributors | MIDI + PDF + LilyPond | **Mixed:** public-domain, CC-BY, CC-BY-SA ([legal](https://www.mutopiaproject.org/legal.html)) | **Only PD/CC-BY entries** |
| D | Project-authored synthetic fixtures (mido / music21 / MuseScore export) | Maybelle project | MIDI (Type-1), optional Ardour session | project's own choice (MIT) | **Yes** |
| E | ["Dance of The Free Wolves"](https://musical-artifacts.com/artifacts/167) Ardour example session | lfz (Lucas Zawacki) | Ardour session + Zyn/Guitarix banks | **Unverified** (page 403'd) | No — see follow-up |
| F | [unfa](https://unfa.xyz/) music / stems / Ardour projects | unfa (Jakub Fiedorowicz) | songs, stems, possibly sessions | **Unverified** (historically CC-BY-SA) | No — reference/outreach |
| G | [Lakh MIDI Dataset](https://colinraffel.com/projects/lmd/) | C. Raffel | ~176k MIDI | CC-BY 4.0 *wrapper* over scraped material | No — provenance risk |
| H | [MAESTRO](https://magenta.withgoogle.com/datasets/maestro) | Google Magenta | MIDI + audio | **CC BY-NC-SA 4.0** | No — NC + SA |
| I | [Bitwig demo projects](https://www.bitwig.com/support/shop_license_activation/) | Bitwig GmbH | .bwproject | **Proprietary EULA** ([EULA](https://ec.crypton.co.jp/download/pdf/eula_bitwig.pdf)) | No — proprietary |

**Notable finding on Ardour itself:** the Ardour *application* is GPL, but Ardour
does not ship redistributable example sessions with permissive fixture rights;
community sessions carry their own separate (often CC-BY-SA) content licenses
([Ardour license page](https://ardour.org/copying.html),
[forum thread](https://discourse.ardour.org/t/share-sesison-files/105443)). So
"official Ardour example sessions with permissive rights" (Open Question in
design.md) appears to **not exist** as a ready-made resource — Ardour coverage
must come from project-authored sessions or a per-file-cleared community session.

---

## Requirement 2 — Fixture license verification

Per-candidate license evidence (exact license, URL, retrieval date, and the
four rights that matter).

### Tier 1 — Commit-safe (CC0 / public domain)

- **CC0-midis (A)** — Confirmed **CC0 1.0 Universal**. The repo LICENSE waives
  all rights: "reuse and redistribute as freely as possible in any form
  whatsoever and for any purposes, including without limitation commercial
  purposes." Redistribution ✔, modification ✔, commercial ✔, attribution **not
  required**. Source: [github.com/m-malandro/CC0-midis](https://github.com/m-malandro/CC0-midis).
  Retrieved 2026-07-22.
- **IMSLP CC0 dataset (B)** — Dataset card states files are "public domain and
  cc0-1.0", crawled from IMSLP 2024-07-21/22; 1,113 files / 406 composers; each
  row carries a `midi_source` and `metadata_source` URL for per-file audit.
  Attribution not legally required (citation is optional/courtesy). Underlying
  scores are public-domain classical works. Source:
  [huggingface.co/datasets/TiMauzi/imslp-midi-cc0-1.0](https://huggingface.co/datasets/TiMauzi/imslp-midi-cc0-1.0).
  Retrieved 2026-07-22. **Caveat:** IMSLP contributor-generated MIDI can carry a
  different tag than the score; verify the specific file's `midi_source` page
  before committing.

### Tier 2 — Attribution-required but permissive (CC-BY / public-domain subset)

- **Mutopia Project (C)** — The [legal page](https://www.mutopiaproject.org/legal.html)
  documents exactly three options, each declared **per piece**:
  1. **Public Domain** — no obligations.
  2. **CC BY** (4.0/3.0/2.5) — credit required, no share-alike.
  3. **CC BY-SA** (1.0–4.0) — credit **and** derivatives must use the same
     license → **share-alike, excluded by design.md**.
  Redistribution ✔, modification ✔, commercial ✔ for options 1–2. **Action:**
  filter to per-piece Public-Domain or CC-BY entries only; reject CC-BY-SA
  entries. Retrieved 2026-07-22.

### Deferred / rejected

- **Lakh (G)** — CC-BY 4.0 is only a *wrapper* the compiler placed over MIDI
  files scraped from the public web; the underlying compositions/arrangements are
  largely commercial copyrighted material with no cleared rights. Provenance is
  unsafe for redistribution. Reference-only. Source:
  [colinraffel.com/projects/lmd](https://colinraffel.com/projects/lmd/).
- **MAESTRO (H)** — **CC BY-NC-SA 4.0**: non-commercial + share-alike → excluded.
  Source: [magenta.withgoogle.com/datasets/maestro](https://magenta.withgoogle.com/datasets/maestro).
- **Bitwig demo projects (I)** — proprietary EULA; the Demo Edition even forbids
  exporting work results. Not redistributable. Excluded. Source:
  [Bitwig EULA](https://ec.crypton.co.jp/download/pdf/eula_bitwig.pdf).
- **"Dance of The Free Wolves" (E)** — license field could not be retrieved
  (musical-artifacts.com returned HTTP 403 to the research fetch). Independent of
  license, the session **bundles third-party dependencies** (a ZynAddSubFX set,
  a Guitarix/LADSPA bank, and 120 BPM loops from Stretta's "Total Harmonic
  Distortion" pack) whose rights are separate and unverified. Deferred. Source:
  [musical-artifacts.com/artifacts/167](https://musical-artifacts.com/artifacts/167).
- **unfa (F)** — no machine-readable license was found on unfa.xyz for
  session/stem files. unfa's public posture is libre/Linux-audio and his work is
  historically CC-BY-**SA** (share-alike), which — even if confirmed — is
  excluded for committed fixtures. Best treated as an outreach/reference target.
  Source: [unfa.xyz](https://unfa.xyz/),
  [unfa on Patreon](https://www.patreon.com/unfa).

---

## Requirement 3 — Attribution manifest

No third-party fixture is yet accepted, so no live manifest is populated. Web
research does let us finalize the **schema** the design calls for. Proposed
machine-readable fields (one record per fixture file or fixture set):

```
- fixture_id
- files: [relative paths]
- work_title
- artist / author
- source_url
- retrieval_date
- license: SPDX id where possible (e.g. "CC0-1.0", "CC-BY-4.0")
- license_url
- attribution_required: bool
- required_credit_text     # verbatim string to render
- redistribution: allowed | reference-only
- transformations: [e.g. "exported to SMF Type-1", "trimmed to 8 bars"]
- checksum: sha256
- dependencies: [plugins/samples the fixture needs, or "none"]
```

For CC0/PD items `attribution_required` is `false` but recording author/source
is still valuable for provenance (Requirement: Preserve Fixture Provenance).
Human-facing credit should appear in README and in any generated test report
that references the fixture (design.md "Make Attribution First-Class").

---

## Requirement 4 — Fixture suitability assessment

What each source can and cannot exercise in the Maybelle pipeline (notes, CC,
pitch bend, track names, markers, tempo maps, time sigs, stems, Ardour session
structure).

- **CC0-midis / IMSLP-CC0 / Mutopia MIDI (A/B/C):** classical/piano-centric
  Standard MIDI Files. Strong for **notes, velocity, tempo maps, time
  signatures, and multi-track (Type-1) structure**. Generally **weak/absent** for
  CC automation, pitch bend, and markers (transcriptions rarely include them).
  No stems, no Ardour session structure. File sizes small (KB) — commit-friendly.
  Note: this matches the project's real Bitwig export (notes + velocity only per
  [README](../../../README.md)), so these are good "authored-notes" fixtures but
  do **not** cover modulation/expression.
- **Ardour community sessions (E, if ever cleared):** the only realistic source
  of true **Ardour session structure, track names, region layout, and stems** —
  but they drag in plugin/sample dependencies and large file sizes, hurting
  reproducibility. Record every dependency (design.md risk: "Real Ardour sessions
  depend on unavailable plug-ins").
- **Project-authored synthetic (D):** the only way to get **guaranteed coverage
  of the hard cases** — CC automation, pitch bend curves, dense marker/tempo/time-
  signature maps, edge cases (running status, meta events, negative CV mappings)
  — deterministically and at tiny size. Weakness: less "real-world messy" than a
  human DAW export.

**Coverage conclusion:** No single third-party source covers the modulation/
expression and Ardour-structure cases. A hybrid is required.

---

## RECOMMENDATION

### Use first (commit these)
1. **Project-authored synthetic fixtures (D) — primary.** Generate small,
   deterministic SMF Type-1 files with a permissive toolchain the project already
   favors: **mido (MIT license), MIDIUtil, music21 (BSD), or pretty_midi (MIT)**.
   These give full, targeted coverage of notes+velocity, CC automation, pitch
   bend, tempo maps, time signatures, and markers, at KB sizes, with zero
   third-party license risk. This is the safest and most controllable path and
   directly satisfies the design's synthetic-fallback goal.
2. **A small curated set from CC0-midis (A) and the IMSLP-CC0 dataset (B) —
   secondary.** Adds realistic, human-authored classical MIDI (notes/velocity/
   tempo/time-sig) under CC0, committable with no attribution obligation.
   Hand-pick a handful of small files and record per-file provenance/checksums.
3. **Mutopia (C), Public-Domain or CC-BY entries only — optional third tier**
   for genre variety, honoring CC-BY attribution in the manifest + README.

### Licensing handling
- Commit only **CC0/public-domain (Tier 1)**; add **CC-BY (Tier 2)** only with
  its attribution recorded in the manifest and README.
- **Never commit or auto-download** share-alike (unfa, Mutopia CC-BY-SA, MAESTRO),
  NC/ND, proprietary (Bitwig), or provenance-murky (Lakh) material. Document them
  as external/manual test candidates only.
- Record SPDX id, source URL, retrieval date, and sha256 for every committed file.

### Synthetic-fallback plan (also the primary path here)
Author fixtures in-repo via a documented generator script (MIT toolchain) that
emits: (a) a "Bitwig-like" notes+velocity-only Type-1 SMF; (b) a "rich" SMF with
CC, pitch bend, markers, tempo map, and multiple time signatures; (c) an
edge-case SMF. Keep the generator committed so fixtures are reproducible and
auditable. Author one minimal, dependency-free **Ardour session** in-house to
cover session-structure parsing without third-party rights entanglement.

### Artists to credit (if their material is later cleared)
- **unfa (Jakub Fiedorowicz)** — flagship libre-Ardour artist; credit boldly if
  outreach yields permissively-licensed material.
- **lfz / Lucas Zawacki** — "Dance of The Free Wolves" author, if cleared.
- Mutopia CC-BY transcribers and IMSLP contributors per-file.

---

## Open / needs non-web follow-up

- **unfa exact license + downloadable session/stem files.** unfa.xyz exposed no
  machine-readable license for project files; his catalog appears CC-BY-SA
  (share-alike, excluded). **Action: direct artist outreach** to ask whether he
  will release a small session/stem set under CC0 or CC-BY for Maybelle testing.
  (design.md Open Question — cannot be settled by web search.)
- **"Dance of The Free Wolves" license.** musical-artifacts.com returned HTTP 403
  to automated fetch; the listed license field must be read manually in a
  browser, and its bundled Zyn/Guitarix/Stretta dependencies each need separate
  rights confirmation before any use.
- **Per-file IMSLP-CC0 verification.** The dataset is aggregate-tagged CC0; a
  human should spot-check individual `midi_source` pages, since contributor MIDI
  can differ from the score's tag.
- **Legal sign-off on treating CC-BY as acceptable for commit.** design.md lists
  MIT/CC0-style terms as preferred; whether CC-BY's attribution obligation is
  acceptable for committed fixtures (vs synthetic/CC0 only) is a **project/legal
  decision**, not a web-research fact.
- **Maximum committed fixture size** (design.md Open Question) — a project policy
  decision, not researchable on the web.
- **Choice of synthetic generator dependency** (mido vs music21 vs MIDIUtil) —
  confirm against the base-platform package/licensing decision in
  `decide-base-platform`.
