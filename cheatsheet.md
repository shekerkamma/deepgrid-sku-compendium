# Cheatsheet: reading and defending the compendium

## Decision rules
- When quoting a characteristic panel, **say "target"**: every panel is illustrative.
- When quoting a market tile, **carry its footnote grade**; only the ~250 M meter rollout and the PIL-5 display item are externally sourced.
- When quoting an addressable figure, **show units × ASP beside it**, or expect a 3–180x gap question.
- When a sheet shows the Artix-7 → 200 MHz strip on an analog part, **replace it** with the sheet's real node path.
- When D100 comes up, **split it**: 130 nm FC + VIO + failsafe island; 28 nm AI die, Track B.
- When quoting radar resolution, **name the chirp profile**: 3.75 cm holds to ~15–19 m, not 150 m.
- When quoting RS-485 reach, **pair it with a rate**: 1.2 km means ~100 kbps.
- When quoting ESD, **name the standard**: IEC 61000-4-2 and HBM are different tests.
- When quoting supervisor accuracy, **quote per rail**: ±1% on 5V0/3V3, ±2% on 1V8/1V2 as drawn.
- When quoting PMIC efficiency, **say "pre-regulator"**: rail-tree efficiency at full load is ~59% by reconstruction.
- When presenting the SDV platform, **state its open-source lineage**.

## Per-SKU one-liners
| SKU | Replaces | Physics reason | Headline targets | Path |
|---|---|---|---|---|
| 1 BLDC | DRV83xx + MCU | 5–120 V B/B on die | PID <1 µs, PWM ×7 to 200 kHz, 95% eff | cycle 1 |
| 2 Meter | ADE9153/V9203 + MCU | 24-bit ΣΔ matching | 0.5S, THD-15, <2 µW AON | cycle 1 |
| 3 PMIC | TI/ADI QML power | 28 V LDMOS, poly/MiM | <20 mV reg, 60 dB PSRR, 94% buck | sky130 20 V → SCL 180 |
| 4 MCU | Microchip/Renesas FuSa | duplicate cores cheaply | FAULTn ≤2 cyc, 99% DC | 130 nm |
| 5 XCVR | TI/ADI/Renesas interface | 5 V thick-oxide ring | 20 Mbps/256 nodes, ±15 kV, <8 µA | cycle 2 |
| 6 SUP | TI/Maxim supervisors | 0.1% poly, curved bandgap | <8 µs, <12 µA, ±1% | sky130 → SCL, first MIL-883 |
| 7 Radar | imported 77 GHz FE | SiGe HBT 350 GHz | 3.75 cm, 15°, NF 3.5 dB | IHP MPW FY28 |
| 8 Display | Novatek/Himax TCON + SD | 0–12 V amps, 24 V shift | SXGA60, <1 LSB, −40% power | cycle 3 |
| 9 Zonal | imported zonal/gateway | channel count | <50 µs GW, <10 ms fault, <2 s OTA | pilot letter |
| D100 | none indigenous | failsafe island | 30 Hz pose, ~10 TOPS (conflict) | Track B |

## Evidence tiers
| Band | Treat as |
|---|---|
| Block annotation | design target |
| Timing diagram | behavioural sketch |
| Characteristic panel | illustrative shape plus target line |
| Market tile | internal estimate unless a source is named |
| Prototype strip | template text |

## Arithmetic checks worth running on any sheet
| Check | Formula |
|---|---|
| Range resolution | c / 2B |
| FMCW max range | f_IF,max · c / (2 · slope) |
| RS-485 reach | rate × length ≲ 10⁸ bit/s·m |
| LDO heat | (Vin − Vout) × I |
| Always-on current | P / V |
| Deglitch counts | time × clock |
| Package yield | p_die ^ n_dies |
| Market | units × ASP vs addressable |

## Red flags before external use
- D100 NPU at 130 nm contradicts the roadmap's honest boundary.
- Radar 150 m panel against a 1024-point FFT at 3.75 cm.
- 20 Mbps with 1.2 km; ±7 V CM for RS-485.
- 1V8 3 A LDO from 5 V (~9.6 W).
- "Rad-tolerant" on SKU-3.
- ADCs drawn inside the SiGe outline.
- Clipped die-to-die label; unconnected AFE and 28 nm dies in the SiP.
- "Track B" meaning two different things.
- "~10/yr" tapeouts over rising bars; a 50-SKU ramp needs ~17.5 slots a year.
