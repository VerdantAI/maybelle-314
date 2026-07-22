## Context

Backstage (the laptop configuration surface) is where a user manages their rig and where in-app help lives. A natural extension is a **gear registry**: a list of the modules in the rack, with quick access to each module's manual, so the operator or an assisting agent can answer "what does this module do / how do I set it" without leaving the tool. Users want to populate it by listing modules manually or by pointing at their ModularGrid rack(s).

Two external constraints shape the design:

- **ModularGrid has no public API and does not permit scraping**, and provides no data dumps; it paused its own API experiment over EU copyright-reform legal uncertainty. The one legitimate structured export is that **premium ("Unicorn") accounts can export the modules and positions of their own racks as XML/JSON** ([ModularGrid forum — "Module API or data dump?"](https://modulargrid.net/e/forum/posts/index/7901), [ModularGrid](https://modulargrid.net/)). So Maybelle must not fetch/parse ModularGrid pages; it can accept a user-provided export and store rack URLs as links.
- **Manuals are copyrighted** manufacturer PDFs. Maybelle can **link** to official manuals and, at the user's request, **cache a copy for that user's own offline use**, but must not bundle or redistribute them.

Both constraints keep the feature aligned with the project's permissive-licensing requirement.

## Goals / Non-Goals

**Goals:**
- Define legitimate ways to populate the registry (manual entry, user's own ModularGrid export, URL bookmarks).
- Define the gear-registry data model and manuals-access model.
- Keep manuals and ModularGrid data as links / user-provided / personal-cache — never redistributed.
- Define how the registry serves in-app lookups and agent assistance, and its relation to the ES-9 profile.

**Non-Goals:**
- Scrape ModularGrid or build a ModularGrid API client.
- Bundle or redistribute manuals or ModularGrid data.
- Auto-identify modules or build a module database.
- Implement the registry, importer, or manual cache.

## Decisions

### Populate the registry three ways, manual entry first

The registry can be populated by (1) **manual module entry**, (2) **importing the user's own ModularGrid rack export (XML/JSON)**, and (3) **storing one or more ModularGrid rack URLs as reference links**. Manual entry always works and has no external dependency; the export gives structured module lists for users who have it; URLs are convenience bookmarks.

Rationale: this satisfies "list modules or point to ModularGrid URLs" while respecting that ModularGrid has no API and forbids scraping.

Alternatives considered:
- Scrape ModularGrid rack pages: rejected — not permitted, legally risky, and against the project's principles.
- Require a ModularGrid account/export: rejected as the only path — many users lack Unicorn; manual entry must stand alone.

### No scraping; URLs are links, structured data is user-provided

Maybelle never fetches or parses ModularGrid content. A ModularGrid URL is stored as an openable **link**; structured module data comes only from the **user's own export file** or manual entry.

Rationale: honors ModularGrid's terms and the EU-copyright caution that led them to pause their own API.

### Manuals: link first, optional personal offline cache, never redistribute

Each module may carry an **official manual URL** (entered by the user or best-effort looked up). "Pre-loaded for easy access" is implemented as: open the official link, and **optionally cache a copy locally for the user's own offline use** at the user's request. Cached manuals are the user's personal copies — never committed to the repo, bundled in a release, or shared between users.

Rationale: gives fast/offline access without redistributing copyrighted material.

Alternatives considered:
- Bundle a manual library: rejected — redistribution of copyrighted PDFs.
- Link-only, no cache: safe but loses offline access at the rack; a user-initiated personal cache is the compromise.

### Registry data model

A registry is a list of modules, each with fields such as: `manufacturer`, `name`, `hp`, `function`/tags, `manual_url`, optional `local_manual_path` (personal cache), `modulargrid_url` (bookmark), and free-form `notes`. Multiple racks/ModularGrid URLs are supported. The registry is Backstage-managed rig reference data.

### Relationship to the ES-9 profile

The gear registry is broader **rack inventory/reference**; the **ES-9 rig profile** (`investigate-es9-config-profiles`) is the **I/O/calibration rig**. They are distinct and the registry is **not required for playback**; they may cross-reference (e.g. note which modules the ES-9 patches to), but the registry stays an optional Backstage convenience.

### Serves in-app lookups and agent assistance

The registry and any cached manuals are a **context source** for in-app module lookups and for agents assisting the user (consistent with the Backstage in-app-help requirement and `docs/authoring/`). An agent can answer "how does module X work" from the user's registered manual rather than guessing.

## Risks / Trade-offs

- Users expect automatic ModularGrid import -> Mitigation: clearly offer manual entry + user-export import + URL bookmark, and explain no-scraping up front.
- Manual caching drifts into redistribution -> Mitigation: personal-use cache only, never committed/bundled/shared; link to official sources.
- Manual URLs rot or are unknown -> Mitigation: allow manual entry/override; treat lookup as best-effort.
- Registry scope creeps toward a full module database -> Mitigation: keep it user-scoped rig reference, not a global DB.
- ModularGrid export format changes -> Mitigation: import is best-effort and versioned; manual entry is the stable fallback.

## Migration Plan

No runtime migration is required; this change adds planning artifacts only. Output feeds the Backstage editor/help design and coordinates with the ES-9 profile.

Rollback is limited to removing this OpenSpec change before implementation.

## Open Questions

- What fields does the ModularGrid Unicorn XML/JSON export actually contain, and how stable is that format?
- How are manual URLs discovered when the user doesn't supply them (best-effort manufacturer lookup vs manual entry only)?
- Where/how is the personal manual cache stored (on the laptop, on the Pi, size limits), and how do agents access it?
- Does the registry cross-reference the ES-9 profile (which modules are patched to which outputs), or stay fully independent in v1?
- Should the registry support multiple racks/ModularGrid URLs as first-class, and how are duplicates reconciled?
