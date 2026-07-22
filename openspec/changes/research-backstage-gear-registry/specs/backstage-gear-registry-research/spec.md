## ADDED Requirements

### Requirement: Legitimate gear-data population
The project SHALL define how the gear registry is populated without scraping ModularGrid.

#### Scenario: Population paths are defined
- **WHEN** the gear-registry research is performed
- **THEN** it defines populating the registry by manual module entry, by importing the user's own ModularGrid rack export (XML/JSON), and by storing ModularGrid rack URLs as reference links
- **AND** it establishes that Maybelle does not scrape or fetch/parse ModularGrid content, because ModularGrid provides no public API and does not permit scraping
- **AND** manual entry works with no external dependency as the always-available path

### Requirement: Gear-registry data model
The project SHALL define the registry data model.

#### Scenario: Module records are defined
- **WHEN** the registry model is defined
- **THEN** each module record carries fields such as manufacturer, name, HP, function/tags, an optional manual URL, an optional local cached-manual path, an optional ModularGrid URL, and notes
- **AND** multiple racks / ModularGrid URLs are supported
- **AND** the registry is distinct from the ES-9 rig profile, not required for playback, with at most an optional cross-reference

### Requirement: Manuals access without redistribution
The project SHALL define manual access that respects copyright.

#### Scenario: Manuals are surfaced safely
- **WHEN** the research defines manual access
- **THEN** manuals are accessed by linking to official sources
- **AND** an optional, user-initiated local cache may store a personal offline copy, never committed to the repo, bundled in a release, or shared between users
- **AND** no copyrighted manuals or ModularGrid data are bundled or redistributed

### Requirement: In-app and agent lookup integration
The project SHALL define how the registry serves lookups and agent assistance.

#### Scenario: Registry serves help
- **WHEN** the research defines integration
- **THEN** the registry and any cached manuals are a context source for in-app module lookups and for agents assisting the user
- **AND** this is consistent with the Backstage in-app-help requirement and the `docs/authoring/` reference material

### Requirement: Licensing and terms boundary
The project SHALL keep the feature within licensing and terms-of-service limits.

#### Scenario: Boundary is explicit
- **WHEN** the feature is specified
- **THEN** it records that there is no scraping of ModularGrid and no redistribution of manuals or ModularGrid data
- **AND** all gear data is user-provided (manual entry or the user's own export) plus links and personal caches only
