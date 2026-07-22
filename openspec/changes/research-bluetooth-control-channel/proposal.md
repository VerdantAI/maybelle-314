## Why

Maybelle 314 runs headless-ish on a Raspberry Pi 5 with a small display and no keyboard, and its primary control surface is rack CV plus the master clock. A wireless side channel from a phone or laptop would let a performer or operator trigger control events, manage song/channel banks, and push updates or new song bundles without wiring a keyboard to the Pi. The Pi 5 has built-in Bluetooth/BLE, so before committing to it we need evidence about what Bluetooth can and cannot do here: whether it can trigger runtime events reliably, whether it is a viable path for software/content updates, and what a simple operator-friendly authentication (PIN/passkey style) looks like without weakening runtime safety.

## What Changes

- Add a research spike for a Bluetooth control channel on the Raspberry Pi 5, using the built-in radio and BlueZ.
- Investigate event triggering: how a phone/laptop (BLE central) can write to the Pi (BLE peripheral / GATT server) to select song/channel banks, drive transport, or fire non-realtime cues, and where the boundary lies against clock-tight CV/MIDI performance triggers.
- Investigate software and content updates over Bluetooth: BLE throughput limits, feasibility of transferring song bundles vs full software images, and safer designs where Bluetooth authorizes or initiates an update fetched over Wi-Fi/USB.
- Investigate simple authentication: BLE pairing/bonding with a passkey (PIN), an application-layer PIN/shared-secret gate on top of GATT, known passkey weaknesses, and a layered recommendation.
- Assess the security and safety surface a wireless channel adds, and how to keep it from compromising runtime timing or safe operation.
- Compare BLE (GATT) against classic Bluetooth (BR/EDR, SPP) for this use, and note phone-platform constraints (iOS/Android).
- Do not implement a BLE server, pairing flow, update mechanism, or authentication in this change.

## Capabilities

### New Capabilities

- `bluetooth-control-channel-research`: Defines research requirements for a Bluetooth-based control, update-initiation, and authentication channel on the Pi runtime.

### Modified Capabilities

- None.

## Impact

- Adds OpenSpec planning artifacts for a Bluetooth control-channel investigation.
- Informs `decide-base-platform` (BlueZ/stack dependency, radio role, service management), `investigate-agent-control-surface` (a wireless control surface alongside CLI/CV), `research-live-trigger-sample-routing` (an additional trigger source and its latency/reliability class), and runtime safety/security assumptions.
- No production code, runtime dependencies, hardware integration, or wireless services are introduced by this proposal.
