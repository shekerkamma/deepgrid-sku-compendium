# Chapter 11: Drone SoC D100
*Sheet 11 · FIG 3 · "PHASE 2 · the revenue-now silicon" · DGCA type-certification path*

## Core Idea
The D100 combines four blocks:
- hard real-time flight control (PX4/ArduPilot on a dual DGridRiscV)
- stereo visual-inertial odometry (EKF pose at 30 Hz)
- an optional ~10 TOPS NPU for detection and segmentation ("AI · variant 2 only")
- a hardware failsafe island with its own power and clock, which bypasses all of the above

The wedge is sound. The node statement is the most contradicted fact in the compendium.

## Frameworks Introduced
- **The failsafe island is the wedge** (on-sheet): link monitor (RC loss, GPS loss, IMU fault; "hardware, not firmware") → safe-state FSM (return-to-home or land; isolated power + clock) → independent path direct to the ESC, bypassing FC, VIO and AI. "Failsafe path is independent of the mission stack."
  - Why it works: certification authorities can reason about a small, isolated hardware path without certifying the mission software.
  - De-risking route: every block in the island is buildable and testable at 130 nm on the mature-node chips first.
- **Geometry, not learned perception.** VIO uses FAST + BRIEF features and an EKF with IMU pre-integration and sliding-window bundle adjustment. It survives GPS jamming (the LoC/LAC use case) because it relies on no GNSS and no trained model.
- **Two variants.** Variant 1 is FC + VIO ($100–300 SoC ASP); variant 2 adds the NPU ($500–800).
- **FPGA timing closure is a prototype milestone** (the sheet's caption): "The shipping part is a 130 nm ASIC at 200 MHz."

## Key Concepts
- **Flight control · hard real-time**:
  - DGridRiscV ×2 (RV32IM_Zicsr): FC core + NAV core, PX4/ArduPilot loop
  - IMU/MAG/BARO over SPI ×3 at 8 kHz
  - ESC OUT: DShot600, 8 channels
  - RC + telemetry: SBUS, CRSF, MAVLink over UART
- **Visual-inertial odometry**:
  - MIPI CSI-2: 2 lanes ×2, stereo 720p60
  - ISP: rectify + lens-shading correction
  - FEATURE: FAST + BRIEF, 2k points per frame
  - POSE ENGINE: EKF, IMU pre-integration, sliding-window BA, 30 Hz 6-DoF
- **AI (variant 2 only)**: NPU INT8/INT4, ~10 TOPS class (MAC array, SRAM 2 MB, DMA); YOLO-family object detection; segmentation for obstacle avoidance.
- **Interconnect**: AXI4 crossbar, 128-bit, 200 MHz (130 nm ASIC).
- **Platform**: LPDDR4 2 GB (32-bit), eMMC/NAND logging, PMU with 5 domains, secure boot, ETH/USB3 (payload + ground link), CAN-FD ×2 + SPI + I²C (gimbal, payload), JTAG.
- **Status (text panel)**: "FPGA prototype validates; product is 130 nm at 200 MHz fixed."

## Mental Models
- Think of **the island as a dead-man's switch in silicon**.
- Use **"separate money, separate clock"** (business plan): D100 is funded by a revenue-gated round, not the seed.
- Treat **three node statements in one document set** as a blocker for any external use.

## Anti-patterns
- **Placing a ~10 TOPS NPU and YOLO-class detection on "130 nm at 200 MHz"**: the roadmap sheet lists "Multi-TOPS NPU for onboard AI inference" and "Object detection / target recognition at frame rate" under "What 130 nm cannot do".
- **LPDDR4 on an open 130 nm PDK without naming the PHY source**: the sheet does not say where an LPDDR4 PHY comes from.
- **Citing the market tiles as third-party**: the footnote cites "DeepGrid Strategic Direction, Jul 2026", an internal document, even though the text panel says "Market (sheet, cited)".
- **Calling D100 "Phase 2"** while the roadmap lists it as SKU 10 in the Phase 1 (2026–2027, 130/180 nm) shipping catalogue.

## Reference Tables

| Where | What it says about the D100 node |
|---|---|
| D100 sheet (FIG 3) | 130 nm ASIC · SkyWater SKY130 / IHP SG13G2 · 200 MHz fixed; NPU ~10 TOPS |
| Roadmap (FIG 14) | SKU 10 in Phase 1 at 130/180 nm; multi-TOPS NPU is a sub-10 nm problem |
| SiP sheet | "The 28 nm AI-compute die (D100 core) joins after the ₹50 Cr round" (Wave 2) |
| Business plan (whitepaper) | TSMC 28 nm, Track B, separate ₹50 Cr revenue-gated round |

| Market (sheet) | Value |
|---|---|
| India drone market | $1.2–1.3 B (2025) → $2.7–3.2 B by 2030–34 |
| SoC ASP | FC + VIO $100–300; + AI $500–800 |
| Demand signal | ideaForge guided 340–450 units in Q4 FY26 alone |
| Use cases | military ISR under GPS jamming (LoC/LAC); GPS-denied warehouse inventory (IKEA/Verity, 250+ drones across 73 sites); urban surveillance and infrastructure inspection; country-of-origin mandate (no critical sub-components from land-border nations) |
| Gap | no indigenous FC + VIO SoC; VyomChakra is FC-only, C-DAC VEGA is exploration |

## Worked Example
**Reconciling the node before anyone quotes it (a decision walk-through)**
1. Split the die by function. The failsafe island, FC pair, IMU/ESC I/O and VIO front end (ISP, FAST/BRIEF, EKF) are fixed-point, control-heavy and plausible at 130 nm. The NPU (~10 TOPS), LPDDR4 interface and 128-bit crossbar are compute- and memory-heavy, and are the roadmap's own "cannot do at 130 nm" items.
2. Map to the documents. The SiP sheet already describes a mature-node module with a 28 nm "D100 core" added in Wave 2; the whitepaper puts D100 on TSMC 28 nm.
3. One consistent statement: "D100 variant 1 (FC + VIO + failsafe island) is proven at 130 nm; the AI variant's compute die is 28 nm, Track B, funded separately."
4. Edit the D100 sheet's product strip and the roadmap's SKU 10 entry to match, and move the NPU block to a "28 nm, Wave 2" region of the figure.
5. Data-rate sanity check for the VIO input: a 720p60 raw stream at 10–12 bits is roughly 550–660 Mbps per camera, within two MIPI D-PHY lanes. ✓

## Open Questions for Diligence
- Which FPGA build ran the failsafe island, and was its independence tested by killing the mission stack?
- What pose accuracy and drift does the VIO achieve on recorded flight data?
- What is the DGCA certification basis being targeted?

## Key Takeaways
1. The failsafe island is a genuine differentiator; prove it at 130 nm first.
2. Split D100 into a 130 nm control/VIO part and a 28 nm AI die in every document.
3. Label the market tiles as sourced from an internal strategy document.
4. Keep D100 money and schedule separate from the mature-node catalogue.

## Connects To
- **ch05, ch10**: lockstep pair reuse.
- **ch13**: the Wave 2 28 nm die.
- **ch14**: the honest boundary this sheet currently crosses.
- **deepgrid-sku-portfolio ch11**: Track B and the ₹50 Cr round.
