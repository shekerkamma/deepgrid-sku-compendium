# Chapter 1: Cover and sheet anatomy
*Cover page; common structure of sheets 2–14*

## Core Idea
The compendium is 14 landscape sheets built "to survive a semiconductor-literate diligence read": nine SKU sheets, the D100, the DG SDV platform, the SiP and the roadmap. Each sheet mixes five kinds of evidence of very different strength on one page, so the first skill is telling them apart.

## Frameworks Introduced
- **Five evidence tiers on one sheet**, strongest to weakest:
  1. **Block annotations**: design targets on each block, e.g. "Rds(on) 22 mohm" or "8 us @ 8 MHz". These are specifications to be met, not results.
  2. **Timing diagram**: a behavioural sketch of one sequence, e.g. Hall sectors to phase drive, or rail droop to reset release. It shows intent and ordering, not measured waveforms.
  3. **Characteristic panels**: always headed "CHARACTERISTIC (illustrative)", each with a dashed target line and a target value in the corner. The shapes are drawn, not measured. SKU-1 says so outright: "illustrative shapes; targets from the product brief".
  4. **Market · Volume · India**: three tiles (volume, ASP band, addressable or positioning) plus use cases. The footnote grades them: "ROUGH INTERNAL ESTIMATE - not third-party researched" on SKU-3 to SKU-7; "rollout scale is public, unit split is ours" on SKU-2; "the PIL-5 item is cited from DDP's 5th list" on SKU-8; "cited: DeepGrid Strategic Direction, Jul 2026" on D100, which cites an internal document.
  5. **Prototype → product strip**: identical on every SKU sheet. Treat it as template text (see anti-patterns).
- **The text panels beside each figure** carry the argument. Every panel title is a claim type:
  - "Replaces" and "Socket & policy hook" on most sheets
  - "Why 130 nm wins" on SKU-5 and SKU-8
  - "Qualification pathfinder" on SKU-6
  - "The die boundary" and "Honesty on the sheet" on SKU-7
  - "The policy anchor" on SKU-8
  - "Scope honesty" and "Where it fits" on SKU-9
  - "The wedge", "Market (sheet, cited)" and "Status" on D100
- **Colour legend (SKU sheets)**: analog, digital/memory, compute, security, movement, interconnect. Some sheets swap in "lockstep core", "safety" or "control". Bus bars are green and name width and speed ("AXI BUS · memory-mapped register file · 200 MHz").

## Key Concepts
- **Figure numbers**: the sheets carry FIG 3–12 and FIG 14, lifted from a larger document; FIG 1, 2 and 13 are absent. D100 is FIG 3, SKU-9 FIG 4, SKU-1 FIG 5, SKU-4 FIG 6, SKU-7 FIG 7, SKU-8 FIG 8, SKU-2 FIG 9, SKU-3 FIG 10, SKU-5 FIG 11, SKU-6 FIG 12, roadmap FIG 14.
- **Target marker**: a dashed red line plus an orange end dot on a characteristic panel. It marks the spec to meet.
- **Attach part**: the market tile on SKU-5 and SKU-6 in place of an addressable figure. The part ships alongside every node or board.
- **Header standards line**: e.g. "SKU 1 · AEC-Q100 · UL94 · ISO 26262 ASIL-B/C · 130 nm CMOS (28 nm planned)". It states the intended qualification, not certification held.

## Mental Models
- Read a sheet **right to left**: text panel for the claim, figure for the design, footnote for the evidence grade.
- Treat **"illustrative" as "no data yet"**. A diligence reader will.
- Treat **any number repeated identically on every sheet as template**, until a sheet-specific source shows otherwise.

## Anti-patterns
- **Quoting a characteristic panel as performance**: "95% efficiency" on SKU-1 is a target line on a drawn curve.
- **Quoting the prototype strip per SKU**: "Artix-7 FPGA · 81.25 MHz · validation only → 130 nm ASIC · SkyWater SKY130 / IHP SG13G2 open PDK · 200 MHz fixed" is stamped on:
  - the analog PMIC, transceiver and supervisor, whose text panels name SCL 180 nm production and have little or no processor to prototype
  - the radar, whose own caveat says the SiGe front end cannot be FPGA-prototyped

  Give each sheet its own strip.
- **Quoting 200 MHz as a per-SKU spec**: the whitepaper calls it an unmeasured processor target, and the DG32-LITE documents report a 50 MHz die clock.
- **Presenting the addressable tile as units × price**: it is not (worked example).

## Reference Tables

| Sheet | Volume tile | ASP tile | Third tile | Footnote grade |
|---|---|---|---|---|
| SKU-1 | 5–15 M/yr | $3–8 | ~$0.4–0.6 B addressable | rough internal estimate; specs from product brief |
| SKU-2 | ~250 M rollout | $2–5 | 10–30 M/yr at peak | rollout public, unit split internal |
| SKU-3 | 10–50 k/yr | $50–200 "rad-tolerant PMIC" | ~$40–90 M defence + space | rough internal estimate |
| SKU-4 | 3–10 M/yr | $4–12 | ~$0.3–0.5 B | rough internal estimate |
| SKU-5 | 20–60 M/yr | $0.4–1.5 | attach part | rough internal estimate |
| SKU-6 | 30–80 M/yr | $0.2–0.8 | attach part | rough internal estimate |
| SKU-7 | 2–8 M/yr | $8–25 | ~$0.15–0.4 B | rough internal estimate |
| SKU-8 | 2–6 M/yr | $3–12 | PIL-5 item, BEL, Dec 2027 | volumes internal; PIL-5 cited |
| SKU-9 | 4–12 M/yr | $6–20 | ~$0.3–0.7 B, zonal only | volumes internal; central compute out of scope |
| D100 | $1.2–1.3 B market 2025 | $100–800 | ideaForge 340–450 units Q4 FY26 | cites internal strategy doc, Jul 2026 |

## Worked Example
**Does "addressable" equal units × ASP? (reconstruction from the tiles)**

| Sheet | Units × ASP range | Addressable tile | Addressable ÷ units × ASP |
|---|---|---|---|
| SKU-1 | 5 M × $3 = $15 M … 15 M × $8 = $120 M | $400–600 M | 3.3x to 40x |
| SKU-3 | 10 k × $50 = $0.5 M … 50 k × $200 = $10 M | $40–90 M | 4x to 180x |
| SKU-4 | 3 M × $4 = $12 M … 10 M × $12 = $120 M | $300–500 M | 2.5x to 42x |
| SKU-7 | 2 M × $8 = $16 M … 8 M × $25 = $200 M | $150–400 M | 0.75x to 25x |
| SKU-9 | 4 M × $6 = $24 M … 12 M × $20 = $240 M | $300–700 M | 1.25x to 29x |

Conclusion: the addressable tile always includes something beyond this SKU's units at this SKU's price, such as adjacent silicon or a wider category, and no sheet says what. Either rename the tile ("India category silicon") or show the units × ASP product beside it.

## Key Takeaways
1. No number in the compendium is measured silicon; grade each by its band.
2. Characteristic panels are targets on drawn curves.
3. Replace the uniform prototype strip with a per-sheet node and prototype path.
4. Define the addressable tile, or diligence will compute units × ASP and find a gap.
5. The text panels carry the argument; keep their honesty statements.

## Connects To
- **ch04, ch06, ch07**: sheets where the template strip contradicts the text panel.
- **ch11, ch14**: the most serious cross-sheet conflict.
- **deepgrid-sku-portfolio ch03**: the DONE/DESIGNED/PLANNED/ESTIMATED labels to apply to each tier.
