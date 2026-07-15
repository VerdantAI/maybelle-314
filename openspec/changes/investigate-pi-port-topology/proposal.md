## Why

Maybelle 314's physical reliability depends on how the Raspberry Pi 5 is powered and how the ES-9, display, storage, and optional MIDI controllers connect. The Pi's USB-C connector is the recommended power input, while the ES-9 uses USB-C for class-compliant audio/MIDI, so the project needs a documented port and power topology before committing to the enclosure, cabling, MIDI input path, or ES-9 integration.

## What Changes

- Add a research spike for Raspberry Pi 5 power, USB, display, GPIO, MIDI, and rack-cabling topology.
- Compare powering the Pi from its USB-C power input, rack power via a dedicated supply/regulator, PoE+ HAT, or other appliance-safe options.
- Compare connecting the ES-9 from Pi USB-A host ports to ES-9 USB-C versus alternative hub or cable arrangements.
- Investigate how external MIDI controllers such as BeatStep Pro or KeyStep Pro can provide track/song selection input to the Pi.
- Decide whether external controller input should connect directly to the Pi over USB, through DIN MIDI via a USB MIDI interface, through the ES-9 MIDI breakout, or through rack CV/gate inputs.
- Do not implement hardware drivers, MIDI routing, power circuitry, or enclosure wiring in this change.

## Capabilities

### New Capabilities
- `pi-port-topology-research`: Defines how the project researches and records Raspberry Pi 5 power, port, USB, ES-9, display, and external MIDI controller topology.

### Modified Capabilities
- None.

## Impact

- Adds OpenSpec planning artifacts for hardware topology research.
- May inform `decide-base-platform`, ES-9 I/O spikes, enclosure design, display selection, MIDI controller support, and runtime device discovery.
- No production code, electrical design, dependencies, hardware wiring, or runtime behavior changes are introduced by this proposal.
