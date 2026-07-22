# Research Findings: Performance Router Screen

**Method:** Web research (WebSearch/WebFetch) against official/vendor docs + design reasoning against the project's prior decisions.
**Retrieval date:** 2026-07-22.
**Status:** Complete for the web/design scope. Runtime-integration and on-hardware items (activity-stream throughput, timing isolation on the real Pi/ES-9) are called out as follow-ups.

## Intro

The router screen is the flagship Performance view on the Pi 5-inch portrait touchscreen: a diagram of the ES-9, the authored tracks shown against the outputs they drive, and live per-output activity as the sequence plays. This document confirms the ES-9 panel/asset approach, defines the mapping source, chooses the live-activity model and transport, defines the per-signal visual language and portrait layout, and evaluates an **optional, explicitly-gated live re-wire** (drag track→port) that is deliberately **not a requirement**.

---

## Requirement 1 — ES-9 diagram and asset approach

**Panel layout (confirmed).** The ES-9 front panel carries **8× 3.5mm TS DC-coupled outputs (~±10V)**, **14× 3.5mm TS DC-coupled inputs (~±10V)**, **2× 1/4" TRS AC-coupled main outs**, **1× 1/4" TRS DC-coupled headphone out + volume pot**, and **S/PDIF in/out (TOSlink)**, on a **16HP** panel (~81 × 128.5 mm) ([ES-9 product page](https://www.expert-sleepers.co.uk/es9.html), [ES-9 v1.3 manual](https://www.expert-sleepers.co.uk/downloads/manuals/es9_user_manual_1.3.pdf)). Maybelle's outputs are the **8 DC outs** (DAW ch 9–16 per the ES-9 profile research); the **14 DC ins** are rack CV/clock into Maybelle.

**Asset decision: self-drawn SVG, per-jack addressable, MIT-clean.** Expert Sleepers product imagery is all-rights-reserved and cannot be bundled, but the **factual jack layout is not copyrightable**. Draw a project-authored **SVG schematic** of the panel with one addressable element per jack (`id="out-1"…"out-8"`, `in-1"…"in-14"`), which (a) is MIT-clean, (b) scales crisply on the ~290 ppi panel, and (c) lets activity light individual jacks and cables. Scope the rendered jacks to the **active ES-9 profile** (which jacks are in use, coupling), and structure the SVG so expander/ADAT outputs can be added later. This satisfies the project's permissive-licensing requirement.

## Requirement 2 — Track-to-output mapping display

**Source of truth:** track→output associations are read from the **song-bundle manifest / ES-9 profile channel map** (the manifest already owns the channel map per the DAW-interchange research). Render each mapping as a label/link on the corresponding output jack; keep **unmapped jacks visually quiet** (dimmed, no label) so the eye goes to what's patched.

**Read-only by default.** In Performance the router **displays** the mapping; routing edits live in Backstage. See the *Optional live re-wire* section for a guarded exception that remains non-required.

## Requirement 3 — Live activity model, decoupled from timing

**Activity data (per output):** `gate` (on/off, held), `trigger` (brief pulse), `pitch/CV` (level/value), `stepped-mod` (discrete value). Optionally per input later.

**Decoupling (non-negotiable):** the real-time CV/gate thread must never block on or be paced by the UI. Pipeline:

```
audio/CV callback  →  lock-free enqueue (event/state)      [real-time thread; never renders]
                 →  publisher coalesces to ~30–60 Hz snapshots   [normal-priority thread]
                 →  SSE stream
                 →  browser: rAF-batched SVG update              [~60 fps / 16.67 ms frame]
```

- The callback only **enqueues**; a separate normal-priority thread coalesces and publishes, so a slow/dead UI cannot perturb output. This is the same runtime-independence principle used for the Bluetooth and Performance/Backstage surfaces.
- The browser batches incoming events per animation frame with **`requestAnimationFrame`**, the standard way to throttle high-frequency updates to the display refresh and avoid flooding the main thread ([MDN requestAnimationFrame](https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame)).
- **Transient stretch:** a 1 ms trigger must still be *seen*, so brief events are held for a **minimum visible duration** (~50–100 ms / a few frames) in the view layer, independent of the true event length.

**Transport decision: Server-Sent Events (SSE).** Activity is **one-way** runtime→UI, which is exactly SSE's sweet spot — a long-running HTTP stream of named text events, built-in browser support, auto-reconnect, low overhead, no client-side connection-state management; WebSocket's bidirectionality is unnecessary for pure telemetry ([Ably: WebSockets vs SSE](https://ably.com/blog/websockets-vs-sse), [freeCodeCamp: SSE vs WebSockets](https://www.freecodecamp.org/news/server-sent-events-vs-websockets/), [SoftwareMill: SSE vs WebSockets](https://softwaremill.com/sse-vs-websockets-comparing-real-time-communication-protocols/)). One SSE stream serves **both** the local Performance router and a remote Backstage monitor (fits the Performance/Backstage client-server model). *If* the optional live re-wire (client→server commands) is built, add a small command endpoint (POST) alongside SSE, or upgrade the router link to WebSocket — decided only if that option is taken.

**v1 scope:** show activity on the **8 outputs**; the 14 inputs are secondary and can be added once the output view is proven.

## Requirement 4 — Signal-type visual language and portrait layout

**Per-signal representation (never color-alone — pair with motion/shape/label):**
| Signal | Representation |
| --- | --- |
| Gate | jack/indicator **held lit** while high; clear on/off state |
| Trigger | brief **pulse/flash** (min ~50–100 ms visible), distinct from a held gate |
| Pitch / CV | **level or numeric value** (small bar/ring or value readout) |
| Stepped mod | **discrete value/step** indicator |

Distinguishing "held gate" vs "trigger pulse" vs "CV level" at a glance from ~1 m is the core legibility requirement; colorblind operators must read state from shape/motion, not hue (per the touchscreen accessibility finding).

**Portrait layout (720×1280):** the tall 16HP module maps naturally to portrait — a large **central ES-9 diagram**, a compact **top status strip** (song/bank, transport, clock-lock), and a **bottom manual-override/action row** in thumb reach. The router is a primary Performance screen. Touch-target and type sizing follow the mm-anchored rules from the touchscreen research (~100 device-px min target at this density).

## Optional: live re-wire (drag track → ES-9 port) — NOT a requirement

**Feasibility:** established UI patterns exist — SVG **bezier "cables"** dragged from a source to a target, snapping to jacks, as in node/patchbay editors ([patchbay-js](https://github.com/HelgeSverre/patchbay-js), [ned node editor](https://github.com/depuits/ned), [Cables.gl connections](https://cables.gl/docs/0_howtouse/ui_walkthrough/ui_walkthrough)). Touch-dragging from a track to an output jack on the portrait panel is practical (the ~100 px jack targets are drag-droppable).

**Reconciliation with "Performance is a safe surface."** A prior decision reserved routing edits for Backstage and made Performance read-only. Live re-wire is best framed **not** as configuration but as a **performance gesture** — repatching mid-set, like physically repatching a modular. The consistent way to allow it without breaking the safety principle is the **blind/edit-mode gate** already adopted for Performance/Backstage: keep the router **read-only by default**, behind an explicit momentary **"patch" sub-mode** the operator must enter. In patch mode:
- drag a track to an output jack to **re-assign live routing**, with immediate visual confirmation (the cable moves) and **one-tap undo**;
- persistence to the manifest is **explicit** (save), so a live experiment doesn't silently rewrite the show;
- exiting patch mode returns to the locked read-only router.

**Recommendation:** keep this **optional and out of v1 scope**. Design the SVG (addressable jacks) and the transport (allow a command channel) so the option remains open, but do not require it. If built, it reuses the Performance/Backstage safety-gate model rather than adding a new one.

## Requirement 5 — Decision evidence handoff

- **Recommended:** self-drawn per-jack SVG (MIT); mapping from the manifest/ES-9 profile (read-only default); activity via a **decoupled publisher → SSE → rAF-batched** pipeline with transient-stretch; per-signal visual language; portrait layout with the diagram as a primary view. Live re-wire is an **optional, guarded** future extension.
- **For `decide-base-platform`:** the runtime must expose a **local HTTP/SSE endpoint** (the runtime is the server; Performance is a local client, Backstage a remote client) and a **non-realtime publisher thread** fed by a lock-free queue from the CV/gate engine. This is a concrete new requirement on the runtime/UI transport.

### Follow-up spikes before implementing
1. **Activity-stream bench validation:** confirm SSE throughput + coalescing keep up and that publishing does not perturb CV/gate timing on the real Pi/ES-9 (blocked: needs runtime + hardware).
2. **SVG authoring:** draw the per-jack ES-9 schematic tied to the ES-9 profile.
3. **Manifest fields:** the track↔output link the router reads (feeds the manifest-format proposal).
4. **(Optional) patch sub-mode:** command channel + undo + explicit save, only if the live re-wire option is taken.

## Sources

- [ES-9 product page](https://www.expert-sleepers.co.uk/es9.html)
- [ES-9 v1.3 user manual (PDF)](https://www.expert-sleepers.co.uk/downloads/manuals/es9_user_manual_1.3.pdf)
- [Ably — WebSockets vs Server-Sent Events](https://ably.com/blog/websockets-vs-sse)
- [freeCodeCamp — SSE vs WebSockets](https://www.freecodecamp.org/news/server-sent-events-vs-websockets/)
- [SoftwareMill — SSE vs WebSockets](https://softwaremill.com/sse-vs-websockets-comparing-real-time-communication-protocols/)
- [MDN — Window.requestAnimationFrame()](https://developer.mozilla.org/en-US/docs/Web/API/Window/requestAnimationFrame)
- [patchbay-js — draggable rope/cable connectors](https://github.com/HelgeSverre/patchbay-js)
- [ned — SVG node editor](https://github.com/depuits/ned)
- [Cables.gl — dragging/reconnecting cables](https://cables.gl/docs/0_howtouse/ui_walkthrough/ui_walkthrough)
