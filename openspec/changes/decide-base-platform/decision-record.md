# Base Platform Decision Record

**Status:** Recommended (2026-07-22), pending the ES-9 timing bench (the one gating hardware spike — see §8).
**Method:** Synthesis of the executed research spikes and decisions (DAW interchange, ES-9 profiles, Pi topology, touchscreen/emulation, Performance/Backstage, router screen, agent control surface, song-bundle manifest) plus the authoring docs. Web/design evidence; hardware-measured items are flagged.

## 1. Summary of the decision

Maybelle 314's base platform is a **Python core + local web UI (Chromium kiosk), on Raspberry Pi OS (64-bit, Bookworm)**, talking to the ES-9 as a class-compliant USB audio device:

- **Language/runtime:** Python for the core (MIDI parsing, clock-following scheduler, MIDI→CV engine, manifest handling), with the timing-critical CV/gate work living in a tight audio-callback filling NumPy buffers.
- **ES-9 I/O:** the ES-9 as a class-compliant USB audio interface via **PortAudio/`sounddevice` (or direct ALSA)** — one duplex stream: DC output buffers = CV/gate/trigger/mod; DC input channels = rack CV selection + Pamela's clock/start/reset. **(Final I/O layer gated on the ES-9 timing bench, §8.)**
- **UI:** a **web/kiosk UI**. The Python process runs a **local web server** (FastAPI + Uvicorn) that serves both surfaces and the **SSE activity stream**; the Pi renders the **Performance** view in **Chromium `--kiosk`**; **Backstage** connects from a laptop browser to the same server.
- **Persistence:** file-based — song bundles (JSON manifest + SMF + assets), config as JSON. No database required.
- **Control surface / agents:** a thin **CLI façade** over the same core (JSON output governed by the manifest's JSON Schema 2020-12); MCP deferred.
- **OS/appliance:** start on **Raspberry Pi OS Desktop (64-bit)** for fast bench validation; target a **Lite + minimal kiosk (labwc + Chromium)** appliance image for production.

This matches the README's Python + browser/kiosk bias and every downstream spec (the web UI emulates 1:1; the router needs a local HTTP/SSE endpoint; the manifest is JSON-Schema-validated; the CLI serves agents).

## 2. Product shape and hard constraints

A Raspberry Pi 5 stored-sequence MIDI/CV controller with two contexts — **Performance** (rack, 5-inch Touch Display 2, 720×1280 portrait) and **Backstage** (laptop config). It **follows** the rack master clock (Pamela's Pro Workout), reads rack CV for selection, and emits pitch CV / gates / triggers / stepped modulation through the ES-9. Authoring is off-device (Bitwig/Ardour → song bundle). Hard constraints: deterministic-enough timing for CV/gate; MIT-compatible/permissive licensing; runtime must run standalone (no Backstage/phone/network dependency); the timing path must never depend on the UI.

## 3. Evaluation criteria

Product fit; ES-9 multichannel DC I/O feasibility; external-clock following; runtime CV handling (quantize/debounce/latch); permissive-license ecosystem; UI emulability (dev without the panel); dev velocity/team familiarity; maintainability; deployment/appliance burden; testing; realtime-timing risk.

## 4. Candidate shortlist and comparison

| Criterion | A: Python core + web/kiosk UI | B: Node/JS core + web UI | C: Rust core + Tauri (native shell) |
| --- | --- | --- | --- |
| MIDI/audio/CV ecosystem (permissive) | **Strong** — Mido, python-rtmidi, sounddevice/PortAudio, NumPy | Moderate — easymidi; weaker DC-audio/CV + numerics | Strong but low-level; smaller high-level ecosystem |
| ES-9 DC multichannel I/O | **Good** via PortAudio/ALSA duplex + NumPy | Weaker for continuous DC buffers | **Best** raw, but most effort |
| UI emulability (dev off-panel) | **1:1** (pure web at 720×1280) | 1:1 (pure web) | Weaker — Tauri native shell ≠ pure browser |
| Realtime timing risk | Moderate (GIL/GC) — mitigated in the audio callback | Moderate (event loop) | **Lowest** |
| Dev velocity / familiarity | **High** | High | Lower (Rust ramp) |
| Fit to README/prior specs | **Best** (Python + web bias, SSE, JSON, CLI) | Partial | Partial |
| Licensing | Permissive throughout | Permissive | Permissive |

## 5. Decision and rationale

**Selected: Candidate A — Python core + local web/kiosk UI.**

- It has the strongest **permissive** MIDI/audio/CV/numerics ecosystem, which is exactly this product's core.
- It **emulates 1:1** as a web UI (the touchscreen spike's decisive point), keeps the Performance/Backstage client-server model natural (one server, two responsive views), and serves the router's **SSE** stream cleanly.
- It matches the **README bias** and every executed spec, minimizing rework.
- **Rejected B (Node):** weaker for continuous DC-audio/CV buffers and numerics; no strong advantage over Python for a mostly-Python-friendly problem.
- **Rejected C (Rust/Tauri):** best raw timing, but higher cost, slower iteration, and a native shell that emulates worse than pure web; timing risk in A is manageable and, if disproven at the bench, addressed by the fallback below.

**Realtime risk & fallback:** Python's GIL/GC is the main risk. It is contained by putting timing in the **PortAudio/ALSA audio callback** (buffer-boundary scheduling; CV/gate resolution = buffer latency, e.g. ~2.7–5.3 ms at 128–256 frames/48 kHz — fine for CV), filling **NumPy** buffers, and keeping Python control/UI off the timing path. **If the ES-9 bench shows Python timing/jitter is inadequate, the fallback is a small Rust/C timing core (audio callback + scheduler) with the Python/web layer unchanged** — the architecture already isolates the timing thread, so this is a contained swap, not a rewrite.

## 6. The stack in detail

- **OS:** Raspberry Pi OS 64-bit (Bookworm); Wayland/labwc, KMS `vc4-kms-v3d`, DSI panel as `DSI-1`. Desktop image first → Lite+kiosk appliance.
- **Core (Python):** MIDI parse (Mido), clock-following scheduler, MIDI→CV engine, manifest load/validate.
- **ES-9 I/O:** one duplex stream (PortAudio/`sounddevice` or ALSA). Outputs → DC CV/gate/trigger/mod (ch 9–16 → outs 1–8). Inputs → rack CV selection + Pamela's clock/start/reset, edge-detected in the callback to derive transport. **(Gated on §8.)**
- **Runtime CV handling:** quantize-to-N-choices with debounce/hysteresis/latch, configured per-control in the manifest.
- **Web layer:** FastAPI + Uvicorn — serves the Performance and Backstage UIs and the **SSE** activity stream; async, and WebSocket-ready if the optional router live-rewire is ever built.
- **UI:** web app; Chromium `--kiosk` (labwc autostart) for Performance; laptop browser for Backstage (which also renders the `docs/authoring/` help and the gear registry).
- **Persistence:** song bundles (JSON manifest + SMF + `assets/`); config as JSON. Optional SQLite later only if an index is needed.
- **Control surface:** thin CLI over the core; JSON output per the manifest schema; risk-tiered safety (read-only broad, file-writes dry-run+diff, hardware-writes explicit-flag). MCP deferred.
- **Dev/test:** pyproject.toml (pip/venv or uv); pytest; synthetic SMF fixtures + curated CC0 set (fixtures spike); UI iterated in Chrome DevTools device mode at 720×1280.
- **Deployment:** systemd units for the runtime and the kiosk; appliance image once the stack is proven.

## 7. Permissive dependency review

| Role | Package | License |
| --- | --- | --- |
| MIDI parse/IO | Mido, python-rtmidi | MIT |
| Audio/CV I/O | sounddevice (+ PortAudio); or pyalsaaudio | MIT / MIT-style; BSD/PSF |
| Numerics (DC buffers) | NumPy | BSD-3 |
| Manifest model/validation | Pydantic; jsonschema (2020-12) | MIT |
| Web server / SSE | FastAPI; Uvicorn (or Flask) | MIT; BSD |
| Config | stdlib json / tomllib; PyYAML (opt) | PSF; MIT |
| Testing | pytest | MIT |

All core dependencies are permissive (MIT/BSD/Apache-2.0/PSF). **Native dependency flagged:** PortAudio (needs the system lib) — MIT-style, fine. **System components** (Chromium, labwc, PipeWire) are OS-level and not linked into our code; their licenses (BSD/GPL) do not affect our app's distribution. No GPL code is linked or vendored into Maybelle.

## 8. Hardware spike precedence (the gating item)

Per the change's "hardware spike precedence" requirement, the **framework choice is recommended on web/design evidence, but the ES-9 I/O layer is finalized only after the ES-9 timing bench**, which is still outstanding (hardware not yet on the bench). The bench must confirm, on a real Pi 5 + ES-9:

- ALSA enumeration of the ES-9 as a 16×16 device; achievable sample rate/buffer;
- CV/gate output timing and jitter under load; input edge-detection accuracy for Pamela's clock/start/reset;
- xrun behavior over a long set; whether PortAudio/`sounddevice` vs direct ALSA vs JACK gives the best determinism.

If Python timing is inadequate, apply the §5 fallback (Rust/C timing core). Everything above this line (language, web/UI, persistence, OS, deps) is not blocked by the bench.

## 9. Assumptions and revisit triggers

Revisit if: the ES-9 bench shows Python timing/jitter is unacceptable (→ Rust/C timing core); the panel/orientation changes; the runtime must generate rack voices/audio (out of current scope); a hard requirement for on-Pi authoring appears; or a needed capability lacks a permissive dependency.

## 10. Implementation impact (first steps after acceptance)

1. Scaffold the Python package (pyproject.toml, `maybelle/` core, `pytest`), CLI entry point.
2. **ES-9 I/O timing spike** on real hardware (§8) — the first implementation task.
3. Manifest model + JSON Schema (from `decide-song-bundle-manifest`) + validator (CLI `validate`).
4. Local FastAPI server + SSE skeleton; minimal Chromium-kiosk Performance shell at 720×1280.
5. Clock-following scheduler + MIDI→CV engine against synthetic fixtures, then hardware.

## 11. Cross-references

Consumes: `research-daw-interchange-options`, `investigate-es9-config-profiles`, `investigate-pi-port-topology`, `research-touchscreen-emulation-and-ux`, `research-performance-backstage-modes`, `research-performance-router-screen`, `investigate-agent-control-surface`, `decide-song-bundle-manifest`, `research-open-test-music-fixtures`, and `docs/authoring/`.
