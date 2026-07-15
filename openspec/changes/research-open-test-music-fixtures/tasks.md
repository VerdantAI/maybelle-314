## 1. Source Research

- [ ] 1.1 Research unfa repositories, websites, videos, and release pages for Ardour sessions, MIDI, stems, or source files with explicit licenses.
- [ ] 1.2 Research official Ardour examples and community session files for license-compatible fixtures.
- [ ] 1.3 Research small open MIDI repositories or datasets with MIT, CC0, 0BSD, BSD, Apache-2.0, public-domain-equivalent, or similarly permissive terms.
- [ ] 1.4 Research whether direct artist permission is practical for a small Maybelle fixture pack.

## 2. License and Attribution Review

- [ ] 2.1 Record source URL, artist/author, artifact types, exact license, retrieval date, and license evidence for each candidate.
- [ ] 2.2 Reject or defer candidates with unclear, non-commercial, no-derivatives, share-alike, or otherwise incompatible terms.
- [ ] 2.3 Define the machine-readable attribution manifest fields and required visible credit format.
- [ ] 2.4 Define how generated reports and docs surface fixture artist credits.

## 3. Suitability Review

- [ ] 3.1 Evaluate candidates for pipeline coverage: notes, CC automation, pitch bend, track names, markers, tempo maps, time signatures, stems, and Ardour session structure.
- [ ] 3.2 Record plug-in, sample, size, reproducibility, and transformation limitations for each candidate.
- [ ] 3.3 Decide which fixture types should be committed, generated, external-only, or excluded.

## 4. Fixture Strategy Recommendation

- [ ] 4.1 Recommend the first fixture strategy: third-party fixtures, synthetic generated fixtures, project-authored fixtures, optional external packs, or a hybrid.
- [ ] 4.2 List artists to credit, permission requests needed, and unresolved licensing questions.
- [ ] 4.3 Feed findings into `research-daw-interchange-options`, song-bundle validation, CI design, and documentation.

## 5. Verification

- [ ] 5.1 Review the research output against every `open-test-music-fixtures` requirement.
- [ ] 5.2 Run OpenSpec validation or status checks for the completed change.
