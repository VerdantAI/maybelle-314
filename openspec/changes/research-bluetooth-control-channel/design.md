## Context

Maybelle 314 is a Raspberry Pi 5 stored-sequence MIDI/CV controller. It follows Pamela's Pro Workout as the master clock, reads rack CV for runtime selection, and outputs CV/gates/triggers/modulation through the ES-9. The Pi has no attached keyboard in normal use, so an operator/performer needs a convenient wireless way to send control events, manage banks, and deliver updates or new song bundles. The Pi 5 has an onboard Bluetooth/BLE radio managed by BlueZ, making Bluetooth the obvious candidate for that side channel.

The research must separate three questions the requester raised — can Bluetooth trigger events, can it update the software, and what is a simple PIN-style authentication — and it must protect the runtime's timing and safety guarantees, since a wireless channel is a new remote attack and failure surface.

Initial research suggests:
- The Pi acts as a BLE peripheral / GATT server and a phone/laptop as the BLE central. A central writing to a GATT characteristic is a clean way to "trigger an event" on the Pi; characteristics can also notify the central of state changes. BlueZ exposes this over D-Bus, and Python examples (BlueZ example-gatt-server/example-advertisement, libraries such as BLE-GATT) are established starting points. Advertising and the GATT server are separate concerns that must both run.
- BLE is well suited to small, occasional control messages, not to bulk transfer or sample-accurate timing. Connection intervals, MTU negotiation (20–247 bytes), and phone-OS throttling/disconnects make it a non-realtime channel. Clock-tight performance triggers should therefore stay on CV/MIDI; Bluetooth events belong to control/config/cue level (bank select, transport, load-song, non-urgent cues).
- Software update over BLE is feasible but slow: with BLE 5 Data Length Extension and a ~247-byte MTU, a ~512 KB image takes roughly 2–4 minutes; without DLE, 15–20+ minutes, over a link the phone may throttle or drop. Robust designs use chunked transfer with sequence numbers/ACKs and, critically, atomic apply with rollback to avoid bricking. Because of these limits, a better pattern is to use Bluetooth to authorize/initiate an update while the actual image is fetched over Wi-Fi/USB; small song bundles or config may be acceptable to send directly over BLE.
- BLE pairing uses an agent capability model (NoInputNoOutput, DisplayOnly, DisplayYesNo, KeyboardDisplay, KeyboardOnly). With a display, the Pi can show a 6-digit passkey (0–999999) that the operator confirms/enters — a natural "PIN." Bonding then persists the trusted device. However, BLE passkey pairing has documented cryptographic weaknesses (passkey bypass by a fraudulent responder), and NoInputNoOutput "Just Works" gives no authentication at all. A layered scheme (BLE bonding for link security plus an application-layer PIN/shared-secret gate before commands are honored) is stronger than relying on pairing alone.
- BLE (GATT) is the phone-friendly path, especially on iOS where classic SPP is not generally available. Classic Bluetooth BR/EDR + RFCOMM/SPP remains an option for laptop tooling but is a poorer fit for mobile apps.

## Goals / Non-Goals

**Goals:**
- Define a repeatable research process for a Bluetooth control channel on the Pi.
- Determine which event types Bluetooth can reliably trigger and classify them against clock-tight CV/MIDI triggers.
- Determine whether and how Bluetooth should participate in software and content updates, including throughput limits and safe apply/rollback.
- Recommend a simple, operator-friendly authentication approach (PIN/passkey plus an application-layer gate) and record its weaknesses.
- Characterize the added security/safety surface and how to contain it.
- Produce decision evidence for the base-platform stack, agent control surface, and live trigger routing.

**Non-Goals:**
- Implement a BLE GATT server, advertising, pairing flow, or bonding storage.
- Implement or ship a software-update or OTA mechanism.
- Choose the final authentication design or credential lifecycle.
- Make Bluetooth a real-time performance trigger path.
- Support arbitrary remote administration of the Pi.

## Decisions

### Treat Bluetooth as an Out-of-Band Control Channel, Not a Realtime Path

The research will scope Bluetooth to control/config/cue-level events and status, explicitly excluding sample-accurate performance triggers, which remain on CV/MIDI and the master clock.

Rationale: BLE timing (connection intervals, MTU, phone-OS throttling, drops) cannot meet clock-tight guarantees, but is fine for bank selection, transport, loading a song, and non-urgent cues.

Alternatives considered:
- Use Bluetooth for performance triggers: attractive for wireless playing, but risks timing jitter and dropouts against a hardware-clocked rack.
- Reject Bluetooth for events entirely: loses a convenient operator surface for non-realtime control.

### Prefer BLE GATT Peripheral Role

The Pi will be researched as a BLE peripheral / GATT server that a phone or laptop central connects to, with classic BR/EDR SPP considered only as a secondary laptop-tooling option.

Rationale: GATT peripheral is the standard, phone-friendly model (notably required on iOS), maps cleanly to "central writes a characteristic to trigger an event," and is well supported by BlueZ.

Alternatives considered:
- Classic Bluetooth SPP: simple serial semantics, but weak mobile support, especially iOS.
- Pi as central: useful for talking to sensors/wearables, but the operator's phone is the natural central here.

### Split Update Authorization from Bulk Transfer

The research will evaluate using Bluetooth to authenticate and initiate updates while large software images are delivered over Wi-Fi or USB, reserving direct BLE transfer for small song bundles or configuration where the time cost is acceptable.

Rationale: BLE throughput makes full-image transfer slow and drop-prone; separating "authorize/trigger" (small, latency-tolerant) from "transfer" (bulk) plays to each channel's strengths and reduces brick risk.

Alternatives considered:
- Full software image over BLE: possible but slow (minutes) and fragile without careful chunking/ACK/rollback.
- No Bluetooth role in updates: simpler, but misses a convenient way to authorize and kick off an update from a phone.

### Layer Authentication: Bonding Plus an Application PIN

The research will recommend a layered scheme — BLE pairing/bonding (passkey shown on the Pi display) for link-level trust, plus an application-layer PIN/shared-secret that must be presented before control or update commands are honored — and will record the known passkey-pairing weaknesses.

Rationale: a display-shown 6-digit passkey is the intuitive "PIN," and bonding persists trusted devices, but passkey pairing has known cryptographic weaknesses and "Just Works" provides no authentication; an application-layer gate keeps command authorization under Maybelle's control.

Alternatives considered:
- Bonding/passkey only: convenient, but inherits BLE pairing weaknesses and offers no per-command authorization.
- Application PIN only over an unauthenticated link: simpler pairing, but exposes the PIN and traffic without link security.
- Full certificate/mutual-TLS-style auth: strong, but heavier than "simple PIN" and likely overkill for the operator use case.

### Contain the Security and Safety Surface

The research will define how the Bluetooth channel is contained: no arbitrary shell/admin, a bounded command set, safe defaults when the channel is idle or a device disconnects, and no ability to disrupt runtime timing.

Rationale: a wireless channel is a new remote attack and failure surface; runtime safety and timing must not depend on it, and it must fail safe.

Alternatives considered:
- Expose broad remote administration: powerful, but a large attack surface inconsistent with a performance appliance.
- Trust any paired device fully: convenient, but unsafe if a phone is lost or a device is spoofed.

## Risks / Trade-offs

- BLE timing jitter/drops make it unsuitable for performance triggers -> Mitigation: scope Bluetooth to non-realtime control/config/cue events and keep clock-tight triggers on CV/MIDI.
- BLE update transfer is slow and can brick the device -> Mitigation: prefer BLE-authorized, Wi-Fi/USB-delivered updates; require atomic apply with rollback for anything applied.
- Passkey pairing weaknesses or "Just Works" undermine authentication -> Mitigation: layer bonding with an application-level PIN/shared-secret and document residual risk.
- A wireless channel expands the attack/failure surface -> Mitigation: bound the command set, no remote shell, fail-safe on disconnect, keep runtime independent of the channel.
- BlueZ/D-Bus integration adds runtime and packaging complexity -> Mitigation: evaluate the dependency and service model as part of the base-platform stack decision.
- Phone-platform differences (iOS vs Android) complicate the client -> Mitigation: standardize on BLE GATT and record per-platform constraints before any app work.

## Migration Plan

No runtime migration is required because this change adds planning artifacts only. Research output should feed `decide-base-platform` (stack/dependency and service model), `investigate-agent-control-surface` (wireless control surface), and `research-live-trigger-sample-routing` (trigger source classification).

Rollback is limited to removing this OpenSpec change before implementation.

## Open Questions

- Which control events are safe and useful over Bluetooth (bank select, song load, transport, cues) versus reserved for CV/MIDI?
- What is the realistic BLE throughput on the Pi 5 for song-bundle-sized transfers, and what bundle size is acceptable to send directly versus fetch over Wi-Fi/USB?
- What update-apply model guarantees atomicity and rollback regardless of transport?
- Is the Pi display always available to show a pairing passkey, or is a fallback PIN provisioning method needed (e.g. preset PIN, config file, on-screen entry)?
- How are bonded devices and the application PIN stored, rotated, and revoked if a phone is lost?
- Does the runtime need to keep functioning fully with Bluetooth disabled, and should Bluetooth be off by default until enabled by the operator?
- Do we standardize on a BLE-only client, or also provide a classic-SPP path for laptop tooling?
