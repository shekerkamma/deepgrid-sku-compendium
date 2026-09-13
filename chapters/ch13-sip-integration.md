# Chapter 13: SiP Integration
*Sheet 13 · "How the Dies Compose"*

## Core Idea
Six wire-bonded mature-node dies and, in Wave 2, one flip-chip 28 nm AI-compute die share a 4-layer organic substrate in a 15×15 mm BGA. They form an SDV domain controller or D100 UAV compute module: "mature-node packaging at mature-node cost".

## Frameworks Introduced
- **Compose, don't integrate.** Separate dies on the process each needs, joined over the substrate, instead of one large mixed-signal die.
  - Why it works: each die keeps its own node (HV, analog, logic), and packaging stays at organic-substrate and wire-bond cost.
  - Failure mode: one bad die scraps the package, so every die must be known-good before assembly. Test time multiplies.
- **Deliberately not UCIe.** Die-to-die links are plain SPI/UART/GPIO over the substrate. They are cheap, familiar and need no advanced packaging, at the cost of bandwidth and of selling bare dies into other vendors' packages.
- **Wave 2 without re-architecture.** The 28 nm AI-compute die (the D100 core) joins after the ₹50 Cr round on the same substrate and under the same supervision.
- **The package is supervised by its own die**: SKU-6 supervises the package, and the power tree comes from SKU-3.

## Key Concepts
- **Dies drawn**: SKU-3 PMIC, SKU-1 DRV, SKU-5 XCVR, SKU-6 SUP, SKU-4 MCU, and "AFE", plus a dashed "28nm AI" slot marked "Wave 2 (flip-chip)".
- **Die-to-die bar**: "SPI / UART / GPIO over substrate · power tree from SKU-3". The label is clipped on the figure.
- **Substrate**: 4-layer organic, PDN + escape routing, BGA 15×15 mm.
- **Attach**: wire-bond ×6, flip-chip ×1.
- **Outcome**: "= SDV domain controller / D100 UAV compute module".
- **SiP-class multi-die**: several bare dies in one standard package, as opposed to 2.5D/3D chiplet integration on an interposer.

## Mental Models
- Think of **the SiP as a small PCB in a package**, with the same partitioning logic and a much harder test problem.
- Use **"slowest link sets the ceiling"**: SPI/UART die-to-die links cap how much data the dies can exchange, fine for control, not for sensor streams into an AI die.
- Treat **the AFE die** as the SKU-2 DG-AFE6 front end until the sheet says otherwise.

## Anti-patterns
- **Assuming known-good die testing is free**: the plan needs per-die wafer-probe coverage before assembly.
- **A clipped label on the figure**: the die-to-die bar text overflows ("ie: SPI / UART / GPIO over substrate · power tree fro…"). Fix it before the sheet goes out.
- **Leaving the AFE and 28 nm dies unconnected in the drawing**: five dies have die-to-die lines; the AFE and 28 nm slot have none. A reviewer will ask how the AI die gets its data.
- **Combining a 120 V motor-drive die and a flip-chip compute die in one 15×15 mm BGA** without stating the isolation and thermal approach.

## Reference Tables

| Attribute | Value |
|---|---|
| Substrate | 4-layer organic, PDN + escape routing |
| Package | BGA 15×15 mm |
| Mature dies | 6, wire-bonded: SKU-3, SKU-1, SKU-5, SKU-6, SKU-4, AFE |
| Compute die | 1, flip-chip, 28 nm, Wave 2 after the ₹50 Cr round |
| Die-to-die | SPI / UART / GPIO over substrate; not UCIe |
| Power | tree from SKU-3 |
| Supervision | SKU-6 |
| Product | SDV domain controller or D100 UAV compute module |

## Worked Example
**Stress-testing the composition before quoting it (diligence walk-through)**
1. **Bandwidth.** Stereo 720p60 VIO input is ~0.5–0.7 Gbps per camera (ch11). SPI at tens of Mbps cannot carry it, so camera data must enter the 28 nm die directly (MIPI to its own pins), not over the die-to-die bar. Draw that path.
2. **Voltage domains.** SKU-1's buck/boost is rated 5–120 V, and SKU-3 takes a 28 V bus. A 15×15 mm BGA must separate these from 0.9–1.8 V compute rails: state creepage, clearance and whether the HV stages stay outside the package.
3. **Thermal.** SKU-3's LDO rails can dissipate ~13 W at full load (ch04), alongside a flip-chip AI die. State the package θJA and which die throttles first.
4. **Yield.** With six dies at an assumed 98% known-good rate each, package yield before assembly loss ≈ 0.98⁶ ≈ **88.6%**; at 95% each it is ≈ **73.5%**. Per-die test coverage is therefore a cost line, not a detail.
5. **Test access.** Decide whether each die is testable after assembly (JTAG chain through SKU-4?) or only before.

## Open Questions for Diligence
- Who is the packaging and test partner? The business plan lists this as unresolved.
- What known-good die test coverage is planned per die?
- Is the "AFE" die SKU-2's front end, or a separate part?

## Key Takeaways
1. The SiP is a credible low-cost integration path for control-class modules.
2. Its ceiling is die-to-die bandwidth; route high-rate data around the SPI/UART bar.
3. HV isolation, thermal and known-good die yield must be on the page.
4. Fix the clipped label and the unconnected dies in the figure.

## Connects To
- **ch04, ch07**: SKU-3 power tree and SKU-6 supervision.
- **ch11**: the Wave 2 28 nm die.
- **ch12**: the platform this composes.
- **deepgrid-sku-portfolio ch06, ch12**: KGD, UCIe self-critique and the packaging-partner gap.
