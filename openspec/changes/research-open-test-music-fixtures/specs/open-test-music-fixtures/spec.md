## ADDED Requirements

### Requirement: Candidate fixture source research
The project SHALL research candidate sources for Ardour sessions, MIDI files, stems, and DAW export artifacts before adding test fixtures.

#### Scenario: Candidate sources are inventoried
- **WHEN** open test music fixture research is performed
- **THEN** it includes unfa and other Ardour community sources, official Ardour examples if available, open MIDI repositories, and generated or project-authored fixture options
- **AND** each candidate records source URL, artist or author, artifact types, expected pipeline coverage, and availability status

### Requirement: Fixture license verification
The project SHALL verify license compatibility for each candidate fixture before inclusion.

#### Scenario: License evidence is recorded
- **WHEN** a fixture candidate is evaluated
- **THEN** the research records the exact license, license URL or text, retrieval date, redistribution rights, modification rights, commercial-use rights, and whether attribution is required
- **AND** it rejects or defers candidates with unclear, non-commercial, no-derivatives, or otherwise incompatible terms

### Requirement: Attribution manifest
The project SHALL define a machine-readable attribution manifest for any accepted third-party fixture.

#### Scenario: Artist credit is captured
- **WHEN** a third-party fixture is accepted
- **THEN** its attribution manifest includes artist name, work title, source URL, license, required credit text, fixture files, transformations, and checksum information
- **AND** the fixture is also credited visibly in human-facing documentation

### Requirement: Fixture suitability assessment
The project SHALL assess whether each candidate fixture covers Maybelle pipeline behavior that synthetic tests do not cover.

#### Scenario: Fixture value is evaluated
- **WHEN** a candidate fixture is reviewed
- **THEN** it identifies which features it exercises, such as notes, CC automation, pitch bend, track names, markers, tempo maps, time signatures, stems, Ardour session structure, or DAW export behavior
- **AND** it records any plug-in, sample, file size, or reproducibility limitations

### Requirement: Fixture intake recommendation
The project SHALL recommend a first fixture strategy before adding third-party music to the repository.

#### Scenario: Fixture strategy is ready
- **WHEN** open test music fixture research is complete
- **THEN** it recommends whether to commit third-party fixtures, generate synthetic fixtures, author original fixtures, use optional external fixture packs, or combine these approaches
- **AND** it lists any artists to credit, permission requests needed, and unresolved licensing questions
