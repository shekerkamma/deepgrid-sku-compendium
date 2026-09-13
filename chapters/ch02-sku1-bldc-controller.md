# Chapter 2: SKU-1 BLDC Controller
*Sheet 2 · FIG 5 · AEC-Q100 · UL94 · ISO 26262 ASIL-B/C · 130 nm CMOS (28 nm planned)*

## Core Idea
One die collapses the DRV83xx-class gate driver plus an external MCU into a single motor controller. It has a hardware field-oriented control datapath whose PID path settles in under 1 µs, a 5–120 V buck/boost rail on the same die, and runtime star/delta winding selection.

## Frameworks Introduced
- **Commutation in hardware, supervision in software.** The DGridRiscV core sets targets over the AXI bus; the FOC datapath closes the loop without the processor in the path.
  - Why it works: a sub-microsecond loop cannot survive interrupt latency or cache misses. The core only has to be deterministic about register writes.
  - Failure mode: if the hardware PID must be re-tuned per motor from firmware, "<1 µs tuning latency" says nothing about how fast the system is commissioned.
- **Current plus angle feedback closes the PID loop.** Sensors (Hall, quadrature or encoder, or sensorless) give rotor angle and speed; current is rebuilt per phase (iR iY iB) and fed back to the PID tuner, as the orange loop drawn on the sheet shows.
- **Power path on the same die.** A 5–120 V buck/boost converter, driven from a PWM pin, feeds PWM ×7 (6 bridge + 1 aux, up to 200 kHz), then pre-drivers for 3 half-bridges (R/Y/B, clock gating per phase). The pre-drivers imply external power FETs.
- **Runtime star/delta.** Winding configuration is selectable while running, trading torque at low speed against speed at the top end.

## Key Concepts
- **Host · control plane**: DGridRiscV ("custom 32-bit RISC-V · 200 MHz · no licence fee") with timers, WDT + CCU. SRAM 32 KB (up to 256 KB) and ReRAM 256 KB (up to 1 MB). Interfaces: I²C slave, UART ×4, SPI, CAN-FD ×2 and JTAG. The rate labels 1 MHz / 3 Mbps / 50 MHz / 5 Mbps appear to map to I²C, UART, SPI and CAN-FD as drawn.
- **Mixed signal**: 16-bit ADC (phase / bus / temperature), DAC ×4 (reference out), on-die temperature sense (−40 to +125 °C), CSA ×3 (current-sense amplifiers for iR iY iB).
- **FOC datapath**: PID tuner (hardware-integrated, <1 µs) → rect→polar (CORDIC, magnitude and angle) → D,Q→3-phase (inverse Park + Clarke, rotor frame to stator) → sine/trapezoid lookup tables (sinusoidal or 6-step).
- **Sensor + feedback**: Hall/quad/encoder interface, angle (ω), iR iY iB rebuild (transfer function), instant current (amplify + digitise).
- **Protection**: thermal, overcurrent, overvoltage; a fault triggers a latched gate shutdown.
- **ReRAM path**: sky130 experimental module, with an SCL eNVM fallback (text panel).

## Mental Models
- Think of **the core as the conductor and the FOC chain as the orchestra**: the timing claim belongs to the datapath.
- Use **"6 bridge + 1 aux"** to explain PWM ×7: three half-bridges need six gate signals, and the seventh drives the buck/boost converter.
- Treat **"no licence fee" as a cost line, not a feature**: it removes the per-unit core royalty from a $3–8 part.

## Anti-patterns
- **Calling 95% efficiency, <1 µs settling or 1 M+ ReRAM cycles measured**: all three are target lines on illustrative panels.
- **Implying 120 V power FETs are on die**: the sheet draws pre-drivers for external half-bridges.
- **Quoting "28 nm planned" from the header** without a plan anywhere else in the compendium; the roadmap keeps motor control at 130/180 nm.

## Reference Tables

| Characteristic panel | Axis | Target |
|---|---|---|
| Torque vs speed | 0 → rated rpm | falling curve, no number |
| Efficiency | 10 → 100% load | 95% |
| PID settling | 0 → 200 µs | <1 µs tune |
| ReRAM endurance | 10 k → 1 M writes | 1 M+ cycles |

| Market (rough internal estimate) | Value |
|---|---|
| Volume | 5–15 M/yr India 2W/3W EV + industrial BLDC |
| ASP | $3–8 |
| Addressable | ~$0.4–0.6 B India motor-control silicon |
| Use cases | EV traction, e-pumps, thermal management; robotics; drones (high-RPM efficiency, fast boot); HVAC compressors, smart appliances |
| Hooks (text panel) | captive in every ASWA joint and D100 module; BEE star-rating push to BLDC, ₹250–550 Cr/yr in fans alone |

## Worked Example
**How much control headroom does <1 µs buy? (reconstruction; the motor is an assumption)**
1. PWM period at the 200 kHz maximum = 5 µs, so a <1 µs PID update runs at least 5 times per PWM period.
2. At a 200 MHz clock, 1 µs = 200 clock cycles for the hardware path: ample for a CORDIC plus inverse Park/Clarke pipeline.
3. Assume a 4-pole-pair motor at 6,000 rpm. Electrical frequency = 4 × 6,000 / 60 = 400 Hz.
4. Three Hall signals give 6 sectors of 60° electrical, so a sector lasts 1 / (6 × 400) ≈ 417 µs.
5. The loop therefore updates about 400 times per commutation sector.

Headroom is not the risk. The risk is whether the 200 MHz clock closes timing on 130 nm, since the FPGA proof runs at 81.25 MHz.

## Open Questions for Diligence
- Which datapath stages are pipelined, and what is the measured end-to-end latency on the Artix-7 build?
- Is ReRAM on the first MPW, or is the SCL eNVM fallback the plan of record?
- What sets the 5–120 V converter's efficiency at the low end of the range?

## Key Takeaways
1. The claim is a two-chip-to-one collapse, with the loop in hardware.
2. Every performance number on the sheet is a target line.
3. Pre-drivers mean external FETs; say so when quoting 120 V.
4. The addressable tile is 3–40x units × ASP; explain it or drop it.
5. First MPW is cycle 1, alongside SKU-2.

## Connects To
- **ch05**: SKU-4 as the FOC safety brain beside SKU-1.
- **ch13**: SKU-1 appears as the "DRV" die in the SiP.
- **deepgrid-sku-portfolio ch08**: commercial-first sequencing and price exposure.
