---
name: deepgrid-sku-compendium
description: "Sheet-by-sheet knowledge base from Deepgrid Semi's SKU Architecture Compendium v3 (technical annex for investor and DPSU diligence, 2026). Use when reading or defending a specific architecture sheet: block diagrams, timing, characteristic targets and market panels for SKU-1 to SKU-9, the D100 drone SoC, the DG SDV platform, the SiP composition and the node roadmap, or when checking the sheets' numbers against each other."
---

<!-- argument-hint: [SKU number, block name, sheet, or "contradictions"] -->

# Deepgrid SKU Architecture Compendium v3
**Source**: Deepgrid Semi, SKU Architecture Compendium v3 (14 landscape sheets, 2026, marked Confidential) | **Chapters**: 14 | **Generated**: 2026-09-13

**Confidential internal material.** Keep this skill private. For the business plan behind these sheets (cost model, 198-day loop, go-to-market, the round), use the sibling skill `deepgrid-sku-portfolio`.

Most of the content lives in the raster figures, not the text layer. Every chapter here was written from the figures at native resolution. Arithmetic marked "reconstruction" is derived from the sheet's own numbers, not stated on it.

## How to Use This Skill

- **Without arguments**: load the reading rules and the contradiction list below
- **With a SKU or block**: ask about `SKU-7`, `failsafe island`, `sequencer`; I read that sheet's chapter first
- **With a chapter**: ask for `ch04`
- **Checking a claim**: ask "does the compendium support X?"; I check the sheet, then the contradiction list

---

## Contradictions to fix before a diligence read

Resolve these before the compendium goes to a semiconductor-literate reader. Each one is checkable from the sheets alone.

1. **The D100 sheet shows what the roadmap says 130 nm cannot do.** D100 (FIG 3) draws a ~10 TOPS NPU running YOLO-family detection, LPDDR4 and a 128-bit AXI4 crossbar, stamped "130 nm ASIC · 200 MHz fixed". The roadmap (FIG 14) lists "multi-TOPS NPU" and "object detection at frame rate" under "What 130 nm cannot do". The SiP sheet puts the D100 core on a 28 nm die in Wave 2. The whitepaper puts D100 on TSMC 28 nm. → ch11, ch14
2. **Radar range and resolution come from different chirp profiles.** 4 GHz over a 40 µs sweep with a 10 MHz anti-alias filter and a 1024-point FFT reaches roughly 15–19 m. The range-profile panel runs to 150 m, which needs about 4,000 bins at 3.75 cm. → ch08
3. **RS-485 at 20 Mbps is paired with 1.2 km of cable.** The usual rule of thumb (data rate × length ≈ 10⁸ bit/s·m) allows about 1.2 km only near 100 kbps. The sheet's ±7 V common mode also understates TIA-485's −7 V to +12 V. → ch06
4. **The PMIC's 1V8 3 A LDO fed from 5 V burns about 9.6 W on die at full load.** The 3V3 LDO adds 3.4 W. The 94% efficiency panel describes the buck, not the rail tree. → ch04
5. **The supervisor claims ±1%, but draws ±2% comparators on its 1V8 and 1V2 rails.** → ch07
6. **The PMIC's market panel says "rad-tolerant PMIC ASP band".** The text panel says there is no rad-hard claim beyond SEU hardening by design. → ch04
7. **The prototype-to-product strip is template boilerplate on every sheet.** "Artix-7 81.25 MHz → 130 nm ASIC · SKY130 / IHP SG13G2 · 200 MHz fixed" also appears on analog-only parts (PMIC, transceiver, supervisor) whose text says SCL 180 nm production. → ch01
8. **"IEC 61000-4-2 ±15 kV HBM contact" mixes two ESD test models.** IEC 61000-4-2 is system-level contact/air discharge; HBM is a component-level model. → ch06
9. **The radar figure draws the ADCs inside the SiGe die outline.** The sheet text puts them on the CMOS die. → ch08
10. **The DG SDV platform's block vocabulary matches the open-source PULP Carfield SoC** (CVA6-class RV64GCH host, TL-UL/OTBN/KMAC secure domain, OBI/Regbus safe domain, AXI-REALM, HMR, TCDM clusters). The sheet states no lineage or licence. → ch12
11. **"Track B" names the SDV platform page in the compendium, but the D100 programme in the whitepaper.** → ch11, ch12
12. **Addressable-market figures sit above units × ASP on every sheet that gives both**, by up to 25–180x. No sheet says what else the figure includes. → ch01
13. **Every product panel says 200 MHz.** The whitepaper calls 200 MHz an unmeasured processor target. The DG32-LITE documents report a 50 MHz die clock. → ch01

## Reading rules

**1. Grade every number by the band it sits in.** A sheet has five evidence tiers:
- block annotations are design targets
- timing diagrams are behavioural sketches
- characteristic panels are labelled "illustrative" and carry a dashed target line, never measured data
- market panels are "ROUGH INTERNAL ESTIMATE" unless a public source is named
- the prototype strip is boilerplate

Nothing on any sheet is silicon measurement. → ch01

**2. One die replaces a two-chip set.** This is the product thesis:
- SKU-1 replaces a DRV83xx-class driver plus an MCU
- SKU-2 replaces an ADE9153/V9203-class AFE plus an MCU
- SKU-8 replaces a Novatek/Himax TCON plus source drivers

Lead a sheet with the socket it removes. → ch02, ch03, ch09

**3. Mature node for physics, stated per sheet:**
- SKU-1: 5–120 V buck/boost beside the core
- SKU-3: 28 V aircraft bus, LDMOS half-bridge, poly resistors and MiM capacitors
- SKU-5: 5 V-tolerant thick-oxide LDMOS ring
- SKU-6: 0.1%-matched poly ladders and a curvature-corrected bandgap
- SKU-8: 0–12 V column amplifiers and a 24 V gate level shift

The one hard limit is 77 GHz, which needs IHP SiGe HBTs. → ch04–ch09

**4. Partition at the ADC.** The radar puts chirp synthesis, PAs and four LNA/mixer/IF chains on a SiGe die, and processing from sampling onward on 130 nm CMOS. The SiGe half "has no FPGA equivalent and is proven on silicon or not at all". → ch08

**5. One lockstep block, reused up the stack:**
- SKU-4: two DGridRiscV cores with a 2-cycle skew and a bus-plus-retire comparator, FAULTn within 2 cycles
- SKU-9: the same pair, where a mismatch drives a safe state
- D100: a dual FC/NAV core
- SKU-3: DICE latches plus TMR on the sequencer

→ ch05, ch10, ch11

**6. The failsafe island is the D100 wedge.** A link monitor (RC loss, GPS loss, IMU fault) feeds a safe-state FSM for return-to-home or landing. That drives an independent path direct to the ESC, on isolated power and clock, bypassing flight control, VIO and AI. It is hardware, not firmware, and on the DGCA type-certification path. → ch11

**7. Scope honesty is printed on the figure, and should stay there:**
- SKU-9: "owns the ZONAL layer … central SDV compute is NOT claimed"
- Radar: "proven on silicon or not at all"
- Roadmap: "None of it is claimable on 130 nm, at any clock"

→ ch08, ch10, ch14

**8. The simplest die goes first through screening.** SKU-6 is a single sense chain per rail, and the first part through MIL-883. It also fills the highest socket count. → ch07

**9. Channel count, not MHz, sizes the part and drives the shrink.**
- SKU-9 carries 23 bus ports, 16 smart fuses, 8 high-side drivers, 24 ADC channels and 12 PWM outputs.
- Phase 2 shrinks GNSS baseband, SDR and drone navigation because an EW receiver "needs many parallel chains".

→ ch10, ch14

**10. One architecture, many dies.** In the SDV platform:
- SKU-9 is the zonal edge
- D100 is the airborne sibling
- SKU-3 powers, SKU-6 supervises and SKU-5 connects

The SiP puts six mature dies on a 4-layer organic BGA 15×15 mm substrate, with die-to-die SPI/UART/GPIO (deliberately not UCIe) and a 28 nm compute die in Wave 2. → ch12, ch13

**11. Arithmetic-check your own figure before an investor does.** The roadmap corrects "10 SKUs a year" plus "1,000 SKUs by Year 5" to a 50-SKU ramp. Apply the same check to every sheet (see the contradiction list). → ch14

---

## Chapter Index

| # | Sheet | Key content |
|---|-------|-------------|
| [ch01](chapters/ch01-how-to-read-a-sheet.md) | Cover and sheet anatomy | Five evidence tiers, template strip, market arithmetic |
| [ch02](chapters/ch02-sku1-bldc-controller.md) | SKU-1 BLDC Controller (FIG 5) | FOC datapath, <1 µs PID, 5–120 V B/B, star/delta |
| [ch03](chapters/ch03-sku2-smart-meter.md) | SKU-2 Smart Meter (FIG 9) | DG-AFE6, sinc³ OSR-256, THD-15, tamper, <2 µW |
| [ch04](chapters/ch04-sku3-hi-rel-pmic.md) | SKU-3 Power Management (FIG 10) | 28 V front end, peak-current buck, 4 rails, SEU FSM |
| [ch05](chapters/ch05-sku4-lockstep-mcu.md) | SKU-4 Microcontroller (FIG 6) | 2-cycle skew lockstep, ECC AHB, 99% DC target |
| [ch06](chapters/ch06-sku5-transceiver.md) | SKU-5 Transceiver (FIG 11) | RS-485 + CAN-FD, thick-oxide LDMOS, skew budget |
| [ch07](chapters/ch07-sku6-voltage-supervisor.md) | SKU-6 Voltage Supervisor (FIG 12) | Chopper comparators, 8 µs deglitch, window WDT |
| [ch08](chapters/ch08-sku7-77ghz-radar.md) | SKU-7 Radar (FIG 7) | SiGe/CMOS split, FMCW frame, CFAR, range check |
| [ch09](chapters/ch09-sku8-display-driver.md) | SKU-8 Display Driver (FIG 8) | SXGA pipeline, 3,840 HV outputs, PIL-5 anchor |
| [ch10](chapters/ch10-sku9-sdv-zonal-gateway.md) | SKU-9 SDV Zonal Controller (FIG 4) | Safety island, EVITA HSM, TSN gateway, smart fuses |
| [ch11](chapters/ch11-d100-drone-soc.md) | D100 Drone SoC (FIG 3) | FC + VIO + NPU, failsafe island, node conflict |
| [ch12](chapters/ch12-dg-sdv-platform.md) | DG SDV Platform (Track B) | Five domains, AXI-REALM, lineage question |
| [ch13](chapters/ch13-sip-integration.md) | SiP Integration | Six dies + 28 nm Wave 2, not UCIe, open questions |
| [ch14](chapters/ch14-node-and-sku-roadmap.md) | Node and SKU Roadmap (FIG 14) | Three phases, SKUs 11–20, honest boundary, 50-SKU ramp |

## Topic Index

- **200 MHz on every sheet** → ch01, ch11
- **Addressable vs units × ASP** → ch01, ch02, ch05, ch08, ch10
- **AXI-REALM, HMR, TCDM, SPM** → ch12
- **Bandgap (Brokaw 12 ppm/°C; curvature-corrected 10 ppm/°C)** → ch04, ch07
- **CFAR, range/Doppler FFT, beamforming** → ch08
- **DICE / TMR / SEU hardening** → ch04
- **Die boundary at the ADC** → ch08
- **DLMS/COSEM, tamper, RTC** → ch03
- **ESD (±15 kV, IEC 61000-4-2 vs HBM)** → ch06
- **EVITA-full HSM, secure boot, OTA** → ch10
- **Failsafe island** → ch11
- **FOC, Park/Clarke, CORDIC, PID** → ch02
- **Freedom from interference** → ch10
- **LDO dissipation** → ch04
- **Lockstep comparator, FAULTn** → ch05, ch10, ch11
- **Market panel wording** → ch01
- **PIL-5 BEL 17-inch display** → ch09
- **Phase 2 logic shrink (SKUs 12, 16, 19)** → ch14
- **Prototype → product strip** → ch01
- **Qualification pathfinder** → ch07
- **RS-485 rate × length, common mode** → ch06
- **Sequencer, PGOOD, RESETn** → ch04, ch07
- **Smart fuse, I²t** → ch10
- **SXGA timing, 108 MHz pixel clock** → ch09
- **TSN 802.1Qbv, PTP 802.1AS** → ch10
- **UCIe (deliberately not)** → ch13
- **VIO, EKF, stereo 720p60** → ch11
- **What 130 nm cannot do** → ch14
- **Window watchdog** → ch04, ch07

## Supporting Files

- [glossary.md](glossary.md): block and standard names used on the sheets
- [patterns.md](patterns.md): reusable architecture and diligence patterns
- [cheatsheet.md](cheatsheet.md): per-SKU targets, evidence tiers and red flags

---

## Scope & Limits

This skill covers the 14 compendium sheets only. The sheets are design intent: no number on them is measured silicon, and every characteristic panel is illustrative. The cover and footers carry company registration details, which are deliberately omitted. Arithmetic checks are reconstructions from the sheets' own figures, and are labelled as such in each chapter.
