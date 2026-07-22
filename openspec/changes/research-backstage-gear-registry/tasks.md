## 1. Data Sources and Legitimacy

- [x] 1.1 Confirm ModularGrid access constraints (no public API, scraping not permitted, no data dumps; premium "Unicorn" accounts can export their own racks as XML/JSON).
- [x] 1.2 Define the legitimate population paths: manual entry, user's own ModularGrid export import, and URL-as-link — no scraping.
- [ ] 1.3 Determine the fields/stability of the ModularGrid Unicorn export format. (blocked: needs a sample export from a Unicorn account.)

## 2. Registry Data Model

- [x] 2.1 Define per-module fields (manufacturer, name, hp, function/tags, manual_url, local_manual_path, modulargrid_url, notes).
- [x] 2.2 Support multiple racks / ModularGrid URLs.
- [x] 2.3 Define the registry's relationship to the ES-9 rig profile (distinct; optional cross-reference; not required for playback).

## 3. Manuals Access

- [x] 3.1 Define manual access as link-first to official sources.
- [x] 3.2 Define an optional user-initiated local cache for personal offline use, never bundled/redistributed/shared.
- [ ] 3.3 Define how manual URLs are discovered when not user-supplied (best-effort lookup vs manual-entry only). (open: needs a lookup strategy decision.)

## 4. Integration

- [x] 4.1 Define how the registry/manuals serve in-app module lookups and agent assistance (consistent with Backstage in-app help and `docs/authoring/`).
- [x] 4.2 Confirm the licensing/ToS boundary: no scraping, no manual/data redistribution; user-provided data + links + personal cache only.

## 5. Decision Handoff

- [x] 5.1 Recommend the population paths, data model, and manuals-access model.
- [x] 5.2 List follow-up work before implementing the registry/importer/cache.
- [ ] 5.3 Feed the registry model into the Backstage editor design. (pending: Backstage editor implementation planning.)

## 6. Verification

- [x] 6.1 Review the output against every `backstage-gear-registry-research` requirement.
- [x] 6.2 Run OpenSpec validation or status checks for the completed change. (`openspec validate` passes, 2026-07-22)
