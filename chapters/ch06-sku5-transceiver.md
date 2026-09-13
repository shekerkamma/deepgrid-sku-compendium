# Chapter 6: SKU-5 Transceiver (RS-485 + CAN-FD)
*Sheet 6 · FIG 11 · 130 nm · SKY130 / IHP SG13G2*

## Core Idea
A dual-channel line driver: RS-485 (half duplex, 20 Mbps, 256 nodes) and CAN-FD (8 Mbps data, ISO 11898-2), each with thick-oxide 5 V LDMOS output stages, 30 mV hysteresis receivers, failsafe bias and ±15 kV ESD. The sheet's design rule is its best line: the skew budget, not raw data rate, sets maximum node count and cable length.

## Frameworks Introduced
- **Why 130 nm suits a line driver** (printed in both channels): a 5 V-tolerant thick-oxide ring is exactly what a bus driver needs, and advanced nodes are actively worse for this part class.
- **The obsolescence + sourcing-bar wedge** (text panel): replacing a discontinued TI/ADI/Renesas interface part behind the FSC-5962 sourcing bar is the fastest defence conversation, because the socket already exists.
- **Skew budget sets reach.** Driver propagation t_pd(D) is 20 ns and receiver t_pd(R) is 50 ns; the spread between edges, not the headline bitrate, limits how many nodes and how much cable the bus tolerates.
- **Failsafe by default.** Idle = recessive with open, short and idle bus detection, so a dead or disconnected bus reads as a known state.

## Key Concepts
Per channel, with the same block set:
- **Logic IF**: 1.8 V core, level shift to 5 V; TXD, RXD, EN/STB.
- **Pre-driver**: slew shaping, dV/dt 3 V/ns trimmed.
- **Output stage**: differential push-pull, thick-oxide 5 V LDMOS.
- **Failsafe bias**: idle = recessive; open/short/idle detection.
- **Fault**: thermal + short, 150 °C shutdown.
- **Protection logic**: RS-485 has an EN interlock (no bus contention); CAN has a TXD timeout (bus-fault detect).
- **Receiver**: hysteresis comparator, 30 mV hysteresis, 50 ns propagation.
- **ESD + EMC**: "IEC 61000-4-2, ±15 kV HBM contact".
- **Local supply**: 3.3 V LDO + bandgap from the 5 V bus, plus a 1.8 V core rail.
- **OSC + PLL**: 40 MHz reference; slew reference (RS-485) or bit-time recovery (CAN).
- **Bus pins**: A/B and CANH/CANL.
- **Common mode as drawn**: RS-485 ±7 V, CAN-FD ±12 V.
- **Variants (text)**: HV bus-fault variant on SCL/IHP; cycle-2 MPW.

## Mental Models
- Think of **the transceiver as an analog part with a digital skin**: every differentiating block is analog or ESD.
- Use **"attach volume"**: one transceiver per bus node, so volume tracks node count, not system count.
- Treat **a trimmed dV/dt** as the EMC lever: slower edges radiate less and allow longer stubs.

## Anti-patterns
- **Pairing 20 Mbps with 1.2 km** (worked example): the eye-height panel's 1.2 km target cannot hold at the channel's headline rate.
- **"IEC 61000-4-2 ±15 kV HBM contact"**: IEC 61000-4-2 is a system-level contact/air-discharge test; HBM is a component-level model with different pulse energy. State each rating against its own standard.
- **"±7 V CM" for RS-485**: TIA/EIA-485 specifies −7 V to +12 V. Quote the standard's range.
- **The template prototype strip** (Artix-7 → 200 MHz fixed) on a part with no processor core.

## Reference Tables

| Characteristic panel | Axis | Target |
|---|---|---|
| Eye height | 0 → 1,200 m cable | 1.2 km |
| ESD survival | 2 → 15 kV HBM | 15 kV |
| Bus loading | 1 → 256 nodes | 256 |
| Quiescent | active → standby | <8 µA |

| Channel | Mode | Rate | Nodes | CM (as drawn) | Standard |
|---|---|---|---|---|---|
| 1 | RS-485 half duplex | 20 Mbps | 256 | ±7 V | TIA/EIA-485 (−7 to +12 V) |
| 2 | CAN-FD | 8 Mbps data | n/a | ±12 V | ISO 11898-2 |

| Market (rough internal estimate) | Value |
|---|---|
| Volume | 20–60 M/yr India industrial + automotive |
| ASP | $0.4–1.5 |
| Positioning | attach part: ships with every node on the bus |

## Worked Example
**Can 20 Mbps and 1.2 km coexist? (reconstruction from a common rule of thumb)**
1. RS-485 design guidance commonly uses data rate × cable length ≲ 10⁸ bit/s·m for twisted pair without equalisation.
2. At 20 Mbps: 10⁸ / 2×10⁷ = **~5 m**, stretching to tens of metres with good cable and slew control.
3. At 1,200 m: 10⁸ / 1,200 ≈ **~83 kbps**, which is the regime where 1.2 km is normally quoted.
4. So the eye-height panel describes a low-rate mode, and the 20 Mbps header describes short-reach use. Show two operating points (rate, length, node count) instead of one of each.
5. Loop delay check for CAN-FD: t_pd(D) + t_pd(R) = 20 + 50 = 70 ns, comfortably within typical ISO 11898-2 transceiver loop-delay limits of a few hundred nanoseconds.

## Open Questions for Diligence
- Which ESD structures on sky130 have silicon data at ±15 kV? This is the free-PDK analog-library gap in its sharpest form.
- Is 256 nodes achieved with 1/8 unit-load receivers, and at what input impedance?
- Which discontinued parts is the first pin-compatible target?

## Key Takeaways
1. The part wins on physics (thick oxide) and procurement (obsolescence, FSC-5962).
2. Quote rate and reach as paired operating points.
3. Name each ESD rating against its own standard.
4. ±15 kV on sky130 must be proven on a dedicated test chip; it cannot enter at the shared-tile level.

## Connects To
- **ch13**: "XCVR" die in the SiP.
- **ch12**: "SKU-5 connects" in the platform.
- **deepgrid-sku-portfolio ch06**: analog and HV blocks enter at validation level 2.
