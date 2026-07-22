## Why

Maybelle 314 targets a 5-inch touchscreen on the Raspberry Pi 5, but most UI work will happen on development machines without the panel attached. We need to confirm it is feasible to emulate the display and touch input during development so the UI can be built and iterated without constant hardware access, and we need to start shaping the UI/UX for a small, glanceable, finger-operated performance surface. Emulation feasibility and the UI framework choice are linked: a browser/kiosk UI is trivial to emulate at the panel's resolution, whereas full Pi 5 hardware emulation is not currently viable, so this evidence should also inform the base-platform kiosk/display decision.

## What Changes

- Add a research spike for emulating the Pi 5-inch touchscreen (display + touch) for development, and for an initial UI/UX direction.
- Confirm the target panel characteristics (5-inch, 800x480, capacitive, DSI/HDMI) and the display/touch stack on the Pi.
- Assess emulation options: web/kiosk UI at a fixed 800x480 with browser touch emulation, a fixed-size desktop window stand-in, and full-system Pi emulation via QEMU, and record which are feasible.
- Confirm that emulating the display and touch is feasible enough to develop the UI without the physical panel, and identify what must still be validated on real hardware (color/contrast, touch calibration, refresh, latency).
- Start UI/UX direction for an 800x480 landscape performance controller: touch-target sizing, contrast/legibility for stage/low-light, glanceable status, and the role of the touchscreen relative to CV-driven selection and the Bluetooth control channel.
- Do not implement the UI, a kiosk shell, an emulation harness, or a QEMU image in this change.

## Capabilities

### New Capabilities

- `touchscreen-emulation-and-ux-research`: Defines research requirements for emulating the Pi touchscreen during development and for the initial UI/UX direction.

### Modified Capabilities

- None.

## Impact

- Adds OpenSpec planning artifacts for touchscreen emulation and UI/UX research.
- Informs `decide-base-platform` (kiosk/display approach and the runtime UI framework choice), `investigate-agent-control-surface`, and `research-bluetooth-control-channel` (shared control/status surface).
- No production code, runtime dependencies, hardware integration, UI, or emulation tooling are introduced by this proposal.
