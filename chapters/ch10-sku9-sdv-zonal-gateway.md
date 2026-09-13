# Chapter 10: SKU-9 Software Defined Vehicle Zonal Controller + Gateway
*Sheet 10 · FIG 4 · zonal controller + gateway · 130 nm · SKY130 / IHP SG13G2*

## Core Idea
A zonal controller combines:
- an ASIL-D lockstep safety island
- an EVITA-full HSM with secure boot and OTA
- a TSN switch with PTP
- 23 in-vehicle network ports
- 16 smart fuses that replace the relay box

A scope box on the figure itself limits the claim: 130 nm at 200 MHz owns the zonal layer, and central SDV compute is not claimed.

## Frameworks Introduced
- **Scope honesty on the figure.** The "SCOPE" banner reads: "130 nm at 200 MHz owns the ZONAL layer: gateway, smart I/O, safety and security edge. Central SDV compute (multi-GHz cores, GPU, NPU, LPDDR5) is a sub-10 nm part and is NOT claimed here." Keep this wording in every derivative.
- **Channels and bounded latency size the part, not MHz.** The gateway routes and rate-limits between domains, and TSN gives bounded latency (the figure's note).
- **Freedom from interference** (ISO 26262 term): an MPU plus a bus firewall keep non-safety software from corrupting the safety island.
- **Replace the relay-and-harness box.** Smart fuses (eFuse + sense) with I²t trip curves replace mechanical fuses and relays. Weight, wiring and diagnostics make the OEM's case for you.
- **Where it fits** (text panel): the bridge between the CV ADAS heritage (AD2) and the DG SDV platform; the mature-node edge of the same architecture.

## Key Concepts
- **Safety island · ASIL-D**:
  - DGridRiscV ×2: RV32IM_Zicsr, lockstep, 200 MHz, +2 cycle skew, with PMP and ECC
  - comparator: bus + retire, mismatch → safe state
  - freedom from interference: MPU + bus firewall
- **Security**:
  - HSM: EVITA-full class, AES-256 and ECC-256
  - secure boot: root of trust, OTP key store, A/B slots
  - OTA engine: signed image verification + rollback
- **Real-time control**:
  - TSN switch: 802.1Qbv time-aware shaper
  - PTP: 802.1AS clock, sub-µs sync
  - service router: SOME/IP + DoIP
- **Interconnect**: AXI4 crossbar, 64-bit, 200 MHz (130 nm ASIC).
- **In-vehicle network**: 100BASE-T1 ×2, 1000BASE-T1 ×1, CAN-FD ×8, CAN-XL ×2, LIN ×8, FlexRay ×2.
- **Zonal power + I/O**: smart fuse ×16 (eFuse + sense, replaces relay box); HS driver ×8 (load control); ADC 12-bit, 24-channel sense; PWM ×12 (actuator drive).
- **Gateway timing (sketch)**: a CAN frame enters, waits for its TSN window and leaves on Ethernet egress inside a bounded-latency window; a safety tick runs throughout; HSM verification runs alongside; the OTA image is checked.

## Mental Models
- Think of **the zone as a building's distribution board**: fuses, switches and a network patch panel, with a security guard at the door.
- Use **"bounded, not fast"**: TSN's value is a latency ceiling you can prove, not a low average.
- Treat **the same lockstep pair as SKU-4's**: the verification is shared.

## Anti-patterns
- **Letting "central gateway" in the use cases drift into "central compute"**: routing CAN/LIN/FlexRay to Ethernet is gateway work; the scope box forbids compute claims.
- **Claiming EVITA-full or ASIL-D compliance**: both are target classes.
- **Quoting <50 µs gateway latency without traffic conditions**: the panel compares best effort with TSN, not a stated load.

## Reference Tables

| Characteristic panel | Axis | Target |
|---|---|---|
| Gateway latency | best-effort → TSN | <50 µs |
| Fault reaction | detect → safe state | <10 ms |
| Smart-fuse trip | 1x → 10x overload | I²t curve |
| OTA verify | image size 1 → 8 MB | <2 s |

| Market (volumes: rough internal estimate) | Value |
|---|---|
| Volume | 4–12 M/yr India zonal/gateway controller sockets |
| ASP | $6–20 |
| Addressable | ~$0.3–0.7 B, zonal layer only (1.25–29x units × ASP) |
| Use cases | zonal controllers replacing relay-and-harness; central gateway CAN/LIN/FlexRay to automotive Ethernet; secure OTA root of trust; EV zonal power distribution with electronic fusing |
| Status (business plan) | waits on a vehicle maker's pilot letter |

## Worked Example
**Why channels, not MHz, size this die (reconstruction by tallying the figure)**

| Function | Count |
|---|---|
| Automotive Ethernet (100BASE-T1 ×2, 1000BASE-T1 ×1) | 3 PHY/MAC ports |
| CAN-FD ×8, CAN-XL ×2 | 10 |
| LIN ×8 | 8 |
| FlexRay ×2 | 2 |
| **Network ports** | **23** |
| Smart fuses | 16 |
| High-side drivers | 8 |
| ADC sense channels | 24 |
| PWM outputs | 12 |

1. Each network port needs its own controller logic and pins; each smart fuse needs a current-sense path and a power device, likely external for zonal currents.
2. Controllers and pins, not processor speed, set die area and package size. A 200 MHz core idles most of the time, while 23 ports and 60 I/O functions cannot.
3. A 1000BASE-T1 port at 1 Gbps serialises a maximum 1,522-byte frame in about 12 µs, so a <50 µs gateway latency leaves room for one TSN window wait. That is plausible, but it depends on the Qbv schedule.
4. The same logic is why the roadmap shrinks parts by channel count (ch14).

## Open Questions for Diligence
- Is the 1000BASE-T1 PHY on die at 130 nm, or an external PHY with only the MAC on die?
- Are the smart-fuse power FETs external, and what currents are targeted?
- What package and pin count carry 23 ports plus 60 I/O functions?
- Which vehicle maker is in pilot discussion?

## Key Takeaways
1. The scope box is the sheet's most valuable sentence; keep it on every version.
2. The part is sized by ports and I/O, which suits a mature node.
3. Lockstep and HSM claims are target classes, not certifications.
4. Clarify on-die versus external PHYs and fuse FETs before quoting die cost.

## Connects To
- **ch05**: lockstep pair and comparator.
- **ch12**: SKU-9 as the zonal edge of the DG SDV platform.
- **ch14**: channel count as the reason to shrink.
