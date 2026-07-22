## 1. Bluetooth Stack Research

- [ ] 1.1 Research the Raspberry Pi 5 Bluetooth/BLE radio and BlueZ capabilities, including GATT server/peripheral support and D-Bus integration.
- [ ] 1.2 Research Python/BlueZ approaches for a GATT server and advertising (BlueZ examples, BLE-GATT and similar libraries), and note the service/packaging implications.
- [ ] 1.3 Compare BLE (GATT) against classic Bluetooth BR/EDR + SPP for this use, including iOS/Android client constraints.

## 2. Event Triggering

- [ ] 2.1 Define how a central (phone/laptop) triggers events by writing GATT characteristics, and how the Pi notifies state back.
- [ ] 2.2 Enumerate candidate control events (song/channel bank select, transport, load-song, non-urgent cues) and classify each as Bluetooth-appropriate or reserved for clock-tight CV/MIDI.
- [ ] 2.3 Characterize BLE latency/reliability (connection interval, MTU, throttling, disconnects) against Maybelle's timing needs.

## 3. Software and Content Updates

- [ ] 3.1 Research BLE throughput on the Pi and estimate transfer times for song-bundle-sized vs full-image payloads.
- [ ] 3.2 Compare direct-over-BLE transfer with a BLE-authorized, Wi-Fi/USB-delivered update model.
- [ ] 3.3 Identify safe-apply requirements: chunking/ACK, integrity checks, atomic apply, and rollback to avoid bricking.
- [ ] 3.4 Recommend which update payloads (config, song bundles, software images) each transport should carry.

## 4. Authentication and Security

- [ ] 4.1 Research BLE pairing/bonding agent modes and passkey (PIN) entry/display on the Pi, including the display-passkey flow.
- [ ] 4.2 Record known passkey-pairing weaknesses and the "Just Works" (no-auth) case.
- [ ] 4.3 Define a layered recommendation: BLE bonding plus an application-layer PIN/shared-secret gate before commands are honored.
- [ ] 4.4 Define credential handling: passkey provisioning, bonded-device storage, PIN rotation/revocation, and lost-device response.
- [ ] 4.5 Define containment: bounded command set, no remote shell/admin, fail-safe on disconnect, and runtime independence from the channel.

## 5. Decision Handoff

- [ ] 5.1 Recommend the Bluetooth role (control channel scope, update participation, auth scheme).
- [ ] 5.2 List follow-up spikes before implementing a GATT server, pairing flow, or update mechanism.
- [ ] 5.3 Feed findings into `decide-base-platform`, `investigate-agent-control-surface`, and `research-live-trigger-sample-routing`.

## 6. Verification

- [ ] 6.1 Review the research output against every `bluetooth-control-channel-research` requirement.
- [ ] 6.2 Run OpenSpec validation or status checks for the completed change.
