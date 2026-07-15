## Context

Maybelle 314 needs realistic test inputs for the DAW interchange and song-bundle pipeline. Good fixtures could include Ardour sessions, Standard MIDI Files, exported stems, track names, markers, tempo maps, time signatures, pitch bend, CC automation, and other artifacts used to prove that Maybelle can transform authored music into clock-following MIDI/CV control data.

The project prefers MIT-compatible or otherwise permissive fixture rights so test data can be committed, redistributed, used in CI, and included in generated examples with minimal friction. Music licensing is different from software licensing: many open music projects use Creative Commons licenses, including non-commercial or share-alike variants that may not be acceptable for this repository. Even when an artist is open-source friendly, each file or release must have explicit rights.

unfa is a promising first research target because he is strongly associated with libre music production, Ardour, Linux audio, and community education. However, the research must verify exact license terms for any candidate material before inclusion.

## Goals / Non-Goals

**Goals:**
- Identify candidate Ardour sessions, MIDI files, stems, and DAW export artifacts suitable for automated pipeline testing.
- Research unfa and other Ardour community sources first.
- Require explicit license verification for every fixture candidate.
- Prefer MIT, CC0, 0BSD, BSD, Apache-2.0, public-domain-equivalent, or similarly permissive terms when applicable to media files.
- Define bold artist attribution requirements for README, fixture manifests, test output, and generated reports.
- Define fallback strategies using synthetic fixtures or project-authored fixtures if suitable third-party material cannot be found.

**Non-Goals:**
- Add any third-party media files.
- Treat Creative Commons non-commercial or no-derivatives material as acceptable.
- Assume GitHub repository license applies to embedded music files without per-file confirmation.
- Provide legal advice.

## Decisions

### Verify Rights Per Fixture

Each candidate fixture must have a recorded source URL, artist/author, license text or SPDX-equivalent, retrieval date, and evidence that redistribution and test use are allowed.

Rationale: music and DAW session assets often mix MIDI, audio, samples, plug-in presets, and project metadata. A repository-level license may not cover every asset.

Alternatives considered:
- Trust the artist's general open-source posture: fast, but legally weak.
- Use only generated fixtures: safe, but less representative of real DAW workflows.

### Prefer Permissive Fixtures, Allow Documented Externals

The first committed fixtures should be permissively licensed or project-authored. Non-permissive but useful examples may be documented as external manual test candidates, but must not be committed or downloaded automatically.

Rationale: CI and repository redistribution should stay clean. External examples can still guide manual research without becoming project dependencies.

Alternatives considered:
- Allow any Creative Commons license: broadens the pool, but non-commercial/share-alike/no-derivatives terms create downstream restrictions.
- Forbid all third-party fixtures: safest, but misses realistic pipeline coverage and community attribution opportunities.

### Make Attribution First-Class

Fixtures must include an attribution manifest and visible documentation credit. Any generated test report that references included third-party music should surface the artist and license.

Rationale: the project should be explicit and generous about crediting artists whose work helps Maybelle testing.

Alternatives considered:
- Keep attribution only in a LICENSE file: minimal compliance, but does not match the project's intent to boldly credit artists.
- Put attribution only in README: visible, but not machine-readable enough for tools and reports.

### Preserve Fixture Provenance

Fixture intake should preserve enough provenance to re-fetch or audit each asset later, including source URL, checksum, license evidence, and any transformation steps.

Rationale: test fixture trust decays if the origin and transformation history are unclear.

Alternatives considered:
- Commit files without provenance: easy initially, but difficult to audit.
- Always download live fixtures in tests: reduces repository size, but makes tests unreliable and dependent on external services.

## Risks / Trade-offs

- Candidate material is not actually MIT-compatible -> Mitigation: require explicit per-fixture license evidence before inclusion.
- unfa material is open but not permissive enough for repository fixtures -> Mitigation: document it as external/manual unless permission is obtained.
- Real Ardour sessions depend on unavailable plug-ins or samples -> Mitigation: prefer minimal sessions and record dependencies.
- Fixtures become too large for the repository -> Mitigation: separate small committed fixtures from optional external fixture packs.
- Attribution is lost in generated outputs -> Mitigation: require machine-readable attribution manifests used by tooling.

## Migration Plan

No runtime migration is required because this change adds planning artifacts only. Research output should feed into DAW interchange testing, song-bundle validation, CI fixture strategy, and documentation.

Rollback is limited to removing this OpenSpec change before implementation.

## Open Questions

- Does unfa publish any Ardour session, MIDI, or stem material under MIT-compatible, CC0, or similarly permissive terms?
- Are any official Ardour example sessions available with permissive fixture rights?
- Which MIDI datasets or repositories provide small files with licenses acceptable for committed tests?
- Should Maybelle request explicit permission from artists for a small fixture pack if public license terms are ambiguous?
- What is the maximum acceptable committed fixture size?
