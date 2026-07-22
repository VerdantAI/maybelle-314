## Why

Users want Backstage to keep track of their rack's gear and give them fast access to each module's manual. Letting a user list their modules — or point to one or more ModularGrid racks — turns Backstage into a rig reference: the operator (and an assisting agent) can look up what a module does and open its manual without leaving the tool. This complements the in-app authoring help already required of Backstage. Before building it, we need to settle how gear data can legitimately be obtained (ModularGrid has no public API and prohibits scraping) and how manuals can be surfaced without redistributing copyrighted PDFs.

## What Changes

- Add a spike for a Backstage **gear registry**: list modules manually and/or reference one or more ModularGrid racks, with each module's manual available for easy access.
- Establish the legitimate data paths: **manual module entry**, **import of the user's own ModularGrid rack export (XML/JSON, available to premium "Unicorn" accounts)**, and **storing ModularGrid URLs as reference links** — explicitly **no scraping** of ModularGrid (no API; scraping not permitted).
- Define the gear-registry data model (per-module fields, manual links, optional local cache).
- Define manuals access: **link to official manuals** and optionally **cache a copy for the user's own offline use**, never bundling or redistributing copyrighted manuals.
- Define how the registry serves in-app lookups and agent assistance, and its relationship to the ES-9 rig profile.
- Do not implement the registry, importer, manual cache, or any ModularGrid integration in this change.

## Capabilities

### New Capabilities

- `backstage-gear-registry-research`: Defines requirements for a Backstage gear registry with manuals, sourced legitimately without scraping ModularGrid or redistributing manuals.

### Modified Capabilities

- None.

## Impact

- Adds OpenSpec planning artifacts for the Backstage gear registry.
- Relates to `research-performance-backstage-modes` (Backstage in-app help), `investigate-es9-config-profiles` (the ES-9 rig profile), and `docs/authoring/` (agent-usable reference material).
- No production code, runtime dependencies, hardware integration, scraping, or bundled third-party manuals/data are introduced by this proposal.
