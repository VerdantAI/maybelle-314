## 1. Precedent Research

- [x] 1.1 Research edit-vs-perform separation in performance/show-control software (QLab blind/live, lighting-console blind editing, cue systems' show/edit modes, Ableton modal edit states).
- [x] 1.2 Research single-core / multiple-responsive-frontend and kiosk-plus-remote-admin architectures.
- [x] 1.3 Extract the safety patterns (staged/blind editing, locked show surface) relevant to protecting live output.

## 2. Architecture Decision

- [x] 2.1 Compare two-simultaneous-views (client/server), two-full-modes (switched), and hybrid options.
- [x] 2.2 Decide the recommended architecture and record the rationale. (Decided in design.md: one core, two role-specific responsive views.)
- [x] 2.3 Define whether Performance and Backstage can run simultaneously and the primary (pre-show vs live-connected) Backstage workflow.
- [x] 2.4 Define the safety-mode gate: how Backstage edits are staged and applied without disrupting live output.
- [x] 2.5 Confirm Performance + runtime remain fully functional with no Backstage/phone client connected.

## 3. Surface Responsibilities

- [x] 3.1 Define the responsibilities of Performance (touchscreen), Backstage (laptop), and the Bluetooth/phone subset.
- [x] 3.2 Map each surface to the shared song-bundle manifest and identify shared vs surface-specific logic.
- [x] 3.3 Reconcile with `research-touchscreen-emulation-and-ux` and `research-bluetooth-control-channel` so the three surfaces form one coherent model.
- [x] 3.4 Define that Backstage surfaces accessible authoring help/diagrams (MIDI→CV model, ES-9 output map, Bitwig/Ardour setup) sourced from `docs/authoring/`.

## 4. Decision Handoff

- [x] 4.1 Recommend the architecture and client/server model for `decide-base-platform`.
- [x] 4.2 List follow-up spikes before implementing the runtime server, the Backstage editor, or the mode state machine. (Captured in design.md Open Questions.)
- [ ] 4.3 Feed findings into the future song-bundle/manifest-format proposal. (blocked: that proposal does not exist yet.)

## 5. Verification

- [x] 5.1 Review the research output against every `performance-backstage-modes-research` requirement.
- [x] 5.2 Run OpenSpec validation or status checks for the completed change. (`openspec validate` passes, 2026-07-22)
