## 1. Decision Inputs

- [ ] 1.1 Identify the initial product shape and hard constraints the base platform must support.
- [ ] 1.2 Spike what event types Ardour-exported MIDI files can contain and which ones the runtime must preserve.
- [ ] 1.3 Spike how Pamela's clock/start/reset signals arrive through the ES-9 and how the runtime derives transport position from them.
- [ ] 1.4 Define the minimum rack-provided CV runtime parameters, including song banks, channel banks, selection, and transport controls.
- [ ] 1.5 Spike quantization, debounce, and latch behavior for runtime CV parameter inputs.
- [ ] 1.6 Evaluate Raspberry Pi OS image options and select the baseline flash image for ES-9, kiosk display, and appliance operation.
- [ ] 1.7 Create a repository decision record for the base-platform decision.
- [ ] 1.8 Define the evaluation criteria in the decision record before selecting a platform.

## 2. Candidate Evaluation

- [ ] 2.1 Shortlist at least two credible candidate base platforms.
- [ ] 2.2 Document each candidate's runtime, primary language or framework, persistence approach, deployment target, and local development model.
- [ ] 2.3 Compare every shortlisted candidate against the same evaluation criteria.
- [ ] 2.4 Record the material strengths, weaknesses, and assumptions for each candidate.
- [ ] 2.5 Review candidate MIT-compatible/permissive packages for MIDI parsing, ES-9 I/O, CV buffering, configuration, validation, local UI, and testing.
- [ ] 2.6 Spike JACK/PipeWire JACK, PortAudio/sounddevice, or ALSA direct access for ES-9 clock input, runtime CV input, and CV/gate output before finalizing the GUI framework.

## 3. Decision Record

- [ ] 3.1 Select one base platform and document the rationale for choosing it over rejected candidates.
- [ ] 3.2 Record assumptions and revisit triggers for the selected platform.
- [ ] 3.3 Summarize follow-up implementation impact, including repository structure, dependency management, local commands, test strategy, environment configuration, and deployment assumptions.

## 4. Verification

- [ ] 4.1 Review the decision record against every `base-platform-decision` requirement.
- [ ] 4.2 Run OpenSpec validation or status checks for the completed change.

## 5. Deferred Spikes

- [ ] 5.1 When an Assimil8or is available, inspect a known-good SD-card folder and preset file to determine whether the app should generate Assimil8or sample folders or presets.
