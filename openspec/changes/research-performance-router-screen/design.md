## Context

Performance mode runs on the official Raspberry Pi Touch Display 2 (5-inch, 720x1280 portrait) at the rack. Prior research scoped the Performance surface to glanceable status plus manual override, with CV driving real-time selection and Backstage (laptop) owning configuration. The router screen is the flagship Performance view: it makes the software track-to-ES-9 routing physically legible and shows live output activity so the operator can confirm the patch is correct and firing during a set.

The ES-9 front panel is well defined and small, which makes an accurate diagram practical:
- **8x 3.5mm TS DC-coupled outputs (~±10V)** — Maybelle's CV/gate/trigger/mod outputs (DAW channels 9-16 per the ES-9 profile research).
- **14x 3.5mm TS DC-coupled inputs (~±10V)** — CV/clock inputs from the rack.
- **2x 1/4" TRS AC-coupled main outs** (not used for CV), **1x 1/4" TRS DC-coupled headphone out** + volume pot, **S/PDIF in/out (TOSlink)**.
- **16HP** panel (~81mm wide x ~128.5mm tall, 3U) ([ES-9 product page](https://www.expert-sleepers.co.uk/es9.html), [ES-9 v1.3 manual](https://www.expert-sleepers.co.uk/downloads/manuals/es9_user_manual_1.3.pdf)).

The 8 DC outputs are the primary subject (that is where Maybelle emits); the 14 inputs are secondary context (rack CV/clock into Maybelle). The mapping itself comes from the song-bundle manifest / ES-9 profile channel map. Live activity must come from the runtime's CV/gate engine.

Initial research suggests:
- A copyrighted product photo of the ES-9 cannot be bundled (Expert Sleepers imagery is all-rights-reserved), but the **factual panel layout is not copyrightable**, so a self-drawn SVG schematic with one addressable element per jack is both MIT-clean and better for the job (jacks can light up, carry labels, and scale crisply on the high-DPI portrait panel).
- The runtime timing thread must not depend on or be blocked by UI rendering (the standing principle from the Bluetooth and Performance/Backstage research). Activity is therefore a **one-way state/event stream** the UI subscribes to at a bounded refresh, never a call path the audio/CV thread waits on.

## Goals / Non-Goals

**Goals:**
- Confirm the ES-9 panel layout and lock the diagram's scope (8 DC outs primary; 14 ins secondary; expanders later).
- Decide a self-drawn, MIT-clean, per-jack-addressable SVG asset approach.
- Define how the manifest/ES-9-profile mapping is rendered on the diagram.
- Define the live per-output activity model and its decoupling from real-time timing.
- Define the visual language per signal type and the portrait-layout fit.
- Feed `decide-base-platform` on the runtime-to-UI live-update path.

**Non-Goals:**
- Implement the router screen, the SVG, or the activity stream.
- Design routing *editing* (that is Backstage).
- Finalize the manifest schema or the visual design system.
- Model every expander/ADAT output in v1.

## Decisions

### Self-drawn SVG ES-9 diagram with addressable jacks (MIT-clean)

The diagram is a project-authored SVG schematic of the ES-9 panel — jack positions, labels, and a per-jack element id — not a copyrighted product photo.

Rationale: the layout is factual (not copyrightable), SVG scales crisply on the ~290 ppi panel, and addressable per-jack elements let activity light individual jacks. Bundling a product photo would violate the project's permissive-licensing requirement.

Alternatives considered:
- Product photo overlay: most "realistic," but all-rights-reserved and not addressable for activity.
- A generic grid of channels (no panel likeness): simplest, but loses the physical "which jack" recognition the operator needs.

### Mapping is read from the manifest / ES-9 profile

Track-to-output associations are read from the song-bundle manifest and the ES-9 profile channel map and rendered as labels/links on the corresponding output jacks. The router screen displays this mapping; it does not edit it.

Rationale: the manifest already owns the channel map (DAW-interchange research), and routing edits belong to Backstage; Performance stays a safe read/monitor surface.

Alternatives considered:
- Let the router edit mappings live: convenient, but violates the Performance-safe-surface principle and duplicates Backstage.

### Live activity is a decoupled one-way stream

Per-output activity (gate on/off, trigger pulse, CV level, stepped-mod value) is published by the runtime as a one-way state/event stream that the UI samples at a bounded UI refresh (event-driven or ~30-60 fps). The real-time CV/gate thread never blocks on, or is paced by, the UI.

Rationale: the instrument's timing must be independent of rendering; a lagging or crashed UI must not perturb output. This matches the runtime-independence principle established for the other surfaces.

Alternatives considered:
- UI reads engine state synchronously: simplest wiring, but couples timing to rendering — unacceptable.
- Log-then-replay activity: fine for review, but not the live "is it firing now" the screen exists for.

### Distinct visual language per signal type, not color-alone

Each output's activity is shown with a representation matched to its signal type — e.g. a gate as a lit/held state, a trigger as a brief pulse/flash, pitch/CV as a level or value, stepped modulation as a discrete value — and always pairs color with motion/shape/label so state is not encoded by color alone.

Rationale: an operator glancing from a meter away must distinguish "held gate" from "trigger pulse" from "CV level," and colorblind operators must read state (the touchscreen accessibility finding).

### Portrait layout fit; router is a primary Performance view

The tall ES-9 panel (~81x128.5mm) maps naturally onto the 720x1280 portrait canvas as a large central diagram, with a compact status strip and manual-override controls around it. The router is one of the primary Performance screens.

Rationale: the module's portrait-ish aspect suits the portrait panel, and the diagram is the most information-dense glanceable view for confirming the patch during a set.

## Risks / Trade-offs

- Activity rendering competes with runtime timing -> Mitigation: strict one-way decoupled stream, bounded UI refresh, UI never in the timing path.
- The 8-output diagram cannot represent expander/ADAT outputs -> Mitigation: v1 scopes the ES-9 panel's 8 DC outs (+14 ins as context); design the asset so expanders can be added later.
- A too-literal panel likeness raises IP concerns -> Mitigation: schematic/stylized self-drawn SVG of the factual layout, no product photo or branding.
- Too much visual activity becomes noise -> Mitigation: match representation to signal type, prioritize the mapped/active outputs, keep unmapped jacks quiet.
- Mapping display drifts from the actual runtime routing -> Mitigation: source both the mapping and the activity from the running system, not a separate copy.

## Migration Plan

No runtime migration is required because this change adds planning artifacts only. Output feeds `decide-base-platform` (the runtime-to-UI live-update path), the Performance UI work, and the manifest/ES-9-profile schemas.

Rollback is limited to removing this OpenSpec change before implementation.

## Open Questions

- What exactly does an output's activity representation look like per signal type (gate vs trigger vs pitch CV vs stepped modulation), and what refresh/observability is enough to feel live without noise?
- What is the transport for the activity stream from the runtime to the local UI (in-process event bus, WebSocket/SSE from a local server, shared memory), and how does it relate to the Performance/Backstage client-server model?
- Should the 14 inputs (rack CV/clock into Maybelle) also show activity, or only the 8 outputs in v1?
- How are expander/ADAT outputs represented when present, and how is the diagram scoped to the active ES-9 profile?
- Does tapping a jack do anything in Performance (show mapped-track detail, or a guarded manual override), or is the screen purely read-only?
- How is the SVG asset kept in sync with the ES-9 profile (channel count, coupling, which jacks are in use)?
