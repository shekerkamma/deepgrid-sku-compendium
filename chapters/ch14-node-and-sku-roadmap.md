# Chapter 14: Node and SKU Roadmap
*Sheet 14 · FIG 14 · methodology diagram · 130/180 nm now · 90/55/45 nm from 2028 · 28 nm from 2030*

## Core Idea
Three phases:
- 130/180 nm ships the ten-SKU catalogue plus seven more mature-node parts (2026–2027)
- 90/55/45 nm takes three logic-bound parts whose channel count demands it (2028–2029)
- 28 nm and below takes compute (2030+)

The figure prints its honest boundary and corrects its own SKU arithmetic.

## Frameworks Introduced
- **Shrink only when channels demand it.** "Why shrink at all: channel count and DSP throughput, not speed: an EW receiver needs many parallel chains." 130 nm analog IO rings are kept where voltage or ESD demands; only logic shrinks.
- **The honest boundary** (printed): "Everything above is a sub-10 nm problem. 28 nm buys some of it. None of it is claimable on 130 nm, at any clock."
- **What 130 nm cannot do**: multi-TOPS NPU for onboard AI inference; object detection or target recognition at frame rate; central SDV compute (multi-GHz cores, GPU, LPDDR5); UAV swarm coordination.
- **The arithmetic check** (printed): the strategy draft said "10 new SKUs every year" and "over 1,000 active SKUs by Year 5", inconsistent by 20x, because 10 a year for 5 years is 50. 1,000 would need ~200 tapeouts a year. "Pick one number before this goes to an investor." The figure shows the 50-SKU ramp.

## Key Concepts
- **Phase 1 · 2026–2027 · 130 nm / 180 nm**: power paths, high-voltage analog, RF front ends, 32-bit MCUs.
  - Shipping catalogue (SKU 1–10): 1 BLDC Controller, 2 Smart Meter, 3 Power Management, 4 Microcontroller, 5 Transceiver, 6 Voltage Supervisor, 7 Radar, 8 Display Driver, 9 SDV Zonal Gateway, 10 Drone SoC D100.
  - Next at 130/180 nm (SKUs 11, 13, 14, 15, 17, 18, 20): Proximity Fuse Ctrl (130 nm), Naval Engine Health (180 nm BCD), Armour Smart Fuse (130/180 HV), Secure Boot Element (130 nm), Cockpit MFD Ctrl (130 nm HV), Optoelectronic Sight (180 nm MS), Avionics PMIC (130 nm BCD).
- **Phase 2 · 2028–2029 · 90/55/45 nm**: comms processors, multi-channel DSP, EW receivers, SDR.
  - Logic shrink (SKUs 12, 16, 19): Military GNSS Baseband (90/55 nm), SDR Transceiver (SiGe + CMOS), Drone Nav Co-Processor (55/45 nm).
- **Phase 3 · 2030+ · 28 nm and below**: multi-TOPS NPU, AI edge, central SDV compute.
- **Ramp panels (illustrative)**:
  - SKU catalogue: 10, 20, 30, 40, 50 by Year 5
  - node mix: share at 130/180 nm falls from Year 3
  - tapeouts per year: rising bars, labelled "~10/yr"
  - defence share of revenue: rising, "PIL-driven"
- **BCD / HV / MS**: bipolar-CMOS-DMOS power process; high-voltage option; mixed-signal process.

## Mental Models
- Use **"which constraint binds?"** Voltage and analog keep a part mature; parallel channels move it to 90–45 nm; raw compute moves it to 28 nm and below.
- Think of **the SKU numbering as a queue**: numbers 11–20 interleave by node (12, 16 and 19 wait for Phase 2), so the number does not imply order.
- Treat **the arithmetic box as a template**: every sheet benefits from one.

## Anti-patterns
- **Listing D100 (SKU 10) in the 130/180 nm Phase 1 catalogue** while Phase 3 says multi-TOPS NPUs and frame-rate detection cannot be done at 130 nm. Split D100 into its 130 nm control/VIO part and a 28 nm AI die (ch11).
- **A "~10/yr" tapeout label over rising bars**: if the catalogue grows by exactly 10 a year, tapeouts must be at least 10 every year, and more once analog parts need 2–3 silicon cycles each. Label the bars with numbers.
- **Counting shipped SKUs from tapeouts**: a SKU needs characterisation and qualification after silicon.

## Reference Tables

| Phase | Years | Node | Parts | Why this node |
|---|---|---|---|---|
| 1 | 2026–2027 | 130 / 180 nm | SKUs 1–10; next 11, 13, 14, 15, 17, 18, 20 | voltage, analog, RF front end, cost |
| 2 | 2028–2029 | 90 / 55 / 45 nm | SKUs 12, 16, 19 | channel count, DSP throughput |
| 3 | 2030+ | 28 nm and below | NPU, AI edge, central SDV compute | compute; much is sub-10 nm |

| SKU | Name | Node noted |
|---|---|---|
| 11 | Proximity Fuse Ctrl | 130 nm |
| 12 | Military GNSS Baseband | 90/55 nm |
| 13 | Naval Engine Health | 180 nm BCD |
| 14 | Armour Smart Fuse | 130/180 HV |
| 15 | Secure Boot Element | 130 nm |
| 16 | SDR Transceiver | SiGe + CMOS |
| 17 | Cockpit MFD Ctrl | 130 nm HV |
| 18 | Optoelectronic Sight | 180 nm MS |
| 19 | Drone Nav Co-Processor | 55/45 nm |
| 20 | Avionics PMIC | 130 nm BCD |

## Worked Example
**Does the ramp survive the business plan's cycle rules? (reconstruction)**
1. The catalogue target is 50 SKUs by Year 5, i.e. 10 new SKUs a year.
2. The business plan says mostly-digital chips need one silicon cycle plus 6–9 months, and mostly-analog chips need 2–3 cycles plus approval. At 198 days a cycle, there are about 1.8 cycles a year.
3. Assume half the catalogue is analog-heavy at 2.5 attempts per SKU and half digital at 1 attempt. Then 10 SKUs a year need 5 × 2.5 + 5 × 1 = **17.5 tapeout slots a year**, spread over about 1.8 shuttle cycles.
4. That is about 17.5 ÷ 1.8 ≈ 10 designs per shuttle cycle. The seed round funds six sky130 runs over 24 months (about 3.7 cycles), roughly 1.6 designs per cycle, so the ramp needs about six times that rate.
5. Conclusion: the 50-SKU ramp is internally consistent on counting SKUs, but tapeout capacity, not the catalogue count, is the binding constraint. The tapeout panel should show 15–20 a year by the ramp's middle years, or the SKU target should fall.

## Key Takeaways
1. Shrink by channel count and DSP throughput, never for speed alone.
2. Keep the honest boundary printed, and make the D100 entry obey it.
3. The arithmetic box is the compendium's best habit; extend it to tapeouts.
4. SKUs 11–20 are defence-heavy mature-node derivatives; 12, 16 and 19 wait for Phase 2.

## Connects To
- **ch08**: SDR Transceiver (16) extends the SiGe + CMOS split.
- **ch09**: Cockpit MFD Ctrl (17) extends the display driver.
- **ch11**: D100's position in Phase 1.
- **deepgrid-sku-portfolio ch04, ch05**: node roadmap and the 198-day loop.
