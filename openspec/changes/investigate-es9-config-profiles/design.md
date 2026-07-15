## Context

Maybelle 314 will use the ES-9 as the bridge between the Raspberry Pi and the rack. The Pi must read Pamela's clock/reset/run and rack-provided CV runtime parameters, then output pitch CV, gates, triggers, and modulation through the correct ES-9 channels into the correct patches.

Expert Sleepers provides an ES-9 configuration tool as an HTML/Web MIDI SysEx app. The firmware page links versioned web tools such as `webapps/es9_config_tool_1.3.html`, and firmware 1.3.0 added the ability to save and load configurations from the tool. Initial inspection of the 1.3 tool shows:
- Web MIDI SysEx is used to request version, sample rate, usage, config dump, upload config, save to hosted/standalone flash slots, restore slots, and reset defaults.
- Saved config dumps are `.syx` files with an ES-9 SysEx header and fixed-length payload validation in the tool.
- The tool models hosted and standalone configurations, input DC blocking, routing from inputs/buses/USB/mix to USB and physical outputs, stereo links, MIDI channels, output DC offsets, filters/EQ, mix smoothing, and mixer state.
- The ES-9 itself provides 16-in/16-out USB audio, DC-coupled 3.5mm outputs, DC-coupled inputs, and an internal mixer, so a bad profile can silently route Maybelle's control signals to the wrong rack destination.

## Goals / Non-Goals

**Goals:**
- Determine whether Maybelle can parse and validate ES-9 `.syx` configuration dumps.
- Determine whether Maybelle can generate a known-good ES-9 profile or assist a user in producing one through the official tool.
- Define a Maybelle patch profile model that maps rack functions to ES-9 physical inputs, physical outputs, USB channels, and expected signal roles.
- Identify validation rules that prevent common dangerous or confusing mistakes, such as clock input routed to the wrong channel, pitch CV sent to the wrong output, DC blocking enabled on CV inputs, or output DC offsets conflicting with calibration.
- Decide whether a future implementation should wrap the official HTML tool, produce `.syx` files for upload, validate downloaded `.syx` files, or build an independent profile utility.

**Non-Goals:**
- Implement the parser, generator, validator, or UI.
- Send SysEx to real ES-9 hardware.
- Modify Expert Sleepers' official tool.
- Guarantee electrical patch correctness without user confirmation and calibration.

## Decisions

### Treat Patch Intent as the Source of Truth

The investigation will define a Maybelle patch profile separate from the ES-9 raw config dump. The profile describes semantic roles such as `pamela_clock_in`, `reset_in`, `song_select_cv_in`, `pitch_cv_out_1`, and `gate_out_1`.

Rationale: the ES-9 config encodes routing and mixer state, but it does not know why a channel matters to Maybelle. A semantic profile lets the app validate intent against hardware configuration and present meaningful errors.

Alternatives considered:
- Use raw ES-9 `.syx` as the only profile: precise, but opaque to users and difficult to connect to song/channel-bank mappings.
- Keep mappings only in song manifests: easy to start, but misses ES-9 global routing and hosted/standalone configuration risk.

### Validate Before Generating or Uploading

The first deliverable should be a validator for downloaded ES-9 configs and Maybelle patch profiles. Config generation or upload should be treated as a later capability after parsing and validation are proven.

Rationale: validation has lower hardware risk and can immediately prevent misconfiguration. Generation and SysEx upload require stronger confidence in the file format and version compatibility.

Alternatives considered:
- Generate config first: useful if the format is straightforward, but risky before round-trip tests.
- Only provide manual setup instructions: safe, but easy for users to misconfigure and hard to verify during performance setup.

### Investigate Official Tool Reuse Without Assuming License or API Stability

The investigation will inspect the official HTML tool's behavior and data model, but it will not assume the code can be copied, redistributed, or treated as a stable API.

Rationale: the tool is useful evidence for file structure and SysEx behavior, but Maybelle needs a maintainable implementation path with clear licensing and version compatibility.

Alternatives considered:
- Fork the HTML tool immediately: fastest path to a UI, but license and maintenance implications are unknown.
- Ignore the tool internals: leaves us guessing about valid config dumps and SysEx commands.

## Risks / Trade-offs

- The ES-9 config format changes between firmware versions -> Mitigation: record the firmware/tool version and require version-specific parsers.
- The official tool code is not licensed for reuse -> Mitigation: use it as behavioral reference only unless reuse rights are confirmed.
- Generated profiles could misroute control voltage -> Mitigation: start with validation, require human-readable patch summaries, and require round-trip tests with downloaded configs.
- ES-9 routing is not enough to prove physical patching -> Mitigation: pair config validation with a rack patch checklist and runtime signal self-tests.
- Hosted and standalone slots differ unexpectedly -> Mitigation: validate both slots or require explicit target slot selection.

## Migration Plan

No runtime migration is required because this change adds planning artifacts only. Research output should feed into `decide-base-platform`, ES-9 I/O spikes, song-bundle output mapping, and a future ES-9 profile utility proposal.

Rollback is limited to removing this OpenSpec change before implementation.

## Open Questions

- What exact fields are present in ES-9 firmware 1.3 `.syx` config dumps, and which are needed by Maybelle?
- Can a config dump be round-tripped through the official tool without semantic changes?
- Is the official HTML tool license compatible with reuse, wrapping, or derivative tooling?
- Should Maybelle support validation only, profile generation only, or both?
- How should patch profiles represent physical cable checks that ES-9 configuration cannot observe?
- Should the runtime refuse to arm playback if the detected ES-9 config does not match the expected patch profile?
