## 1. Decision Inputs

- [x] 1.1 Identify the initial product shape and hard constraints the base platform must support. (decision-record §2)
- [x] 1.2 Spike what event types Ardour-exported MIDI files can contain and which ones the runtime must preserve. (`research-daw-interchange-options`: notes+velocity common denominator; DAW-host export bench still outstanding.)
- [x] 1.3 Spike how Pamela's clock/start/reset signals arrive through the ES-9 and how the runtime derives transport position from them. (Design: read as DC input channels, edge-detect in the audio callback — hardware confirmation pending §8 bench.)
- [x] 1.4 Define the minimum rack-provided CV runtime parameters, including song banks, channel banks, selection, and transport controls. (manifest `selection` model)
- [x] 1.5 Spike quantization, debounce, and latch behavior for runtime CV parameter inputs. (manifest `selection`: choices/debounce/hysteresis/latch)
- [x] 1.6 Evaluate Raspberry Pi OS image options and select the baseline flash image for ES-9, kiosk display, and appliance operation. (Desktop first → Lite+kiosk appliance)
- [x] 1.7 Create a repository decision record for the base-platform decision. (`decision-record.md`)
- [x] 1.8 Define the evaluation criteria in the decision record before selecting a platform. (decision-record §3)

## 2. Candidate Evaluation

- [x] 2.1 Shortlist at least two credible candidate base platforms. (A: Python+web/kiosk, B: Node+web, C: Rust+Tauri)
- [x] 2.2 Document each candidate's runtime, primary language or framework, persistence approach, deployment target, and local development model.
- [x] 2.3 Compare every shortlisted candidate against the same evaluation criteria. (decision-record §4)
- [x] 2.4 Record the material strengths, weaknesses, and assumptions for each candidate.
- [x] 2.5 Review candidate MIT-compatible/permissive packages for MIDI parsing, ES-9 I/O, CV buffering, configuration, validation, local UI, and testing. (decision-record §7)
- [ ] 2.6 Spike JACK/PipeWire JACK, PortAudio/sounddevice, or ALSA direct access for ES-9 clock input, runtime CV input, and CV/gate output before finalizing the GUI framework. (blocked: needs ES-9 + Pi bench — the gating spike, §8; framework recommended, I/O layer provisional pending this.)

## 3. Decision Record

- [x] 3.1 Select one base platform and document the rationale for choosing it over rejected candidates. (decision-record §5)
- [x] 3.2 Record assumptions and revisit triggers for the selected platform. (decision-record §9)
- [x] 3.3 Summarize follow-up implementation impact, including repository structure, dependency management, local commands, test strategy, environment configuration, and deployment assumptions. (decision-record §10)

## 4. Verification

- [x] 4.1 Review the decision record against every `base-platform-decision` requirement.
- [x] 4.2 Run OpenSpec validation or status checks for the completed change. (`openspec validate` passes, 2026-07-22)

## 5. Deferred Spikes

- [ ] 5.1 When an Assimil8or is available, inspect a known-good SD-card folder and preset file to determine whether the app should generate Assimil8or sample folders or presets. (deferred hardware)
