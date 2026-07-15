## Why

Maybelle 314 needs realistic Ardour sessions, MIDI files, and exported artifacts to test the DAW-to-song-bundle pipeline without creating licensing risk. We should prefer material from open music communities, including Ardour community artists such as unfa if suitable rights are available, and we should prominently credit any artist whose work is included.

## What Changes

- Add a research spike for permissively licensed Ardour sessions, MIDI files, stems, and DAW export fixtures.
- Investigate unfa and other Ardour/community sources first, then broader MIDI and open music fixture sources.
- Require explicit license verification before any third-party music fixture is committed, downloaded in CI, or redistributed.
- Define an attribution manifest so included artists are boldly credited in test fixture docs, README material, and generated reports.
- Compare using third-party fixtures, generated synthetic fixtures, and project-authored fixtures where suitable licensed examples cannot be found.
- Do not add any third-party music files or download scripts in this change.

## Capabilities

### New Capabilities
- `open-test-music-fixtures`: Defines how the project researches, approves, credits, and uses permissively licensed music fixtures for pipeline testing.

### Modified Capabilities
- None.

## Impact

- Adds OpenSpec planning artifacts for test fixture sourcing and attribution.
- May inform DAW interchange research, song-bundle validation, fixture generation, CI tests, and documentation.
- No production code, test fixtures, dependencies, or third-party media files are introduced by this proposal.
