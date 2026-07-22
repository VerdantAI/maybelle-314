## Context

Maybelle 314 is a Raspberry Pi 5 stored-sequence MIDI/CV controller. Two usage contexts want different interfaces:

- **Performance** — at the rack, on the official Raspberry Pi Touch Display 2 (5-inch, 720x1280 portrait). Glanceable, finger-operated, real-time, must stay robust and never be disrupted. Prior research (`research-touchscreen-emulation-and-ux`) scoped this surface to status + manual override + panic, with CV driving real-time selection.
- **Backstage** — configuration of triggers, channel maps, banks, ES-9 output assignments, calibration, and CV-selection/debounce behavior. Done on a computer/laptop with a standard-size interface and a keyboard, not on the 5-inch panel. This is essentially editing the Maybelle song-bundle manifest (the DAW-interchange research recommends a bundle = Standard MIDI File(s) + a project-owned manifest that carries exactly this config).

The question is whether Performance and Backstage are **two simultaneous views** of one running system or **two full modes** the tool switches between. Because Performance runs on the Pi and Backstage on a laptop, they are naturally on different machines, which makes "simultaneous" a real option rather than a screen toggle.

Initial research suggests:
- Performance/show-control tools consistently separate an editing context from a running/show context, and make the running context safe. QLab edits Light cues "in blind" (changes are not reflected on stage until the cue runs) and edits "live" only via a separate Light Dashboard ([QLab 5 Light Cues docs](https://qlab.app/docs/v5/lighting/light-cues/)). CuryCue (a TouchDesigner cue system) has a **Show Mode** — simplified, "safe mode to prevent accidental changes during a performance" — and an **Edit Mode** with full authoring, used in planning/rehearsal ([CuryCue](https://github.com/netzzy/curycue)). Lighting consoles have the same blind-vs-live split, and Ableton's MIDI Map Mode is an explicit modal edit state.
- Kiosk/UI architecture supports one system serving multiple responsive frontends: a browser UI written to responsive standards runs on different devices in kiosk mode, and the micro-frontend pattern composes independently-built frontends over shared services ([responsive kiosk UI](https://redswimmer.medium.com/how-to-design-a-responsive-kiosk-user-interface-part-1-b013c77d3e20), [micro frontend](https://en.wikipedia.org/wiki/Micro_frontend)). Remote administration of a kiosk device alongside its local UI is a standard pattern.
- The `investigate-agent-control-surface` research already recommends a **shared core library with thin façades** (CLI now, MCP later). A Performance view and a Backstage view are two more façades over that same core, which argues against building two separate apps.

## Goals / Non-Goals

**Goals:**
- Decide whether Performance and Backstage are simultaneous views, switched modes, or a hybrid.
- Ground the decision in performance/show-control and kiosk-architecture precedents.
- Define how Backstage edits are staged/applied without disrupting live output, and how Performance survives Backstage disconnection.
- Define each surface's responsibilities and how they map to the shared song-bundle manifest.
- Feed the base-platform UI-framework and runtime client/server decisions.

**Non-Goals:**
- Build either UI, the runtime server, or a mode state machine.
- Finalize the manifest schema (a separate change) or the visual design system.
- Decide the exact network/auth transport (coordinated with `research-bluetooth-control-channel` and base-platform).

## Decisions

### Answer: one core, two role-specific responsive views — not two apps, not a screen toggle

Performance and Backstage are **two role-specific frontends over one shared core/runtime**, delivered as a responsive UI: a compact portrait layout for the Performance touchscreen and a standard desktop layout for Backstage on a laptop. They are **not** two separate applications, and **not** a single-screen mode toggle.

Rationale: they run on different machines and want different form factors, so a responsive multi-frontend over one core (the façade model from `investigate-agent-control-surface`) fits directly and avoids duplicated logic and drift.

Alternatives considered:
- Two separate apps: clearer separation, but duplicates the domain model and risks config/runtime divergence.
- One app with an in-screen Performance/Backstage toggle: simple, but forces the rack touchscreen and the laptop to share one layout and conflates the safety boundary.

### They can run simultaneously (client/server), and Backstage is primarily pre-show

The runtime is the single source of truth; the Performance view runs locally on the Pi, and Backstage connects as a client. They **can be live at the same time** (Backstage monitors/edits while Performance shows status), but the primary Backstage workflow is **pre-show/offline** configuration of the bundle manifest.

Rationale: simultaneous connection enables monitoring and last-minute tweaks, while the common case (build the show, then perform) is served by editing the bundle off the rack.

Alternatives considered:
- Strictly offline Backstage (edit bundle, redeploy, no live link): simplest and safest, but loses live monitoring and quick fixes.
- Always-simultaneous with live-applied edits: flexible, but dangerous without a safety gate (below).

### Keep an explicit safety-mode gate over live output (blind/staged editing)

Independent of the view split, there is an explicit operational gate: while a performance is live, Backstage edits are **staged (edited "in blind") and applied transactionally**, never mutating live output mid-cue. Performance itself is a locked, minimal, safe surface.

Rationale: this is the consistent lesson from QLab blind/live and CuryCue show/edit — the running context must be protected from accidental change.

Alternatives considered:
- Live-apply every Backstage edit immediately: fastest feedback, but a mistap can disrupt a live set.
- No mode concept at all: simplest, but unsafe and contrary to every show-control precedent.

### Performance must survive Backstage disconnection

The Performance surface and the runtime remain fully functional with no Backstage client connected (and with Bluetooth/network off). Backstage and the phone surface are optional clients, not dependencies.

Rationale: the rack instrument cannot depend on a laptop being present; this mirrors the runtime-independence principle from `research-bluetooth-control-channel`.

### Surface responsibilities and the manifest

- **Performance (Pi touchscreen):** glanceable status, manual override, panic — reflects CV-driven state.
- **Backstage (laptop, standard size):** the full editor for the song-bundle manifest — triggers, channel→ES-9 maps, banks, calibration, CV-selection/debounce — plus live monitoring when connected.
- **Bluetooth/phone (`research-bluetooth-control-channel`):** a lightweight subset of Backstage for at-venue quick config, not the full editor.
All three read/write the same shared song-bundle manifest model; Backstage is its primary editor.

## Risks / Trade-offs

- Simultaneous editing disrupts a live performance -> Mitigation: the blind/staged safety gate; Performance is a locked surface.
- Backstage becomes a hard dependency -> Mitigation: runtime + Performance fully functional standalone; Backstage is an optional client.
- Responsive UI compromises both form factors -> Mitigation: distinct layouts per role over shared logic, not one layout stretched.
- Overlap/confusion with the phone surface -> Mitigation: define phone as an explicit lightweight subset of Backstage, one manifest model.
- Client/server adds runtime complexity -> Mitigation: evaluate the transport/server against the base-platform decision; keep the core UI-agnostic.

## Migration Plan

No runtime migration is required because this change adds planning artifacts only. Research output should feed `decide-base-platform` (UI framework + client/server model), the Performance and phone surface changes, and the future manifest-format proposal.

Rollback is limited to removing this OpenSpec change before implementation.

## Open Questions

- Is live-connected Backstage in scope for v1, or is v1 offline-config + redeploy with Performance-only at the rack?
- What transport carries the Backstage client link (local web server over Wi-Fi/Ethernet, and how does it relate to the Bluetooth channel)?
- What is the exact "apply" transaction for staged edits, and can any edits be safely live during a performance (e.g. brightness) versus strictly staged?
- How much of the manifest editor is shared code between the Backstage (full) and phone (subset) surfaces?
- Does Backstage need multi-user or single-operator assumptions, and any auth beyond the base-platform/Bluetooth decisions?
