## 1. Source Research

- [x] 1.1 Document Raspberry Pi 5 port inventory, power input requirements, USB host ports, GPIO header, display connectors, Ethernet/PoE+ option, and PCIe option from primary sources.
- [x] 1.2 Document ES-9 USB, MIDI breakout, channel count, class-compliance, power, and cabling requirements from primary sources.
- [x] 1.3 Document external controller port options for BeatStep Pro, KeyStep Pro, and any other likely controllers.

## 2. Power Topology

- [x] 2.1 Compare official USB-C power, rack-powered 5V conversion, PoE+ HAT, powered USB hub assumptions, and any other credible power options.
- [x] 2.2 Record current capacity, grounding/noise risks, mechanical fit, cable strain, serviceability, boot reliability, and safety concerns for each power option.
- [x] 2.3 Recommend the first power topology for hardware spikes and list fallback options.

## 3. ES-9 USB Topology

- [ ] 3.1 Test Pi USB-A host port to ES-9 USB-C with a known data-capable cable. (blocked: needs Pi 5 bench)
- [ ] 3.2 Record ES-9 enumeration, ALSA/JACK/PipeWire visibility, MIDI visibility, sample-rate support, channel count, latency observations, and hotplug behavior. (blocked: needs Pi 5 bench)
- [ ] 3.3 Test whether a powered USB hub changes stability, device naming, latency, or power behavior. (blocked: needs Pi 5 bench)
- [x] 3.4 Decide whether the ES-9 should have a reserved physical Pi USB port and enclosure label.

## 4. External MIDI Controller Paths

- [ ] 4.1 Test direct USB MIDI controller input to the Pi. (blocked: needs Pi 5 bench)
- [ ] 4.2 Test DIN MIDI input through a USB MIDI interface connected to the Pi. (blocked: needs Pi 5 bench)
- [ ] 4.3 Test DIN MIDI input through the ES-9 MIDI breakout if breakout hardware is available. (blocked: needs Pi 5 bench)
- [ ] 4.4 Test controller CV/gate outputs through ES-9 inputs for track/song/channel selection if the controller supports useful CV/gate output. (blocked: needs Pi 5 bench)
- [x] 4.5 Compare each controller path for latency, power, device naming stability, configuration complexity, and fit for runtime selection workflows.

## 5. Topology Recommendation

- [x] 5.1 Produce a recommended baseline wiring diagram for Pi power, ES-9 data, display, service access, and optional controllers.
- [x] 5.2 List required cables, adapters, hubs, rack power components, and labels for the recommended topology.
- [x] 5.3 Document runtime device-discovery assumptions and fallback behavior when optional controllers are absent or renamed.
- [ ] 5.4 Feed relevant findings back into `decide-base-platform`, ES-9 I/O spikes, and enclosure planning.

## 6. Verification

- [ ] 6.1 Review the research output against every `pi-port-topology-research` requirement.
- [x] 6.2 Run OpenSpec validation or status checks for the completed change. (`openspec validate` passes, 2026-07-22)
