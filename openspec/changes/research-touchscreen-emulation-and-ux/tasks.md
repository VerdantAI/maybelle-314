## 1. Panel and Stack Confirmation

- [ ] 1.1 Confirm the target 5-inch panel: interface (DSI/HDMI), resolution (expected 800x480), touch controller, orientation, and driver requirements on the Pi 5.
- [ ] 1.2 Record the Pi display/touch stack (KMS/DRM, libinput, kiosk/browser or native toolkit) relevant to rendering and touch input.

## 2. Emulation Feasibility

- [ ] 2.1 Evaluate web/kiosk emulation: Chromium `--kiosk` at 800x480, Chrome DevTools device mode with "Force touch", and the DevTools Protocol Emulation domain for automated touch.
- [ ] 2.2 Evaluate a fixed-size desktop window stand-in (native or web) at 800x480, including mouse-as-touch and touch-monitor options.
- [ ] 2.3 Evaluate full-system Pi 5 emulation via QEMU and record why it is or is not feasible (no `raspi5` machine type, no GPU/DSI/touch modeling).
- [ ] 2.4 Recommend the development emulation approach and confirm it is sufficient to build the UI without the physical panel.

## 3. Hardware Validation Boundary

- [ ] 3.1 List properties that require the real panel: contrast under venue lighting, capacitive touch accuracy/calibration, refresh/latency, orientation, and finger ergonomics.
- [ ] 3.2 Define a minimal on-hardware validation checklist to run before finalizing UI layouts.

## 4. UI/UX Direction

- [ ] 4.1 Define layout constraints for 800x480 landscape: touch-target sizing, spacing, type scale, and contrast/dark-theme for low light.
- [ ] 4.2 Identify the core screens and glanceable status (song/channel/bank, transport, clock lock, ES-9 activity) and any manual-override interactions.
- [ ] 4.3 Define the division of responsibilities between the touchscreen, CV-driven selection, and the Bluetooth/phone control surface.
- [ ] 4.4 Note accessibility considerations (colorblind-safe, readable at distance) and immediate-feedback/latency expectations.

## 5. Decision Handoff

- [ ] 5.1 Recommend the emulation approach and its tooling for the base-platform kiosk/display decision.
- [ ] 5.2 Summarize how emulability should weigh in the UI-framework choice.
- [ ] 5.3 List follow-up spikes before implementing the UI or an emulation harness.
- [ ] 5.4 Feed findings into `decide-base-platform` and coordinate with `research-bluetooth-control-channel`.

## 6. Verification

- [ ] 6.1 Review the research output against every `touchscreen-emulation-and-ux-research` requirement.
- [ ] 6.2 Run OpenSpec validation or status checks for the completed change.
