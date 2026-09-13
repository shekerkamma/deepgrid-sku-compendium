# Chapter 4: SKU-3 Power Management (Hi-Rel PMIC)
*Sheet 4 · FIG 10 · 28 V avionics rail tree · 130 nm · header says SKY130 / IHP SG13G2*

## Core Idea
A 28 V aircraft-bus PMIC does everything on one die:
- input conditioning to DO-160 / MIL-STD-461
- a peak-current-mode synchronous buck to 5 V
- four sequenced rails (3V3, 1V8, 1V2, 0V9) with window monitors and foldback current limits
- a Brokaw reference with OTP trim
- an SEU-hardened sequencer

It is the compendium's proof of real analog design, and its rail tree has a thermal problem worth fixing on paper first.

## Frameworks Introduced
- **Four-stage power architecture**: input conditioning → pre-regulator → rail generation → supervision. Each stage has its own protection, so a fault is contained where it starts.
- **Peak-current-mode inner loop.** A type-III compensated error amp (60 dB DC) and a PWM comparator with slope compensation at 500 kHz feed the gate driver (adaptive dead-time, 20 ns non-overlap). Current sense (sense-FET + amp) closes a cycle-by-cycle limit.
  - Why it works: cycle-by-cycle current limiting protects the LDMOS half-bridge on every switching period, which is what a 28 V bus with 40 V transients needs.
- **Programmable sequencing with a reset gap.** Rails come up in programmable order in 1 ms steps (8 slots). PGOOD asserts when all rails are in regulation, and RESETn releases 1 ms later.
- **SEU hardening by design only**: DICE latches plus TMR on the sequencer FSM. The text panel adds "no rad-hard claim beyond that".

## Key Concepts
- **Input conditioning · 5 V-tolerant IO ring**:
  - EMI + TVS: CM choke + pi filter, clamp 36 V
  - ideal diode: reverse polarity, back-to-back LDMOS
  - inrush: soft-start FET, dI/dt limited
  - UVLO + OVLO: 18 V / 34 V, hysteretic
- **Power stage**: LDMOS half-bridge, 5V0 at 8 A, Rds(on) 22 mΩ.
- **Reference + bias**:
  - bandgap: Brokaw cell, 1.20 V, 12 ppm/°C
  - trim: OTP 6-bit, one-time, post-package
  - bias: PTAT + CTAT
  - oscillator: RC 8 MHz, ±2% trimmed
  - the figure's note: "poly resistors + MiM caps: 130 nm is a good analog node"
- **Rail generation (all from 5V0)**:
  - LDO 3V3 2.0 A (PMOS pass, 60 dB PSRR)
  - LDO 1V8 3.0 A (PMOS pass, core supply)
  - buck 1V2 5.0 A (sync, 1 MHz)
  - buck 0V9 6.0 A (sync, 1 MHz)
  - each rail has a window monitor (OV/UV, ±3%) and an OC limit with foldback; soft-start ramp; thermal shutdown at 150 °C junction
- **Supervision · sequencing · telemetry**:
  - sequencer (FSM + delay counter)
  - telemetry: 12-bit SAR with an 8:1 mux, V/I per rail over SPI
  - windowed watchdog, 1 ms / 10 ms
  - SEU harden block
  - outputs: SPI, PGOOD, RESETn

## Mental Models
- Think of **LDOs as heaters with a regulated output**: dissipation = (Vin − Vout) × I.
- Use **"contain the fault at its stage"**: a clamp, UVLO/OVLO, cycle-by-cycle limit and per-rail foldback are four separate fences.
- Treat **one-time post-package trim** as a production-test step to cost, not just a spec.

## Anti-patterns
- **"Rad-tolerant PMIC ASP band"** (market tile) against "no rad-hard claim" (text panel): use one wording. The honest one is "SEU-hardened by design".
- **Quoting 94% efficiency for the whole PMIC**: the panel describes the pre-regulator over 10–100% load; LDO rails dissipate far more (worked example).
- **Stamping "Artix-7 · 81.25 MHz → 200 MHz fixed" on this sheet**: nothing here is prototyped on an FPGA at a processor clock, and the text panel names sky130 20 V devices → SCL 180 nm production with a MIL-883 flow.
- **Header "SKY130 / IHP SG13G2" versus text panel "SCL 180 nm production"**: state the production node once.

## Reference Tables

| Characteristic panel | Condition | Target |
|---|---|---|
| Load regulation | 0 → I_max | <20 mV |
| Line transient | 28 → 40 V step | recovers <50 µs |
| PSRR | 100 Hz → 1 MHz | 60 dB @ 100 Hz |
| Efficiency | 10 → 100% load | 94% |

| Market (rough internal estimate) | Value |
|---|---|
| Volume | 10–50 k/yr India hi-rel avionics + defence power |
| ASP | $50–200 |
| Addressable | ~$40–90 M defence + space |
| Hooks (text) | TI/ADI QML sockets in every avionics and vetronics LRU; SRIJAN NSG-5962 class; AVL-locked for decades; 60–75% GM |

## Worked Example
**Rail-tree power at full rated current (reconstruction; assumes all four rails at rated current together)**

| Rail | Source | Output power | Dissipated in regulator | Current drawn from 5V0 |
|---|---|---|---|---|
| 3V3 LDO, 2.0 A | 5 V | 6.6 W | (5 − 3.3) × 2.0 = **3.4 W** | 2.0 A |
| 1V8 LDO, 3.0 A | 5 V | 5.4 W | (5 − 1.8) × 3.0 = **9.6 W** | 3.0 A |
| 1V2 buck, 5.0 A | 5 V | 6.0 W | ~0.4 W at 94% | ~1.3 A |
| 0V9 buck, 6.0 A | 5 V | 5.4 W | ~0.3 W at 94% | ~1.2 A |
| **Total** | | **23.4 W** | **~13.7 W** | **~7.5 A** |

1. The 5V0 demand of ~7.5 A sits just under the 8 A power-stage rating: consistent, with little margin.
2. The pre-regulator must deliver ~37.5 W; at 94% it draws ~40 W from the bus, about 1.4 A at 28 V.
3. The LDOs dissipate ~13 W on die against 23.4 W delivered, so rail-tree efficiency is about 23.4 / 40 ≈ 59% at full load, not 94%.
4. Fixes to consider: move 1V8 to a buck, or feed the LDOs from an intermediate rail closer to their outputs. Either way, state the package thermal resistance behind the 150 °C shutdown.

## Open Questions for Diligence
- What package and θJA carry 13 W of LDO dissipation, or is full simultaneous load not a real use case?
- Which analog blocks have a silicon-proven sky130 or SCL example to start from? The free-PDK analog library gap applies here first.
- What does the MIL-883 flow cost per unit at 10–50 k/yr?

## Key Takeaways
1. This sheet is the evidence of genuine analog design; defend it on circuit detail.
2. Recompute efficiency at the rail tree, not the buck.
3. Remove "rad-tolerant" unless there is dose and latch-up data.
4. Give the sheet its real node path: sky130 20 V devices → SCL 180 nm, with a MIL-883 flow.
5. PGOOD then RESETn +1 ms is the sequencing contract other SKUs rely on.

## Connects To
- **ch07**: SKU-6 reuses the bandgap/trim/window-monitor ideas at lower power.
- **ch13**: "power tree from SKU-3" in the SiP.
- **deepgrid-sku-portfolio ch04**: why 130 nm, and the analog-library risk.
