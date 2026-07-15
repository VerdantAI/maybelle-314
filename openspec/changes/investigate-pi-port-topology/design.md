## Context

Maybelle 314 will run on a Raspberry Pi 5 in or near a Eurorack system. The Pi must connect to an Expert Sleepers ES-9 for audio/CV I/O, drive a 5-inch display, accept rack-derived runtime CV/clock signals through the ES-9, and may accept input from external MIDI controllers such as BeatStep Pro or KeyStep Pro.

Initial research establishes several relevant facts:
- Raspberry Pi 5 provides 2 USB 3.0 host ports, 2 USB 2.0 host ports, USB-C power input with Power Delivery support, a standard 40-pin header, Gigabit Ethernet with optional PoE+ HAT, dual MIPI camera/display connectors, and PCIe 2.0 x1 for fast peripherals.
- Raspberry Pi recommends a high-quality 5V/5A USB-C power supply for Pi 5.
- ES-9 has a USB-C socket, is class-compliant USB 2.0, exposes 16-in/16-out USB audio, has 8 DC-coupled analogue outputs, 14 DC-coupled analogue inputs, and provides MIDI I/O through a breakout.
- KeyStep Pro-class controllers can provide MIDI over USB, DIN MIDI, CV/gate, clock, and reset-style signals. KeyStep Pro specifically lists 1 MIDI input, 2 MIDI outputs, USB, clock synchronization, CV/Gate/Mod outputs, and drum gate outputs.

The practical question is not just "which port works"; it is which topology is mechanically reliable, electrically safe, low-latency, serviceable on stage, and leaves enough ports for display, storage, keyboard/mouse/service access, and optional controllers.

## Goals / Non-Goals

**Goals:**
- Document the Raspberry Pi 5 physical port and power budget relevant to Maybelle.
- Compare power options: official USB-C supply, rack-powered 5V regulator/USB-C feed, PoE+ HAT, powered USB hub, and other appliance-safe options.
- Compare ES-9 connection options, especially Pi USB-A host to ES-9 USB-C data cable while leaving Pi USB-C for power.
- Compare external MIDI controller input paths: direct USB to Pi, DIN MIDI through a USB MIDI interface, DIN MIDI through the ES-9 breakout, and rack CV/gate through the ES-9.
- Define a recommended baseline topology plus alternatives and test criteria.
- Identify cable, hub, grounding, port naming, hotplug, and boot-order risks.

**Non-Goals:**
- Design a Eurorack power regulator circuit.
- Modify Raspberry Pi hardware.
- Implement MIDI routing, USB device discovery, or ES-9 drivers.
- Guarantee compatibility with every external controller.

## Decisions

### Prefer USB-C Power for the Initial Pi Baseline

The initial research baseline will power the Pi through its USB-C power input and use USB-A host ports for ES-9 and external USB devices.

Rationale: this follows the official Pi 5 power path and leaves the data topology simple. It also avoids coupling early software spikes to rack power supply design.

Alternatives considered:
- Rack-powered 5V feed into the Pi: attractive for an integrated module, but requires careful power, noise, grounding, current, connector, and safety validation.
- PoE+ HAT: clean single-cable power/network option, but consumes mechanical space and may not fit the rack enclosure concept.
- Power through GPIO 5V pins: possible in some Pi contexts, but riskier and outside the first-spike comfort zone without a dedicated electrical design.

### Treat ES-9 as the Primary USB Audio/CV Device

The first topology to test will connect a Pi USB-A host port to the ES-9 USB-C input using a compliant USB-A-to-USB-C data cable.

Rationale: the Pi 5 USB-C port is primarily needed for reliable power, and the Pi exposes multiple USB-A host ports. The ES-9 is class-compliant USB 2.0, so it should not require the Pi's USB-C connector for data.

Alternatives considered:
- USB-C hub/dock from the Pi power connector: likely wrong for Pi 5 because the connector is the power path, not the simplest host topology.
- Powered USB hub for ES-9 and controllers: may become necessary for multiple devices or power isolation, but should be tested after direct connection.
- ES-9 through another computer or bridge: adds unnecessary complexity for the Maybelle runtime.

### Keep External MIDI Controller Paths Pluggable

The research will not assume one controller path. It will evaluate direct USB MIDI to Pi, DIN MIDI through a USB interface, ES-9 MIDI breakout, and rack CV/gate/controller signals through ES-9 audio inputs.

Rationale: different controllers expose different physical ports, and each path has different latency, port naming, power, and routing implications. Maybelle should define a stable logical input model and let hardware topology map into it.

Alternatives considered:
- Route all controller input through ES-9: convenient if using ES-9 MIDI breakout or CV/gate inputs, but may consume scarce ES-9 inputs or depend on breakout hardware.
- Require direct USB MIDI to Pi: simple for USB controllers, but does not support DIN-only controllers without extra hardware.
- Require rack CV only: aligns with modular performance, but excludes useful external controllers for setup, selection, or override workflows.

## Risks / Trade-offs

- Pi undervoltage or brownout during performance -> Mitigation: verify the selected power source under ES-9, display, and controller load.
- USB bus instability or port renumbering -> Mitigation: test direct and hub topologies, record device IDs, and define stable device selection rules.
- Grounding/noise from rack power or USB cabling -> Mitigation: compare official supply, rack power, and hub-powered arrangements with ES-9 CV/audio measurements.
- Controller path ambiguity -> Mitigation: model controller inputs logically, then document physical mappings for USB, DIN, ES-9 MIDI, and CV/gate.
- External controller competes with ES-9 for USB power/bandwidth -> Mitigation: test powered hubs and reserve USB 3 ports for higher-throughput devices if needed.

## Migration Plan

No runtime migration is required because this change adds planning artifacts only. Research output should feed into `decide-base-platform`, ES-9 I/O spikes, enclosure design, and runtime device-discovery requirements.

Rollback is limited to removing this OpenSpec change before implementation.

## Open Questions

- Which Pi 5 USB port should be reserved for ES-9, and should it be physically labeled or keyed in the enclosure?
- Does ES-9 behave reliably from a Pi USB-A port using an A-to-C data cable at the required sample rate and channel count?
- How much current do the selected 5-inch display, controllers, and any USB hub draw?
- Is rack power desirable enough to justify a dedicated 5V regulator and power-noise validation?
- Should external MIDI controllers connect directly by USB, through a DIN USB-MIDI adapter, through the ES-9 MIDI breakout, or through CV/gate inputs?
- Should Maybelle support more than one simultaneous controller input path?
