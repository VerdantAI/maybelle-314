## 1. ES-9 Panel and Asset

- [x] 1.1 Confirm the ES-9 front-panel jack layout and labels (8 DC outs, 14 DC ins, 2 AC main outs, headphone + pot, S/PDIF) from the official manual/product page.
- [x] 1.2 Decide the asset approach: a self-drawn, MIT-clean SVG schematic with one addressable element per jack; confirm no copyrighted product imagery is used.
- [x] 1.3 Define how the SVG scopes to the active ES-9 profile (channel count, coupling, in-use jacks) and how expanders/ADAT could be added later.

## 2. Mapping Display

- [x] 2.1 Define how track-to-output associations are read from the song-bundle manifest / ES-9 profile channel map.
- [x] 2.2 Define how mappings are rendered on the diagram (labels/links on output jacks) and what happens for unmapped jacks.
- [x] 2.3 Confirm the router displays mapping only; routing edits remain in Backstage. (Read-only default; optional gated live-re-wire sub-mode documented as non-required.)

## 3. Live Activity Model

- [x] 3.1 Define per-output activity data: gate on/off, trigger pulse, pitch/CV level, stepped-modulation value.
- [x] 3.2 Define the one-way activity stream from the runtime to the UI and its decoupling from the real-time CV/gate thread (bounded UI refresh, UI never in the timing path).
- [x] 3.3 Choose/compare the activity-stream transport (in-process event bus, local WebSocket/SSE, shared memory) and relate it to the Performance/Backstage client-server model. (Chosen: SSE.)
- [x] 3.4 Decide whether the 14 inputs show activity in v1 or only the 8 outputs. (v1 = 8 outputs; inputs deferred.)

## 4. Visual Language and Layout

- [x] 4.1 Define the visual representation per signal type (gate/trigger/pitch/stepped-mod), never color-alone.
- [x] 4.2 Define the 720x1280 portrait layout: central ES-9 diagram plus status strip and manual-override controls.
- [x] 4.3 Decide any tap interaction (mapped-track detail, guarded override) versus read-only. (Read-only default; optional guarded patch sub-mode.)

## 5. Decision Handoff

- [x] 5.1 Recommend the asset approach, mapping-source, and activity-stream design.
- [x] 5.2 List follow-up spikes before implementing the SVG, the activity stream, or the screen.
- [x] 5.3 Feed the runtime-to-UI live-update path into `decide-base-platform` and the manifest/ES-9-profile schemas. (Documented as a runtime requirement: local HTTP/SSE endpoint + non-realtime publisher.)

## 6. Verification

- [x] 6.1 Review the research output against every `performance-router-screen-research` requirement.
- [x] 6.2 Run OpenSpec validation or status checks for the completed change. (`openspec validate` passes, 2026-07-22)

## Blocked / needs follow-up

- Activity-stream bench validation (SSE throughput + timing isolation on the real Pi/ES-9) — blocked: needs runtime + hardware.
- SVG asset authoring, manifest track↔output field, and the optional patch sub-mode — implementation-time follow-ups.
