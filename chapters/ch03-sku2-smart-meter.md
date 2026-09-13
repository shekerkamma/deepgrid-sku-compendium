# Chapter 3: SKU-2 Smart Meter SoC
*Sheet 3 · FIG 9 · Class 0.5S metrology*

## Core Idea
A metrology AFE, decimation, a power-quality engine, a RISC-V core, a crypto engine and an always-on RTC share one die. It replaces the ADE9153/V9203-class AFE plus a separate meter MCU. Tamper detection, logged against a tamper-backed RTC, is what tenders require.

## Frameworks Introduced
- **Tamper is a silicon requirement, not a feature.** Case and magnet tamper inputs, a tamper log in the RTC, and a domain that stays on below brownout. Software cannot add evidence the hardware failed to keep powered.
  - Why it works: tender language makes tamper evidence mandatory, so a die that keeps it at microwatts disqualifies a two-chip design that does not.
- **Three power islands.** Metrology (green), compute + security (blue) and always-on (amber) are separate groups; the always-on PMU + RTC runs from a 32.768 kHz crystal with a 2.0 V brownout at <2 µW.
- **Multilayer AHB-Lite, 4 masters / 9 slaves.** Metrology, CPU, DMA and the security engine can all master the 32-bit bus at 200 MHz, so samples move without CPU copies.
- **Flash-less by necessity.** The free PDK has no embedded flash, so memory is ROM + ECC SRAM + encrypted QSPI, with an SCL eNVM variant for sealed meters (text panel).

## Key Concepts
- **DG-AFE6**: six-channel 24-bit delta-sigma front end (V/I sense on 6 channels).
- **DECIM**: sinc³ decimation at OSR 256; 24-bit output.
- **METRO**: P, Q, S (active, reactive, apparent power) and THD to the 15th harmonic.
- **DGridRiscV**: RV32IM_Zicsr.
- **DMA**: 4-channel direct memory access.
- **DG-SE**: security engine, AES-256 and SHA-256.
- **24→32**: sample-width extension marker where 24-bit metrology data meets the 32-bit bus.
- **Peripherals**: RTC (tamper log), TIMER (capture/compare), LCD (4×40 segment), UART (DLMS/COSEM), TAMPER (case · magnet), GPIO (relay + LED).
- **DLMS/COSEM**: the utility metering data model and protocol used by Indian smart-meter specifications.

## Mental Models
- Think of **the always-on island as the meter's black box**: it must outlive the mains.
- Use **"rollout scale is public, unit split is ours"** as the evidence grade: ~250 M meters is public, 10–30 M/yr at peak is Deepgrid's split.
- Treat **Class 0.5S** as the accuracy class to hold from 0.05 to 10 × I_b (the accuracy panel's x-axis).

## Anti-patterns
- **Presenting accuracy, THD rejection or <2 µW as achieved**: all three are illustrative panels with target lines.
- **Starting masks before a signed meter-maker letter**: the business plan's stop rule S1 applies to this chip.
- **Assuming a sealed meter can use QSPI flash**: that is why the SCL eNVM variant exists.

## Reference Tables

| Characteristic panel | Axis | Target |
|---|---|---|
| Accuracy vs load | 0.05 → 10 I_b | 0.5S |
| THD rejection | harmonic 3 → 15 | bars fall below target line |
| Always-on power | run → RTC-only | <2 µW |

| Market | Value | Grade |
|---|---|---|
| Rollout | ~250 M smart meters targeted | public |
| ASP | $2–5 metrology SoC | internal |
| Run-rate | 10–30 M/yr at peak | internal split |
| Use cases | single- and three-phase residential; prepaid/smart-prepaid for DISCOMs; industrial submetering and feeder monitoring; tamper as a hard requirement | |
| Path | FPGA-validated blocks → 130 nm; cycle-1 MPW alongside SKU-1 | text panel |

## Worked Example
**Two numbers the sheet implies but does not state (reconstruction)**
1. **Always-on current.** <2 µW at the 2.0 V brownout threshold = 2 µW / 2.0 V = **<1 µA** for the PMU, RTC and tamper log together. That is the budget a battery-backed RTC must meet after mains loss.
2. **Minimum modulator rate for THD-15.** On a 50 Hz grid, the 15th harmonic is 750 Hz. The decimated output must sample above 2 × 750 = 1.5 kHz, and at OSR 256 the modulator must run at ≥1.5 kHz × 256 ≈ **384 kHz**. The sheet gives neither rate; the chosen modulator clock is a design fact worth adding.

## Open Questions for Diligence
- Which modulator clock and output data rate were chosen, and is Class 0.5S shown across the full 0.05–10 I_b range on FPGA plus an external AFE?
- Does the DG-AFE6 sit on the first MPW, or is the analog front end a later cycle?
- How are AES keys provisioned in a flash-less device?

## Key Takeaways
1. The differentiator is tamper evidence kept alive at microwatts.
2. Flash-less memory is forced by the PDK; SCL eNVM covers sealed meters.
3. Only the ~250 M rollout is public; everything else is internal.
4. This is the largest revenue line in the plan, and the most exposed to a missing customer letter.

## Connects To
- **ch02**: shares cycle-1 MPW with SKU-1.
- **ch13**: the "AFE" die in the SiP appears to be this front end.
- **deepgrid-sku-portfolio ch13**: stop rule S1 and the meter row in the crash case.
