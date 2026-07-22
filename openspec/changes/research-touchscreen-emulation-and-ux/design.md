## Context

Maybelle 314 is a Raspberry Pi 5 stored-sequence MIDI/CV controller with a 5-inch touchscreen for its local UI. The README's current bias is a Python-centered runtime with a browser/kiosk UI, but the UI framework and kiosk/display approach are still open in `decide-base-platform`. Most UI development will happen on laptops/desktops, so the team needs a dependable way to emulate the display and touch input, and an initial UI/UX direction sized for the panel and the performance context (often dark venues, quick glances, finger operation).

Emulation feasibility and the UI framework are coupled: if the UI is a web/kiosk app, emulating it is as simple as running a browser at the panel's resolution with touch emulation; if the UI is a native GPU/DSI-bound toolkit, emulation is much harder because the Pi 5 hardware itself does not emulate well.

Initial research suggests:
- The 5-inch panels sold for the Pi are typically 800x480 capacitive (5-point) displays over DSI (driver-free, up to 60Hz), with HDMI + USB-touch variants also available. So the design target is a fixed 800x480 landscape capacitive touchscreen.
- Full-system Pi 5 emulation is not currently viable: QEMU provides machine types only up to the Pi 4 (no `raspi5`), and emulators generally lack GPU acceleration and do not model DSI/I2C/SPI/USB touch peripherals. Attempting to emulate the whole device (BCM2712 + GPU + DSI touch) is therefore not a practical development path.
- Emulating the display and touch is very feasible when the UI is a web/kiosk app. Chromium can run fullscreen in `--kiosk` mode, and Chrome DevTools device mode allows a custom 800x480 device with "Force touch," which replaces mouse events with real touch events (touchstart/move/end). The Chrome DevTools Protocol Emulation domain can enable touch programmatically for automated testing.
- A fixed-size desktop window (native or web) sized to 800x480 is a simple stand-in for layout work, with mouse-as-touch acceptable for early iteration and a touch monitor useful for realism.
- Some things only real hardware validates: color/contrast under venue lighting, capacitive touch calibration and accuracy, panel refresh/latency, DSI orientation, and finger ergonomics at physical size.

## Goals / Non-Goals

**Goals:**
- Confirm the target panel characteristics and the Pi display/touch stack.
- Determine which emulation approaches are feasible for developing the UI without the physical panel, and recommend one.
- Identify what must still be validated on real hardware.
- Produce an initial UI/UX direction for an 800x480 landscape performance controller.
- Feed evidence into the base-platform kiosk/display and UI-framework decisions.

**Non-Goals:**
- Build the UI, a kiosk shell, or an emulation harness.
- Produce a full QEMU or full-system Pi image.
- Finalize the UI framework or the visual design system.
- Decide the complete on-device interaction model or every screen.

## Decisions

### Emulate the Display Target, Not the Pi Hardware

The research will treat "emulation for development" as emulating the 800x480 display and touch input on a dev machine, not emulating the Pi 5 system.

Rationale: QEMU has no Pi 5 machine type and does not model the GPU or DSI/USB touch, so full-system emulation is impractical; the practical need is to iterate the UI at correct dimensions with touch behavior.

Alternatives considered:
- Full-system QEMU Pi 5 emulation: would exercise the real OS image, but is not available and would still miss GPU/touch.
- Develop only on hardware: accurate, but slow to iterate and gated on panel availability.

### Prefer a Web/Kiosk UI for Emulability

The research will weigh the UI framework partly on how easily it emulates, favoring a browser/kiosk UI that runs at 800x480 with touch emulation on any dev machine.

Rationale: a web/kiosk UI makes emulation trivial (Chromium kiosk + DevTools device mode + Force touch) and matches the README's current bias, while a native GPU/DSI-bound toolkit is hard to emulate given the hardware limits.

Alternatives considered:
- Native Pi GUI toolkit: potentially lighter at runtime, but harder to emulate and iterate off-device.
- Hybrid (web UI in a native shell): flexible, but adds a shell dependency to evaluate against the base-platform decision.

### Keep a Hardware Validation Checklist

The research will define a short list of properties that only the physical panel can confirm, so emulated development does not create false confidence.

Rationale: emulation cannot reproduce venue-light contrast, capacitive touch accuracy, refresh/latency, orientation, or finger ergonomics; these must be checked on the real panel.

Alternatives considered:
- Trust emulation fully: fast, but risks shipping a UI that is illegible or mis-sized in the real context.
- Require hardware for every change: safe, but defeats the purpose of emulation.

### Frame the UI/UX for a Small Performance Surface

The research will produce an initial UI/UX direction constrained to 800x480 landscape, large touch targets, high contrast for low light, glanceable status, and a clear division of roles between the touchscreen, CV-driven selection, and the Bluetooth control channel.

Rationale: the device is operated live, often in the dark, by finger; legibility, target size, and immediate feedback dominate, and the touchscreen is primarily status/config/manual-override while CV drives real-time selection.

Alternatives considered:
- Dense, information-rich UI: fits more on screen, but is unreadable and mis-tappable at 5 inches in a venue.
- Touchscreen as the primary selection path: possible, but competes with the CV-driven selection model and live ergonomics.

## Risks / Trade-offs

- Emulation gives false confidence versus the real panel -> Mitigation: maintain a hardware validation checklist (contrast, touch accuracy, refresh/latency, orientation, ergonomics).
- Choosing a hard-to-emulate UI framework slows iteration -> Mitigation: weight emulability in the framework decision and prefer a web/kiosk UI.
- 800x480 is too small for the intended information -> Mitigation: prioritize glanceable status and large targets; move detail to secondary screens or the phone/Bluetooth surface.
- Touch emulation diverges from real capacitive behavior -> Mitigation: validate gestures and target sizes on hardware before finalizing.
- UI work drifts ahead of the base-platform framework decision -> Mitigation: keep this spike to direction and feasibility, not implementation.

## Migration Plan

No runtime migration is required because this change adds planning artifacts only. Research output should feed `decide-base-platform` (kiosk/display approach and UI framework), and coordinate with `research-bluetooth-control-channel` on the shared control/status surface.

Rollback is limited to removing this OpenSpec change before implementation.

## Open Questions

- Is the chosen panel DSI or HDMI, and what is its confirmed resolution, orientation, and touch controller?
- Is the UI framework a web/kiosk app, a native toolkit, or a web UI in a native shell, and how does emulability weigh in that choice?
- What is the minimal emulation harness for the team: Chromium kiosk at 800x480 with Force touch, a fixed-size window, or both?
- Which UI responsibilities belong on the touchscreen versus CV-driven selection versus the Bluetooth/phone surface?
- What are the concrete legibility and touch-target requirements for the venue lighting and ergonomics we expect?
- Do we need a device-frame preview (pinned 800x480 + simulated touch) for non-engineer design iteration?
