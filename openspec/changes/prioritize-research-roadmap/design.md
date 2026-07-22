## Context

Maybelle 314 currently has one base-platform decision change and seven supporting research tracks. Some research can start immediately with web searches and source review. Other work is blocked on hardware access, bench testing, a DAW host machine, artist/license outreach, or unavailable deferred hardware such as Assimil8or.

The roadmap needs to preserve momentum without pretending all research has the same urgency. The base-platform decision is the architectural center: it should receive early evidence from DAW interchange, Pi/ES-9 port topology, ES-9 profile validation, and package/runtime research. Agent control surface, open test fixtures, and live trigger routing are important, but they mostly refine tooling, testing, and future feature boundaries after the first platform constraints are known.

## Goals / Non-Goals

**Goals:**
- Prioritize active research efforts by how strongly they block the base platform decision.
- Flag research that can begin with web-only work.
- Identify research channels that require hardware, DAW host access, source-code review, license review, artist outreach, or live-show domain research.
- Persist an initial schedule that agents and humans can follow later.
- Keep the schedule connected to existing OpenSpec change names.

**Non-Goals:**
- Complete any research item.
- Replace the detailed tasks inside each existing OpenSpec change.
- Choose the base platform.
- Commit to implementation order after research is complete.

## Decisions

### Use Three Priority Bands

Research SHALL be grouped into three bands:

| Band | Meaning | Research Efforts |
| --- | --- | --- |
| P0 | Blocks or strongly shapes the base-platform decision | `decide-base-platform`, `research-daw-interchange-options`, `investigate-pi-port-topology`, `investigate-es9-config-profiles` |
| P1 | Should begin early because it improves validation and tooling, but does not block first hardware/runtime choice | `research-open-test-music-fixtures`, `investigate-agent-control-surface` |
| P2 | Important product-shape exploration that should not block first stored-sequence controller decisions | `research-live-trigger-sample-routing`, `research-synced-lfo-sampler-authoring` |

Rationale: the Pi/ES-9/DAW path determines whether the core product can run. Test fixtures and agent tooling accelerate the work once that path is clearer. Live trigger/sample routing can expand the product, but it should not dilute the controller-first platform decision.

Alternatives considered:
- Run all research equally: creates context switching and delays the base platform.
- Delay all non-hardware research: wastes time because several useful tracks can be completed by web/source review before bench hardware is available.

### Start With Web-Only Research

The first schedule pass SHALL favor research that can be completed with web searches, official documentation, open-source repositories, public issue trackers, package metadata, and license texts.

Web-first items:

| Effort | Web-only start? | Early web questions |
| --- | --- | --- |
| `research-daw-interchange-options` | Yes | Ardour exports, Ardour-on-Pi reports, DAWproject, MIDI file behavior, cross-DAW interchange, modules/software that import DAW/MIDI files |
| `investigate-es9-config-profiles` | Yes | ES-9 config tool behavior, SysEx/config formats, hosted/standalone routing docs, profile validation feasibility |
| `investigate-pi-port-topology` | Partially | Raspberry Pi 5 power/USB/display specs, ES-9 USB class compliance, controller connection options |
| `decide-base-platform` | Partially | package/license survey, Raspberry Pi OS variants, Python/Node/Tauri runtime constraints |
| `research-open-test-music-fixtures` | Yes | permissive Ardour/MIDI fixture sources, unfa candidate material, license evidence, attribution requirements |
| `investigate-agent-control-surface` | Yes | CLI/schema/MCP/skill best practices, dry-run and JSON-output patterns |
| `research-live-trigger-sample-routing` | Yes | live SFX/show-control/VJ/lighting software surface, protocols, cue systems, sample-trigger precedents |
| `research-synced-lfo-sampler-authoring` | Yes | single-cycle/AKWF waveform specs and licensing, Assimil8or preset format and clock-sync behavior, Assimil8or configurator licensing (A8Manager et al.) |

Rationale: this lets agents make progress immediately and narrows the hardware questions before bench time.

Alternatives considered:
- Wait for hardware before web work: reduces wrong assumptions, but slows the project unnecessarily.
- Research implementation libraries first: useful, but package choices depend on DAW/ES-9/Pi findings.

### Identify Blocking Research Channels

Each effort SHALL record which research channels are required before it can be considered complete.

Blocking channels:

| Channel | Needed For | Notes |
| --- | --- | --- |
| Official docs/web | All efforts | First pass for product specs, protocols, package licenses, examples, and support statements |
| Open-source/source review | DAW interchange, ES-9 profiles, agent tooling, fixtures | Needed when docs omit formats, CLI behavior, schemas, or license details |
| Hardware bench: Pi 5 + ES-9 | Base platform, port topology, ES-9 I/O, ES-9 profiles | Needed for actual USB/audio/MIDI discovery, latency, channel mapping, and power behavior |
| DAW host testing | DAW interchange, fixtures | Needed to export known Ardour sessions and compare MIDI/stem/session outputs |
| License review/outreach | Open test fixtures, synced-LFO sampler authoring | Needed before third-party music/session fixtures are committed or redistributed, and to resolve waveform (AKWF CC-BY/CC0) and configurator (all-rights-reserved) licensing |
| Live show domain research | Live trigger routing | Needed to compare SFX, VJ, lighting, show-control, OSC/MIDI/DMX/timecode practices |
| Deferred hardware access | Assimil8or-related routing, synced-LFO sampler authoring | Not available now; keep as later external-sampler research. Note the authoring-side waveform/preset/licensing research can complete web-only; only bench validation of sampler clock-sync needs the module |

Rationale: calling out channels prevents a research task from being marked complete when it only answered the web-search portion.

Alternatives considered:
- Treat each research task as done after notes are collected: too weak for hardware and licensing decisions.
- Require bench validation for everything: too slow and unnecessary for software landscape research.

### Initial Schedule

The initial schedule SHALL use four passes:

| Pass | Focus | Efforts | Blocking Channel |
| --- | --- | --- | --- |
| Pass 1: Web triage | Gather public evidence and narrow unknowns | `research-daw-interchange-options`, `investigate-es9-config-profiles`, `investigate-pi-port-topology`, `decide-base-platform` package/license survey | Web/docs/source review |
| Pass 2: Test and tooling support | Prepare validation inputs and agent workflow assumptions | `research-open-test-music-fixtures`, `investigate-agent-control-surface`, `research-synced-lfo-sampler-authoring` (licensing/format web triage) | Web/docs/source review, license review |
| Pass 3: Hardware and DAW bench | Validate the architecture-critical assumptions | `investigate-pi-port-topology`, `investigate-es9-config-profiles`, `research-daw-interchange-options`, `decide-base-platform` | Pi 5, ES-9, DAW host, MIDI controllers if available |
| Pass 4: Product expansion research | Explore future live cue/sample/show-control workflows | `research-live-trigger-sample-routing`, `research-synced-lfo-sampler-authoring` | Web/domain research first, later hardware/software integration if prioritized |

Pass 1 should produce enough evidence to decide what must be measured on hardware. Pass 3 should produce enough evidence to finalize or revise the base platform decision.

### Feed Base Platform Last

The base platform decision SHALL remain open until its blocker inputs are summarized.

Required inputs before the decision:
- Pi OS baseline candidates and constraints from web research.
- Runtime/framework package survey with permissive license compatibility.
- DAW interchange recommendation for the first import path.
- Pi/ES-9 port and power topology recommendation.
- ES-9 I/O/config validation recommendation.
- Known test fixture strategy, even if only synthetic fixtures are available initially.

Rationale: picking Python, Node, or Tauri without ES-9 I/O and DAW interchange evidence would be premature.

Alternatives considered:
- Decide platform immediately based on preference: faster, but likely to cause rework.
- Wait for every future research track: too conservative; live trigger routing and agent skill publishing do not need to block the first platform.

## Risks / Trade-offs

- Web research produces stale or incomplete evidence -> Mitigation: record source URLs, retrieval dates, and distinguish official docs from community reports.
- Hardware access arrives before web work is complete -> Mitigation: use Pass 1 notes to define a short bench checklist rather than blocking on perfect research.
- License-compatible fixtures are hard to find -> Mitigation: fall back to synthetic or project-authored fixtures while keeping artist outreach optional.
- Live trigger/sample routing expands scope -> Mitigation: keep it P2 until the controller-first Maybelle path is proven.
- The roadmap becomes stale -> Mitigation: require updates when a research track changes priority, gains a blocker, or produces a decision input.

## Migration Plan

No runtime migration is required. This change adds planning artifacts only. The roadmap should be revisited after Pass 1 web triage and again before the base platform decision is finalized.

Rollback is limited to removing this OpenSpec change before implementation.

## Open Questions

- Which hardware will be physically available for the first bench pass: Pi 5, ES-9, display, Pamela's Pro Workout, MIDI controller, or none?
- Should the first Pass 1 output be a single research report or one report per OpenSpec change?
- How formal should source citation be for web findings: markdown links only, structured evidence tables, or both?
- Do we want a dated `docs/research-roadmap.md` later, or should the roadmap remain entirely inside OpenSpec artifacts?
