# Research Findings: Touchscreen Emulation and UI/UX

**Method:** Web research (WebSearch/WebFetch) against official/vendor documentation.
**Retrieval date:** 2026-07-22.
**Status:** Complete; findings mapped onto each Requirement in `specs/touchscreen-emulation-and-ux-research/spec.md`. On-hardware items are called out and remain open.

## Intro

Maybelle 314 needs a dependable way to iterate its 5-inch touchscreen UI on dev laptops without the physical panel, plus an initial UI/UX direction for a small, glanceable, finger-operated live-performance surface. This document confirms the panel/stack facts, evaluates three emulation approaches and recommends one, draws the hardware-validation boundary, and lays out a concrete 720x1280 portrait UI/UX direction. It feeds `decide-base-platform` (kiosk/display + UI-framework) and coordinates with `research-bluetooth-control-channel`.

> **PANEL LOCKED (2026-07-22):** the display is the **official Raspberry Pi Touch Display 2, 5-inch — 720x1280 native portrait, DSI, 5-finger capacitive, viewing area 63x111.5 mm, ~11.4 px/mm (~290 ppi)**. This supersedes the earlier 800x480-landscape assumption throughout this document. Two consequences: (a) the layout is **portrait 720x1280** (a tall narrow stack), not landscape; (b) at ~290 ppi, **touch-target and type sizes must be anchored in physical mm and converted at ~11.4 device-px/mm** — a generic "48 px" target is ~4.2 mm here and too small. Sizes below are given in mm with native-scale device-px; the CSS-px value depends on the kiosk's configured device-scale-factor / devicePixelRatio (see the Requirement 4 note).

**Headline decisions:** (1) Emulate the *display target* (720x1280 portrait + touch) in a **Chromium `--kiosk` window plus Chrome DevTools device mode with touch emulation**, not the Pi hardware. (2) This weighs strongly toward a **web/kiosk UI framework**, because it is the only stack that emulates trivially and 1:1. (3) A **hardware validation checklist** guards against emulation over-confidence. (4) The touchscreen is primarily **status + config + manual override**; **CV drives real-time selection**; the **Bluetooth/phone** surface handles heavier config away from the rack.

---

## Requirement 1 — Target panel confirmation

**Panel CONFIRMED: official Raspberry Pi Touch Display 2, 5-inch.** Native **720x1280 portrait**, 24-bit RGB, **5-finger capacitive**, DSI, auto-detected/driver-free on Pi 5, powered from the Pi GPIO (5V pin 2 / GND pin 6), viewing area **63 x 111.5 mm**, rotatable 90/180/270 ([Raspberry Pi Touch Display 2 docs](https://www.raspberrypi.com/documentation/accessories/touch-display-2.html); purchased unit = Micro Center 697845, "5-inch portrait"). Pixel density is **720 / 63 mm ≈ 11.4 px/mm (~290 ppi)** — roughly phone-class.

**Density implication (the reason locking the panel mattered).** At ~290 ppi, physical size and pixel count diverge sharply from the common third-party 800x480 5-inch DSI panels (~192 ppi, viewing area ~108 x 64.8 mm) that the proposal originally assumed ([Waveshare 5inch DSI LCD](https://www.waveshare.com/5inch-dsi-lcd.htm), [PiShop 800x480 listing](https://www.pishop.us/product/5inch-capacitive-touch-display-for-raspberry-pi-dsi-interface-800-480/)). All touch-target and type sizing must therefore be **anchored in mm and converted at ~11.4 device-px/mm**, and the layout is **portrait 720x1280** (native) unless deliberately rotated to a **1280x720 landscape** at the rack (a mount/rotation decision, see below).

**Orientation / driver stack (Pi 5, Bookworm).** DSI panels are driver-free but orientation on Bookworm's **KMS driver (`vc4-kms-v3d`)** is **not** set by the legacy `display_rotate`. Rotation is done via the **`dtoverlay` line** (e.g. `dtoverlay=vc4-kms-dsi-...,rotate=90`) and/or the **Wayland compositor** using **`wlr-randr --output DSI-1 --transform 90`**; touch mapping after rotation is handled by the touchscreen overlay params (`swapxy`, `invx`, `invy`, `rotation`) and/or a **libinput `CalibrationMatrix`** ([Raspberry Pi Linux issue #6085 / forum threads on Pi5 DSI rotation](https://forums.raspberrypi.com/viewtopic.php?t=379738)). **On-hardware:** confirmed orientation, touch-axis mapping, and calibration remain hardware tasks.

## Requirement 1 (cont.) — Pi display/touch stack

- **Graphics:** Bookworm on Pi 5 defaults to **Wayland (labwc compositor)** with the **KMS/DRM `vc4-kms-v3d`** driver; DSI shows up as output `DSI-1`.
- **Touch input:** delivered through **libinput**; rotation requires a matching input transform (compositor transform or libinput calibration matrix), otherwise touch coordinates are offset ([Raspberry Pi kiosk tutorial](https://www.raspberrypi.com/tutorials/how-to-use-a-raspberry-pi-in-kiosk-mode/), rotation forum threads above).
- **UI shell:** the Pi runs a browser fullscreen. **Chromium `--kiosk`** launched from **`~/.config/labwc/autostart`** is the documented Pi 5 kiosk path (see Requirement 2).

## Requirement 2 — Emulation feasibility assessment

Three approaches evaluated; the display-target approaches are feasible, full-system is not.

### 2a. Web/kiosk emulation at 720x1280 with browser touch emulation — **FEASIBLE (recommended)**
- **Runtime shell is reproducible on-device and off:** the same web UI runs under **Chromium `--kiosk`** on the Pi. The official Pi kiosk invocation is `chromium <url> --kiosk --noerrdialogs --disable-infobars --no-first-run --start-maximized` from the labwc autostart file ([Raspberry Pi kiosk tutorial](https://www.raspberrypi.com/tutorials/how-to-use-a-raspberry-pi-in-kiosk-mode/)).
- **Exact 720x1280 portrait + touch on a laptop:** Chrome DevTools **Device Mode** lets you **Add custom device** with an explicit **width/height (720x1280)**, a **device pixel ratio** matching the kiosk config, and a **Device Type** of **"Desktop (touch)"** or **"Mobile"**, both of which make the page **fire `touch` events** (touchstart/move/end) instead of `click` ([Chrome DevTools device mode docs](https://developer.chrome.com/docs/devtools/device-mode)). The **Sensors** panel additionally exposes **"Emulate touch screen" / force-touch** overrides ([DevTools device mode](https://developer.chrome.com/docs/devtools/device-mode)). Note: DevTools Device Mode reproduces the CSS-px geometry and touch event model, but **cannot reproduce the panel's physical size** — a target that looks fine at 720x1280 in a laptop viewport may still be too small in the hand; that check belongs to the hardware pass (Requirement 3).
- **Programmatic/automated touch:** the **Chrome DevTools Protocol Emulation domain** exposes **`Emulation.setEmitTouchEventsForMouse`** (turn mouse input into touch, `mobile`/`desktop`) and **`Emulation.setTouchEmulationEnabled`**, drivable from Puppeteer/Playwright for automated touch tests ([CDP Emulation domain](https://chromedevtools.github.io/devtools-protocol/tot/Emulation/)).
- **Net:** the display geometry, touch event model, and kiosk shell are all reproducible on a dev machine with high fidelity to the device.

### 2b. Fixed-size desktop window stand-in at 800x480 — **FEASIBLE (supporting role)**
- A plain 720x1280 window (a `chromium --window-size=720,1280 --app=<url>`, or a native window pinned to 720x1280) is the fastest layout/ergonomics check; **mouse-as-touch is acceptable for early iteration**, and plugging in any USB touch monitor adds real multi-touch realism. Good for non-engineer design review (a pinned "device frame"), weaker than 2a on the touch event model unless combined with DevTools touch emulation.

### 2c. Full-system Pi 5 emulation via QEMU — **NOT FEASIBLE**
- QEMU provides Raspberry Pi machine types only **`raspi0, raspi1ap, raspi2b, raspi3ap, raspi3b, raspi4b`** — there is **no `raspi5`** as of 2026 ([QEMU raspi machine docs](https://www.qemu.org/docs/master/system/arm/raspi.html), corroborated by [QEMU-on-Pi5 discussion](https://github.com/sdr-enthusiasts/docker-shipfeeder/issues/17)).
- Even the existing Pi machine types **do not model the GPU (no acceleration), the DSI display, or USB/DSI touch peripherals**, so a full-system image would render nothing useful for touchscreen UI work and would still miss the exact hardware being emulated. Full-system QEMU is therefore **not a practical development path** for this UI.

### 2d. Recommendation (sufficiency)
**Adopt 2a as the primary harness and 2b as a supporting stand-in.** Build the UI as a web app; iterate in Chrome at a 720x1280 portrait custom device with a touch Device Type (and CDP touch emulation for automated tests); use a fixed 720x1280 window / USB touch monitor for quick ergonomics and design review. This is **sufficient to build and iterate the UI without the physical panel**, provided the hardware-validation checklist (Requirement 3) is run before finalizing layouts — geometry emulates 1:1, but physical target size does not.

## Requirement 3 — Hardware validation boundary

Emulation reproduces geometry and the touch **event** model, but **not** the physical/perceptual properties. The following require the real panel:

- **Contrast/legibility under venue lighting** (dark stage, colored/strobing light, glare, viewing angle of an IPS panel).
- **Capacitive touch accuracy & calibration** — real fingertip footprint (~16-20 mm), false/edge touches, post-rotation axis mapping.
- **Refresh/latency** — actual DSI refresh, compositor + browser paint latency, perceived tap-to-feedback delay.
- **Orientation/rotation** correctness (display transform + matching touch input transform) on the actual panel.
- **Finger ergonomics at true 5-inch physical size** — target reach, thumb zones, mis-taps, use while standing at the rack.
- **Backlight/brightness range** adequacy for both dark venues and bright rehearsal spaces.

### Minimal on-hardware validation checklist (run before finalizing layouts)
1. Boot the kiosk on the real panel; confirm **native 720x1280 resolution + intended orientation** (native portrait, or the chosen rotation), and that **touch coordinates track the display** after any rotation.
2. **Contrast check in a dark room and under stage-style colored light** — confirm all status text/icons remain readable; verify at an arm's-length glance.
3. **Touch-target hit test** — tap every primary control with a fingertip (and thumb) standing at the rack; log mis-taps; confirm real targets meet the **~9 mm (≈100 device-px) minimum** in practice, and that live controls at ~12 mm feel comfortable.
4. **Latency feel** — measure/observe tap-to-visual-feedback; confirm it feels immediate (<~100 ms target) during live use.
5. **Glance test** — from ~0.5-1 m, confirm song/channel/bank, transport, and clock-lock status are readable in <1 second.
6. **Backlight sweep** — confirm min/max brightness usable in dark and lit conditions.
7. **Sustained-use / burn-in** — leave a fixed status screen on for a long set; confirm no thermal/refresh issues.

## Requirement 4 — Initial UI/UX direction (720x1280 portrait)

> **Sizing note (density + scale factor).** All sizes below are anchored in **physical mm** and converted at **~11.4 device-px/mm** (the confirmed panel). Whether those equal your **CSS px** depends on the kiosk's **device-scale-factor / devicePixelRatio**: at native scale (factor 1, a 720-CSS-px-wide viewport) mm×11.4 = CSS px directly; if you run Chromium at `--force-device-scale-factor=2` (a 360-CSS-px-wide viewport, phone-like), halve the CSS-px numbers — the **physical mm stay the target of record**. Decide the kiosk scale factor early and set the DevTools custom device to match.

### Layout constraints
- **Canvas:** fixed **720x1280 portrait** (native), no scrolling of primary status; treat it like an instrument face, not a web page. A **1280x720 landscape** variant is possible if mounted rotated (decide at mount time; requires a matching touch-input transform).
- **Vertical stack** (portrait): a persistent top **status block** (largest, glanceable), a middle **secondary-status / selection** region, and a bottom **primary-action row** within easy thumb reach. The tall/narrow aspect suits a stacked instrument face; avoid dense multi-column layouts.
- **Grid:** a coarse grid with generous gutters (**≥1 mm ≈ 11 px** between targets).

### Touch-target sizing (mm-anchored; device-px at native scale)
- **Minimum interactive target: ~9 mm ≈ 100 device-px square**, with **≥1 mm (~11 px) spacing**. This meets **Material Design 48 dp ≈ 9 mm** and the **Apple HIG 44 pt** / **WCAG 2.5.5** floors *in physical terms* — note those guidelines are density-independent (dp/pt), so the ~290 ppi panel needs ~100 px, not 48 px, to hit the same physical size.
- **Primary live controls (transport, manual override, panic): ~12-13 mm ≈ 135-150 device-px** — bigger than the minimum, because these are hit fast, by feel, possibly in the dark. On the 720-px-wide portrait canvas this comfortably fits a row of ~5 large buttons.
- Rationale: fingertip contact is ~16-20 mm; ~9 mm is the small end of comfortable, so promote anything used live to the ~12 mm+ tier.

### Type scale (dark, glanceable; mm-anchored)
- **Primary status value (current song/channel number): ~10-14 mm cap height ≈ 130-180 device-px font size**, heavy weight — readable across the room.
- **Labels / secondary status: ~4-5 mm ≈ 50-64 device-px.**
- **Minimum body text: ~3.5 mm ≈ 40 device-px**; avoid smaller on this panel.
- Keep to **2-3 sizes**; rely on size + weight + position, not many fonts. (Halve the px if running at device-scale-factor 2.)

### Contrast / dark theme for low light
- **Default to a dark, high-contrast theme** (near-black background, off-white/high-luminance text). Meet **WCAG AA**: **>=4.5:1** for normal text, **>=3:1** for large text and for **UI component/graphic boundaries (1.4.11)**; aim **AAA (7:1)** for the always-on status readout ([WCAG 1.4.3 Contrast (Minimum)](https://www.w3.org/WAI/WCAG21/Understanding/contrast-minimum.html), [WebAIM contrast](https://webaim.org/articles/contrast/)).
- Avoid pure-white full-screen fields (glare/afterimage in a dark venue); use dark surfaces with bright text and a small number of saturated accent colors for state.
- Consider a brightness/night control given the venue-lighting variance.

### Glanceable status set (always visible)
- **Song / channel / bank** (current selection — the CV-driven state).
- **Transport state** (running / stopped, and clock source).
- **Clock lock** to Pamela's Pro Workout (locked vs. free/searching) — a top-priority indicator.
- **ES-9 output activity** (gates/triggers firing, per-channel liveness).
- Optional: bar/beat position, current bank page.
Each should be **legible in under a second from ~1 m**.

### Manual-override interactions (touchscreen-owned)
- Manual song/channel/bank selection when not driven by CV (setup/soundcheck), **transport start/stop/reset override**, **mute/panic (all-notes-off / stop outputs)**, and brightness. Keep these to a small set of **large, unambiguous, well-separated** controls with immediate visual confirmation.

### Division of responsibilities
- **CV-driven selection (real-time, primary):** song bank, song, channel bank, channel, transport, performance params — quantized/debounced/latched from rack CV per the README selection model. The touchscreen **reflects** this state; it does not compete to be the live selector.
- **Touchscreen (local, at the rack):** glanceable status, manual override/soundcheck, panic, brightness, and light config. Optimized for fast, gloved-finger-in-the-dark operation.
- **Bluetooth / phone surface (`research-bluetooth-control-channel`):** heavier/seated configuration, song-bundle management, detailed editing, diagnostics, and anything needing a keyboard or more screen real estate — done **away from** the rack. Avoid duplicating the live transport as the primary control here; keep it as a secondary/remote surface.

### Accessibility & feedback
- **Never encode state by color alone** — pair every color with **shape/icon/text/position** so red-green colorblind users can read state ([Section508 color usage](https://www.section508.gov/create/making-color-usage-accessible/), WCAG 1.4.1). Prefer a colorblind-safe accent set (e.g. blue/orange rather than red/green) and use luminance contrast for the primary distinction.
- **Readable at distance** — large primary readout, high contrast, minimal clutter.
- **Immediate feedback** — every tap gets instant visual (and where possible haptic-analog) confirmation; target **<~100 ms** tap-to-feedback; validate real latency on hardware (Requirement 3).

## Requirement 5 — Decision evidence handoff

- **Emulation approach + tooling (for `decide-base-platform` kiosk/display):** Chromium `--kiosk` on the Pi (labwc autostart), and off-device iteration via Chrome DevTools Device Mode (720x1280 portrait custom device, touch Device Type, device pixel ratio matched to the kiosk scale factor) + a fixed 720x1280 window / USB touch monitor; CDP Emulation domain for automated touch tests. Full-system QEMU is explicitly **out** (no `raspi5`, no GPU/DSI/touch modeling).
- **How emulability weighs on the UI-framework choice:** strongly favor a **web/kiosk UI**. A browser UI emulates **1:1** (same runtime on device and laptop, exact geometry, real touch events, free automation). A native GPU/DSI-bound toolkit would be **hard to emulate** given no Pi 5 machine type and no GPU/touch modeling, slowing iteration and gating it on hardware. This aligns with the README's Python-runtime + browser/kiosk bias; a Python backend serving a local web UI in Chromium kiosk is the low-risk default, with a native shell (e.g. Tauri) only if the ES-9 I/O spike forces it.
- **Cross-references:** feeds `decide-base-platform` (kiosk/display + UI framework); coordinate the shared status/control model with `research-bluetooth-control-channel`.

### Follow-up spikes before implementing the UI or an emulation harness
1. **Decide mount orientation** (native portrait 720x1280 vs. rotated 1280x720 landscape at the rack) and the **kiosk device-scale-factor**, then lock the CSS-px grid; panel model itself is confirmed (Touch Display 2 5-inch).
2. Stand up a **minimal kiosk harness** (Chromium kiosk + 720x1280 DevTools device profile at the chosen scale factor + optional USB touch monitor) as a reusable dev setup.
3. **On-hardware validation pass** (checklist above) once a panel is available.
4. Nail down the **status data model** shared with the Bluetooth/phone surface.
5. Prototype and hardware-test the **live control tier** (target sizes 64 px+, latency) before committing the visual system.

---

## Sources

- [Waveshare 5inch DSI LCD — product](https://www.waveshare.com/5inch-dsi-lcd.htm)
- [Waveshare 5inch DSI LCD — wiki](https://www.waveshare.com/wiki/5inch_DSI_LCD)
- [PiShop — 5inch Capacitive DSI 800x480](https://www.pishop.us/product/5inch-capacitive-touch-display-for-raspberry-pi-dsi-interface-800-480/)
- [Raspberry Pi Touch Display 2 documentation](https://www.raspberrypi.com/documentation/accessories/touch-display-2.html)
- [Raspberry Pi Touch Display (original 7-inch, 800x480) documentation](https://www.raspberrypi.com/documentation/accessories/display.html)
- [Raspberry Pi Pi5 DSI rotation forum thread](https://forums.raspberrypi.com/viewtopic.php?t=379738)
- [QEMU — Raspberry Pi machine types (no raspi5)](https://www.qemu.org/docs/master/system/arm/raspi.html)
- [QEMU on Raspberry Pi 5 discussion](https://github.com/sdr-enthusiasts/docker-shipfeeder/issues/17)
- [Chrome DevTools — Simulate mobile devices with Device Mode](https://developer.chrome.com/docs/devtools/device-mode)
- [Chrome DevTools Protocol — Emulation domain](https://chromedevtools.github.io/devtools-protocol/tot/Emulation/)
- [Raspberry Pi — How to use a Raspberry Pi in kiosk mode](https://www.raspberrypi.com/tutorials/how-to-use-a-raspberry-pi-in-kiosk-mode/)
- [WCAG 2.5.5 Target Size guidance (Apple 44pt / Material 48dp summarized)](https://testparty.ai/blog/wcag-target-size-guide)
- [WCAG 1.4.3 Contrast (Minimum) — W3C Understanding](https://www.w3.org/WAI/WCAG21/Understanding/contrast-minimum.html)
- [WebAIM — Contrast and Color Accessibility](https://webaim.org/articles/contrast/)
- [Section508.gov — Making Color Usage Accessible](https://www.section508.gov/create/making-color-usage-accessible/)
