## 1. Roadmap Inventory

- [ ] 1.1 Confirm the active OpenSpec research efforts and their current artifact status.
- [ ] 1.2 Assign each research effort to P0, P1, or P2 based on whether it blocks base platform selection, supports validation/tooling, or explores later product expansion.
- [ ] 1.3 Record dependency links between research efforts and the `decide-base-platform` change.

## 2. Web-First Research Planning

- [ ] 2.1 Mark which research efforts can begin with web-only work.
- [ ] 2.2 For each web-first effort, list the early questions answerable from official docs, public repositories, package metadata, issue trackers, source code, or license texts.
- [ ] 2.3 Identify which web findings must be verified later by hardware, DAW host testing, license review, outreach, or domain expert review.

## 3. Blocking Channel Review

- [ ] 3.1 Record required blocker channels for each research effort.
- [ ] 3.2 Separate web-only progress from completion criteria that require non-web evidence.
- [ ] 3.3 Identify unavailable or deferred blockers, including Assimil8or access and any missing bench hardware.

## 4. Initial Schedule

- [ ] 4.1 Define the first web triage pass for DAW interchange, ES-9 configuration/profile research, Pi port topology, and base platform package/runtime research.
- [ ] 4.2 Define the second pass for test fixture sourcing and agent control surface research.
- [ ] 4.3 Define the hardware and DAW bench pass needed before the base platform decision is finalized.
- [ ] 4.4 Define the later product-expansion pass for live trigger/sample/show-control routing.

## 5. Base Platform Inputs

- [ ] 5.1 List the research outputs required before finalizing OS, runtime/framework, ES-9 I/O stack, package set, bundle format, and kiosk/display approach.
- [ ] 5.2 Mark any base-platform conclusions that would remain provisional without hardware or DAW host evidence.
- [ ] 5.3 Feed the roadmap outputs into `decide-base-platform` once the first research pass is complete.

## 6. Verification

- [ ] 6.1 Review the roadmap against every `research-roadmap-prioritization` requirement.
- [ ] 6.2 Run OpenSpec validation or status checks for the completed change.
