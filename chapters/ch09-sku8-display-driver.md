# Chapter 9: SKU-8 Rugged Display Driver
*Sheet 9 · FIG 8 · rugged TFT source driver + TCON · 130 nm · SKY130 / IHP SG13G2*

## Core Idea
A timing controller and a high-voltage source driver share one die: LVDS/DSI/parallel RGB in, a temperature-compensated pixel pipeline, 1,280 × 10-bit column DACs driving 3,840 sub-pixel outputs at 0–12 V, and a frame-freeze BIST. It has the most concrete policy anchor in the compendium: a PIL-5 line item for a BEL 17-inch rugged SXGA display due December 2027.

## Frameworks Introduced
- **The policy anchor** (text panel): a named DPSU socket with a date, cited from the DDP's 5th list on the sheet itself. It is the strongest demand evidence on any sheet.
- **Why 130 nm wins** (text panel): HV output amplifiers (0–12 V swing, thick oxide) and a 24 V gate-driver level shift are mature-node territory.
- **Temperature-compensated gamma** (the figure's note): LCD response shifts with panel temperature, so gamma is corrected against the ambient light + temperature sensor. That is what "rugged" means electrically.
- **Charge-sharing precharge** cuts driver power by ~40% by recycling charge between columns before each line is driven.
- **Safety in the pixel path**: frame-freeze detection, ASIL-B capable. A frozen image on a cockpit or vehicle display is a hazard, not a glitch.

## Key Concepts
- **Video input**:
  - LVDS RX: 4 lanes + clock, SXGA 1280×1024 at 60 Hz
  - DSI RX: 2 lanes, 1.5 Gbps per lane
  - line buffer: dual-port SRAM, 4 lines × 1280 × 24 bit
- **Pixel pipeline**: de-gamma (input linearise) → colour (3×3 matrix + white point) → gamma LUT (10-bit per channel) → dither (temporal + spatial).
- **Timing + test**:
  - TCON: H/V timing generator, programmable porches
  - PLL: pixel clock, 108 MHz for SXGA
  - test pattern: colour bars
  - BIST + safety: frame-freeze detection, ASIL-B capable
- **Control bus**: SPI/I²C register file, 200 MHz.
- **Column (source) drivers, high-voltage**:
  - DAC bank: 10-bit, "1 per column", resistor-string + buffer
  - output amplifiers: rail-to-rail, 0–12 V swing, thick oxide
  - column outputs ×1,280 (3 sub-pixels, so 3,840 outputs)
- **Row + backlight**:
  - gate driver interface: row scan, up to 24 V level shift
  - VCOM: common electrode
  - backlight: PWM + LED string, local dimming in 8 zones
  - ambient: light + temperature sense
- **Frame timing**: 1,024 active rows, 16.7 ms frame. Cycle-3 MPW (text).

## Mental Models
- Think of **the TCON as digital and the source driver as analog**: this SKU sells the combination because imports ship them as two parts.
- Use **"sunlight readable"** as a system target (contrast from 0 to 100 k lux) that depends on backlight and panel as much as on this chip.
- Treat **the PIL-5 item as permission, not an order**: the business plan's three buyer clocks apply.

## Anti-patterns
- **"1 DAC per column" beside 3,840 outputs**: either there are 3,840 DACs, or 1,280 DACs are multiplexed across R, G and B. The sheet does not say, and the choice drives area and settling time.
- **Quoting −40% driver power as measured**: it is a target on an illustrative panel.
- **Price confusion**: the sheet says $3–12 per socket, while the business plan's FY31 table uses $70–110. Reconcile before quoting either.

## Reference Tables

| Characteristic panel | Axis | Target |
|---|---|---|
| Luminance uniformity | corner → centre | >90% |
| Gamma error | trim steps | <1 LSB |
| Sunlight contrast | 0 → 100 k lux | readable |
| Driver power | precharge off → on | −40% |

| Market | Value | Grade |
|---|---|---|
| Volume | 2–6 M/yr India rugged + industrial panel driver sockets | internal |
| ASP | $3–12 TCON + source driver | internal |
| Anchor | PIL-5: BEL 17-inch rugged SXGA display, due Dec 2027 | cited, DDP 5th list |
| Replaces | Novatek/Himax-class TCON + source drivers | text |
| Use cases | defence displays (cockpit, console, vehicle MFD); industrial HMI and medical monitors; rail and metro passenger information; automotive cluster and centre stack | |

## Worked Example
**Checking the video arithmetic (reconstruction)**
1. **Pixel clock.** VESA SXGA at 60 Hz uses a total raster of 1,688 × 1,066 including blanking; 1,688 × 1,066 × 60 ≈ 107.96 MHz → **108 MHz**. ✓ Frame time 1 / 60 = **16.7 ms**. ✓
2. **Line buffer.** 4 × 1,280 × 24 = 122,880 bits ≈ **15.4 KB**: small, so on-die SRAM is realistic.
3. **DSI capacity.** Active video 1,280 × 1,024 × 60 × 24 = 1.84 Gbps; 2 lanes × 1.5 Gbps = 3.0 Gbps. ✓ About 39% margin for blanking and protocol overhead.
4. **LVDS capacity.** At a 108 MHz pixel clock with 7 bits per lane per cycle, 756 Mbps per lane across 4 data lanes carries 24-bit colour plus sync. ✓
5. **Outputs.** 1,280 columns × 3 sub-pixels = **3,840** HV outputs at 0–12 V: the area and pin-count driver of the die, and the reason packaging (likely chip-on-film for a source driver) belongs in the plan.

## Open Questions for Diligence
- How many HV output amplifiers physically exist, and what is their offset spread across 3,840 outputs? That spread sets luminance uniformity.
- What package carries 3,840 outputs?
- Has BEL confirmed that the PIL-5 display's panel interface and resolution match this part?

## Key Takeaways
1. This sheet has the clearest demand anchor; lead with the PIL-5 item and date.
2. The HV output stage is the physics case and the area driver.
3. Resolve DAC-per-column versus sub-pixel multiplexing.
4. Reconcile the $3–12 socket price with the business plan's $70–110.
5. The video arithmetic checks out.

## Connects To
- **ch14**: SKU 17 Cockpit MFD Controller (130 nm HV) is the next derivative.
- **deepgrid-sku-portfolio ch10, ch12**: PIL-5 matches and list-entry permission.
