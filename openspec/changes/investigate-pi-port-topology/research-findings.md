# Pi Port Topology — Research Findings (Web)

**Type:** Web-research output (desk research only). No physical Pi 5 bench was used.
**Retrieval date:** 2026-07-22.
**Scope:** Raspberry Pi 5 physical I/O and power budget for hosting the Expert Sleepers
ES-9 (USB), the official Raspberry Pi Touch Display 2 (5-inch, 720x1280 portrait, DSI),
and optional USB MIDI controllers simultaneously, plus thermal/cooling for sustained live use.

This document answers the **web-doable** parts of `tasks.md` and maps findings onto each
Requirement in `specs/pi-port-topology-research/spec.md`. Claims that can only be settled by a
physical Pi 5 (real current draw, USB enumeration/timing, latency, thermal under load) are
collected in the final **Open / needs bench hardware** section and are NOT asserted as settled.

---

## Key numbers up front (the load-bearing facts)

| Item | Value | Source |
|---|---|---|
| Pi 5 USB host ports | 2x USB 3.0 (5 Gbps) + 2x USB 2.0 | [Pi 5 product page / datasheet] |
| Pi 5 display/camera connectors | 2x 4-lane MIPI DSI/CSI (dual-purpose, 22-way FFC) | [Pi 5 product page] |
| Pi 5 power input | USB-C, 5V/5A PD (25W @ 5V) | [Pi 5 power supplies doc] |
| Official PSU | 27W USB-C PD, 5.1V/5A | [27W PSU product brief] |
| USB downstream budget (all 4 ports combined) | **600 mA default; auto-raised to 1.6 A when a 5A/PD supply is detected** | [Pi 5 power supplies doc], [USB PD white paper] |
| ES-9 power | **Eurorack bus-powered** (+12V 451 mA, -12V 133 mA @96k); NOT USB bus-powered | [Expert Sleepers ES-9 page] |
| ES-9 USB | Class-compliant USB 2.0, USB-C, 16-in/16-out, 44.1–96 kHz, 24-bit | [Expert Sleepers ES-9 page] |
| Touch Display 2 (5") power | From **GPIO header 5V** (pins 2 & 6) — not USB, not a separate supply | [Touch Display 2 doc] |
| Touch Display 2 (5") FFC on Pi 5 | Supplied **22-way to 15-way FFC**, either DISP/CAM connector | [Touch Display 2 doc] |
| Active Cooler steady temp under load | ~60–63°C; throttle onset 80°C (soft) / 85°C (hard) | [Heating & cooling Pi 5] |

The two decisive facts for the power question: **the ES-9 takes its power from the Eurorack bus, not from the Pi's USB**, and **the Touch Display 2 takes its power from the GPIO 5V pins, not from the USB downstream budget**. Both are therefore effectively "free" against the constrained 1.6 A USB-peripheral limit.

---

## Findings by Requirement

### Requirement: Raspberry Pi 5 port inventory

**Settled by web/spec research.**

Raspberry Pi 5 physical I/O relevant to Maybelle:

- **Power in:** single USB-C connector, 5V/5A with USB-PD negotiation. This is the recommended power path; it is *not* a data host port.
- **USB host:** 2x USB 3.0 (5 Gbps) + 2x USB 2.0 (480 Mbps). All four hang off the RP1 I/O controller. The two USB 3.0 ports can run at full 5 Gbps simultaneously (unlike Pi 4, where the two USB3 ports shared a single 5 Gbps link).
- **Display/camera:** 2x 4-lane MIPI connectors, each usable as **DSI (display) or CSI (camera)**. These are the newer 22-way (0.5 mm) FFC connectors, distinct from the 15-way connectors on Pi 4. Labeled DISP0/CAM0-style dual-purpose.
- **PCIe:** PCIe 2.0 x1 via the FPC connector (for NVMe HAT etc.) — an alternative fast-storage/expansion path that does not consume a USB port.
- **GPIO:** 40-pin header (includes 5V/GND pins used to power the Touch Display 2).
- **Networking:** Gigabit Ethernet; PoE+ available via a HAT (would occupy the GPIO/PoE header area).
- **Video out:** 2x micro-HDMI (not needed if the DSI Touch Display 2 is the primary UI).
- **Service/debug:** UART debug header; dedicated 4-pin fan/JST connector for the Active Cooler.

Port-role candidates for Maybelle:
- **Power:** USB-C (official 27W PSU) — see power requirement below.
- **ES-9 data:** one USB port (see ES-9 requirement; USB 3.0 port recommended for a dedicated controller path even though the ES-9 is a USB 2.0 device).
- **Display:** one MIPI connector as DSI + GPIO 5V power cable.
- **Storage:** microSD (boot) and/or NVMe via PCIe HAT (keeps USB free) or a USB SSD.
- **USB MIDI / service (keyboard/mouse):** remaining USB ports.

---

### Requirement: Power topology comparison

**Mostly settled by web/spec research; real draw needs bench.**

**USB downstream current limit is the one genuine constraint.** Per the official power-supplies
documentation and the USB-PD white paper: the total current across all four USB ports is limited
to **600 mA by default**, and this is raised to **1.6 A only when the Pi has negotiated 5V/5A with
a PD supply** (or `usb_max_current_enable=1` / `PSU_MAX_CURRENT=5000` is forced in firmware config).
With any lower supply, first boot shows the "current draw restricted to 600 mA" warning.

Consequence for Maybelle: use the **official 27W PSU (or an equivalent true 5V/5A USB-PD supply)**
so the 1.6 A USB budget is unlocked. This is the primary reason to standardize on the official PSU.

What actually draws on each rail:

- **Pi 5 board itself** (SoC + RP1 + RAM): draws from the 5V/5A input directly, not from the USB
  budget. Idle is a few watts; the 5A supply gives generous headroom over the board + peripherals.
- **Touch Display 2 (5")**: powered from **GPIO 5V (pins 2 & 6)** — this comes off the same
  5V/5A input rail but is **not** metered against the 1.6 A *USB* limit. The original Touch Display
  drew ~200 mA @5V typical at max brightness; the TD2 5-inch is expected to be the same order
  (single-digit-hundreds of mA). *Exact figure not published — confirm on bench.*
- **ES-9**: **draws ZERO from the Pi.** It is Eurorack-bus-powered (+12V 451 mA / -12V 133 mA);
  the USB cable carries data only. This removes the ES-9 entirely from the Pi power budget.
- **USB MIDI controller(s)**: class-compliant USB MIDI devices are typically bus-powered and low
  draw (commonly <100 mA; small keystep/beatstep-class controllers are self-powered or modest).
  These, plus any USB stick/SSD and service keyboard/mouse, are what actually consume the 1.6 A
  USB budget — and 1.6 A is ample for that set.

Power-source options compared:

| Option | Current capacity | Mechanical fit | Grounding/noise | Boot reliability | Serviceability | Notes |
|---|---|---|---|---|---|---|
| **Official 27W USB-C PSU** (recommended baseline) | 5V/5A; unlocks 1.6A USB | Simple, but external brick + USB-C strain-relief needed | Separate mains ground from rack; star-ground concerns exist but decoupled from rack rails | Best — PD handshake known-good, no undervoltage warnings | Easy: standard cable swap | Only path that guarantees the 1.6A USB limit is lifted without config hacks |
| **Rack-powered 5V regulator → USB-C** | Depends on design; must sustain ≥5A cleanly and must present a valid PD/`PSU_MAX_CURRENT` signal or USB stays capped at 600 mA | Integrated, no external brick | Shares rack ground — must validate for audio/CV noise into the ES-9 | Risk: undervoltage/brownout under transient load; must fake PD or set firmware override | Harder — custom circuit | Attractive for a single-cable module but is a real electrical-design project; defer |
| **PoE+ HAT** | Up to ~25W class depending on HAT | Consumes GPIO/HAT space — conflicts with the Touch Display 2 GPIO power cable and enclosure depth | Isolated supply (good noise story) | Good, but PoE+ HATs feed via GPIO 5V and may not reliably assert the 5A/1.6A USB unlock | Single Ethernet cable is clean on stage | GPIO-space conflict with the display power cable is the main blocker |
| **Powered USB hub** | Offloads USB-device power from the Pi | Extra box + cable | Another ground path | N/A for Pi power itself | Adds a failure point | Only needed if USB *device* power or port count exceeds budget — not needed for the baseline set |

**Recommended first power topology:** official 27W USB-C PSU into the Pi's USB-C input. Fallbacks,
in order: (1) any certified 5V/5A USB-PD supply; (2) a validated rack 5V→USB-C feed *with* a proven
PD/`PSU_MAX_CURRENT=5000` current-limit assertion, pending noise validation; (3) PoE+ only if the
GPIO/display-power conflict is resolved.

---

### Requirement: ES-9 USB connection validation

**Partly settled (electrical/bandwidth headroom); enumeration/latency/hotplug need bench.**

- The ES-9 is **class-compliant USB 2.0** over a **USB-C** socket, so an A-to-C (or C-to-C into a
  USB port) *data-capable* cable from a Pi host port is the correct path. No vendor driver is
  needed on Linux for class-compliant USB audio (ALSA `snd-usb-audio`). *The Expert Sleepers page
  documents macOS/iOS/Windows only; Linux class-compliant operation is the expected—but
  bench-unconfirmed—path.*
- **Power:** because the ES-9 is Eurorack-bus-powered, connecting it to the Pi does **not** load the
  Pi's USB budget. This is the single biggest simplifier in the whole topology.
- **Bandwidth headroom (settled by arithmetic):** 16 in + 16 out @ 96 kHz, 24-bit ≈
  16 × 96,000 × 3 bytes × 2 directions ≈ **9.2 MB/s ≈ ~74 Mbit/s of payload**, comfortably inside
  USB 2.0's 480 Mbps. So bandwidth is not a limiter even at max rate/channel count. USB isochronous
  audio *timing/interrupt* behavior on the Pi, however, is a bench question (see open items).
- **Port choice:** the ES-9 is a USB 2.0 device and will work in any Pi port. Reserving one of the
  **USB 3.0 ports** for it is still recommended, because on Pi 5 the USB3 ports have more dedicated
  controller bandwidth and keep the ES-9's path away from other USB2 traffic. **DSI (display) is a
  wholly separate interface from USB**, so the ES-9 on USB3 and the Touch Display 2 on DSI do not
  contend at all.
- Whether the ES-9 needs a **powered hub** for stability: no power reason exists (rack-powered), so a
  hub would only be for enumeration/timing robustness — a bench question, not a foregone need.

---

### Requirement: External MIDI controller input paths

**Web-comparable; per-device latency/naming stability need bench.**

Four candidate paths (from `design.md`), with web-level assessment:

| Path | Port cost on Pi | Power | Latency (expected) | Naming stability | Config complexity | Notes |
|---|---|---|---|---|---|---|
| **Direct USB MIDI → Pi** | 1 USB port (USB2 is fine) | Device bus-powered, low draw, within 1.6A budget | Lowest / native | ALSA/`amidi` device index can renumber on replug — needs stable-ID rule (by-id udev) | Low | Simplest for USB-class controllers (BeatStep Pro, KeyStep Pro both expose USB MIDI) |
| **DIN MIDI → USB-MIDI interface → Pi** | 1 USB port | Interface bus-powered | Native + interface | Same renumber caveat | Low–med | Supports DIN-only controllers; adds an adapter |
| **DIN MIDI → ES-9 MIDI breakout** | 0 extra USB (rides the ES-9) | Rack-powered | Depends on ES-9 MIDI path | Ties to ES-9 device | Med | ES-9 exposes only **1 in / 1 out on the optional breakout** — scarce; consumes the single MIDI-in |
| **Controller CV/gate → ES-9 analog inputs** | 0 USB | Rack-powered | Analog / immediate | N/A (analog) | Med (needs quantize/debounce) | Aligns with Maybelle's CV selection model; consumes scarce ES-9 analog inputs |

Web-settled: all four are physically viable; the ES-9 MIDI breakout is a **single** in/out, so it is
the scarcest resource. Direct USB MIDI is the lowest-friction default and fits the USB power budget
trivially. Keeping the path pluggable (per `design.md`) is the right call. **Latency numbers, and
whether device indices stay stable across hotplug/reboot, are bench items.**

---

### Requirement: Recommended topology handoff

See the RECOMMENDATION section below (baseline + fallbacks + cable list + device-discovery
assumptions). Requirement satisfied at the web-research level; hardware confirmation is the bench
spike.

---

## RECOMMENDATION — proposed baseline port/power topology

**Power**
- Feed the Pi from the **official Raspberry Pi 27W USB-C PD supply** (5.1V/5A). This is the only
  option that guarantees the USB downstream limit is lifted from 600 mA to **1.6 A** without firmware
  overrides, and gives generous board+display headroom on the 5V rail.

**Display**
- **Touch Display 2 (5-inch, 720x1280 portrait)** on either MIPI connector configured as **DSI**,
  using the **supplied 22-way-to-15-way FFC**, powered by the **supplied GPIO 5V power cable**
  (GPIO pins 2 & 6). DSI is independent of USB, so it never contends with the ES-9.

**ES-9 (primary audio/CV)**
- Connect the ES-9's USB-C to a **Pi USB 3.0 port** via a data-capable cable (USB-A-to-C or C-to-C
  into the USB3 port). ES-9 is Eurorack-bus-powered, so it adds **zero** to the Pi power budget.
  Reserve and physically label this port in the enclosure.

**Storage**
- Prefer **NVMe via the PCIe HAT** (or boot from microSD) so storage does not consume a USB port or
  USB current. A USB SSD is an acceptable fallback (counts against the 1.6 A budget but is modest).

**MIDI controller (optional)**
- **Direct USB MIDI** into a **USB 2.0 port** as the default; low power, native latency. Keep DIN-via-
  USB-interface and ES-9-breakout as documented fallbacks. Reserve CV/gate-through-ES-9 for the
  performance selection model.

**Service access**
- The remaining USB 2.0 port for a service keyboard/mouse or dongle. All of {USB MIDI + storage +
  service} together sit well inside 1.6 A.

**Powered hub?**
- **Not required** for the baseline. The ES-9 (rack-powered) and display (GPIO-powered) are off the
  USB budget, leaving only low-draw USB devices. Add a powered hub only if the device count grows
  beyond the 4 ports or if bench testing reveals enumeration/power-isolation problems.

**Cooling**
- Fit the **official Active Cooler** (or equivalent active heatsink+fan on the 4-pin fan header) for
  sustained live use. Web reports put it at ~60–63°C under load versus 80°C soft / 85°C hard throttle
  — ample margin, and firmware manages the fan (on at 60°C, ramp at 67.5°C, full at 75°C). Passive
  cooling risks throttling in a warm, enclosed, on-stage box.

**Power-budget verdict:** With the official 27W PSU, **everything can be hosted directly and
simultaneously**. No powered hub and no external display power supply are needed, because the two
biggest peripherals draw their power elsewhere (ES-9 from the Eurorack bus, display from GPIO 5V),
leaving only low-draw USB devices inside the 1.6 A limit.

**Cable / parts list**
- Official 27W USB-C PSU.
- Data-capable USB cable, Pi USB3 (A) → ES-9 (C).
- Touch Display 2 (5") with its supplied 22-way-to-15-way FFC + GPIO power cable.
- Official Active Cooler.
- Optional: PCIe NVMe HAT + NVMe SSD (or USB SSD); USB MIDI cable/interface; enclosure port labels.

**Runtime device-discovery assumptions**
- Select the ES-9 by stable identifier (ALSA card name / USB by-id), not by numeric index, since
  USB/ALSA indices can renumber across reboot/hotplug.
- Treat MIDI controllers as optional: match by stable by-id name; degrade gracefully (fall back to
  rack CV selection) when absent or renamed.

---

## Open / needs bench hardware (NOT settled by web research)

The following require a physical Pi 5 + ES-9 + Touch Display 2 rig and are flagged blocked:

1. **Real current draw** — actual mA on the 5V rail for the Pi 5 board + Touch Display 2 (5") under
   live load, and the true USB draw of the chosen MIDI controller/storage, to confirm margin under
   the 1.6 A USB limit and the 5A supply overall. (TD2 5" draw is unpublished.)
2. **ES-9 enumeration & visibility on Linux** — confirm class-compliant enumeration on Raspberry Pi
   OS (ALSA `snd-usb-audio`), channel count (16/16), sample-rate support (44.1–96 kHz), and MIDI
   visibility via the breakout. Expert Sleepers documents macOS/iOS/Windows only.
3. **USB isochronous audio timing / xruns / latency** — round-trip latency and stability of the ES-9
   over the Pi's USB stack (JACK/PipeWire/ALSA), with and without a powered hub, at the target
   buffer sizes. Bandwidth headroom is fine on paper; interrupt/scheduling behavior is not.
4. **Hotplug / renumbering behavior** — device index stability for ES-9 and MIDI controllers across
   replug and reboot; validate the by-id selection rule.
5. **MIDI path latency** — measured latency and jitter for direct-USB vs DIN-via-interface vs ES-9
   breakout vs CV/gate-through-ES-9, and fitness for song/track/channel selection.
6. **Thermal under the real enclosure** — sustained-load temperatures inside the actual (possibly
   sealed, rack-adjacent, warm) enclosure with the Active Cooler, versus the open-bench ~60–63°C
   figures, to confirm no throttling during a long set.
7. **Rack-power option (if pursued)** — noise/grounding measurements of a rack 5V→USB-C feed into the
   ES-9's CV/audio, and confirmation it asserts the 5A current-limit unlock.

---

## Sources

- [Raspberry Pi 5 product page](https://www.raspberrypi.com/products/raspberry-pi-5/)
- [Raspberry Pi 5 power supplies documentation](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#power-supplies) — USB current limit 600 mA default / 1.6 A with 5A PD supply; `usb_max_current_enable`
- [Raspberry Pi power-supplies source (documentation repo)](https://github.com/raspberrypi/documentation/blob/master/documentation/asciidoc/computers/raspberry-pi/power-supplies.adoc)
- [Raspberry Pi 27W USB-C Power Supply product brief (datasheet, PDF)](https://pip.raspberrypi.com/documents/RP-008245-DS-27w-usb-c-power-supply-product-brief.pdf)
- [Raspberry Pi — Buy a 27W USB-C Power Supply](https://www.raspberrypi.com/products/27w-power-supply/)
- [USB Power Delivery on Raspberry Pi 5 — white paper (RP-009856-WP-1, PDF)](https://pip-assets.raspberrypi.com/categories/685-app-notes-guides-whitepapers/documents/RP-009856-WP-1-USB%20Power%20delivery%20on%20Raspberry%20Pi%205.pdf)
- [Raspberry Pi Touch Display 2 — documentation](https://www.raspberrypi.com/documentation/accessories/touch-display-2.html) — GPIO 5V power, 22-way-to-15-way FFC on Pi 5, 720x1280
- [Getting started with Raspberry Pi Touch Display 2](https://www.raspberrypi.com/news/getting-started-with-raspberry-pi-touch-display-2/)
- [Expert Sleepers ES-9 — official page](https://www.expert-sleepers.co.uk/es9.html) — class-compliant USB 2.0, USB-C, Eurorack bus power (+12V 451 mA / -12V 133 mA), 16/16, 44.1–96 kHz, MIDI breakout 1 in/1 out
- [Heating and cooling Raspberry Pi 5 (official news / Active Cooler behavior)](https://www.raspberrypi.com/news/heating-and-cooling-raspberry-pi-5/)
