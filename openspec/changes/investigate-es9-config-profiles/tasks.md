## 1. Official Tool Research

- [ ] 1.1 Record official ES-9 firmware/tool versions, download URLs, and the relationship between firmware and config tool versions.
- [ ] 1.2 Inspect the official HTML tool for configurable areas, SysEx commands, save/load behavior, browser requirements, and Web MIDI SysEx constraints.
- [ ] 1.3 Determine whether the official tool has a license compatible with reuse, wrapping, or derivative tooling.

## 2. Config Dump Analysis

- [ ] 2.1 Obtain at least one ES-9 `.syx` config dump from the official tool.
- [ ] 2.2 Decode the config dump header, version, payload length, terminator, and field ordering.
- [ ] 2.3 Map config fields for input DC blocking, routing, hosted/standalone slots, MIDI channels, DC offsets, mixer state, stereo links, EQ, and smoothing.
- [ ] 2.4 Round-trip a config through the official tool and confirm whether the decoded semantic content is stable.

## 3. Maybelle Patch Profile Model

- [ ] 3.1 Define semantic rack roles for Pamela clock, reset, run/start, song bank CV, song selection CV, channel bank CV, pitch CV outputs, gates, triggers, and modulation outputs.
- [ ] 3.2 Map each semantic role to required ES-9 physical channel, USB channel, direction, voltage range, calibration assumptions, and safety behavior.
- [ ] 3.3 Define how song bundles and channel banks reference the ES-9 patch profile without duplicating low-level routing details.

## 4. Validation Strategy

- [ ] 4.1 Define validation rules comparing a Maybelle patch profile against a downloaded ES-9 config dump.
- [ ] 4.2 Define warning/error severity for routing mismatches, DC blocking conflicts, DC offset conflicts, stereo link conflicts, mixer routing surprises, hosted/standalone slot mismatches, and MIDI channel conflicts.
- [ ] 4.3 Define a human-readable patch checklist that covers physical cable assumptions the ES-9 config cannot verify.
- [ ] 4.4 Decide whether the runtime should refuse to arm playback, warn only, or require confirmation when validation fails.

## 5. Profile Utility Recommendation

- [ ] 5.1 Compare implementation paths: validator-only, `.syx` generator, official-tool wrapper, independent profile maker, and manual checklist.
- [ ] 5.2 Recommend the first implementation path and list unresolved hardware/license spikes before any utility writes config to an ES-9.
- [ ] 5.3 Feed relevant findings back into `decide-base-platform`, ES-9 I/O spikes, and the future song-bundle-format proposal.

## 6. Verification

- [ ] 6.1 Review the research output against every `es9-profile-validation-research` requirement.
- [ ] 6.2 Run OpenSpec validation or status checks for the completed change.
