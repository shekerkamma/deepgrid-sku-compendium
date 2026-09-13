# Chapter 7: SKU-6 Voltage Supervisor
*Sheet 7 · FIG 12 · quad-rail, latched fault · 130 nm · SKY130 / IHP SG13G2*

## Core Idea
Four identical sense chains (divider → chopper comparator → deglitch counter) feed a maskable, latched fault matrix and open-drain FAULTn/RESETn outputs, with a curvature-corrected reference and a window watchdog. It is the simplest die in the catalogue, which is exactly why it goes through MIL-883 first.

## Frameworks Introduced
- **Qualification pathfinder** (text panel): the simplest die carries the screening flow for everything behind it, while filling the highest socket count (every LRU, every D100 module).
  - Why it works: screening surprises are cheapest to learn on a part with one sense chain per rail.
- **The deglitch counter is the design story.** A digital counter (8 µs at 8 MHz) ignores switching-noise spikes without slowing a real fault response beyond 8 µs. The figure's note: "the deglitch counter is what stops a switching-noise spike from resetting the board".
- **FAULTn before RESETn.** FAULTn asserts as soon as the deglitch completes and the fault latches; RESETn follows. When the rail recovers, FAULTn clears, and RESETn releases after a trimmed 200 ms delay. The board learns why before it is reset.
- **Accuracy from trimmed poly and a corrected bandgap.** The figure's note: "bandgap + trimmed poly resistors are what make the threshold accuracy claim possible on 130 nm."

## Key Concepts
- **Sense chain (one per rail)**:
  - VIN1 5V0, VIN2 3V3, VIN3 1V8, VIN4 1V2, each with 2 kV ESD
  - divider: poly R ladder, 0.1% match
  - comparator: chopper-stabilised, 8 mV hysteresis; ±1% on 5V0 and 3V3, **±2% on 1V8 and 1V2**
  - deglitch: digital counter, 8 µs at 8 MHz
- **Fault logic**: fault matrix (maskable, latched) with priority encoder, mask register, latch, I²C registers.
- **Output stage**: RESETn (open-drain, 200 ms delay, trimmed); FAULTn (open-drain, asserts before RESETn); MR (manual push-button in).
- **Reference + timebase**:
  - bandgap: curvature-corrected, 1.20 V, 10 ppm/°C (PTAT/CTAT)
  - trim: OTP 5-bit, ±0.5% after trim
  - oscillator: RC 8 MHz, ±2% over −40 to +125 °C, with a reference current
- **Watchdog**: window logic (open + close, 1 ms / 10 ms window), WDI kick input, timeout counter → RESETn.
- **Path (text)**: sky130 → SCL; cycle-1 or cycle-2 MPW.

## Mental Models
- Think of **the supervisor as the board's referee**: it must be the last thing to fail and the first thing trusted.
- Use **"deglitch in counts, not time"**: 8 µs at 8 MHz is 64 clock edges, so oscillator tolerance moves it by the same ±2%.
- Treat **quiescent current as the hidden spec**: an always-on 8 MHz oscillator and four chopper comparators must fit in <12 µA.

## Anti-patterns
- **Claiming "±1% from −40 °C to +125 °C" for all rails**: the figure draws ±2% comparators on 1V8 and 1V2. Either quote per-rail accuracy or fix the low-rail design.
- **The template "81.25 MHz → 200 MHz fixed" strip** on a part with no processor.
- **Assuming the watchdog windows are the oscillator's accuracy**: the 1 ms / 10 ms windows inherit the ±2% RC tolerance.

## Reference Tables

| Characteristic panel | Axis | Target |
|---|---|---|
| Threshold drift | −40 → +125 °C | ±1% |
| Response time | overdrive 5 → 200% | <8 µs |
| Quiescent current | 4 → 1 rails | <12 µA |

| Timing sequence (fault response) | Event |
|---|---|
| t0 | VIN3 droops |
| comparator | trips (after hysteresis) |
| +8 µs | deglitch completes; FAULTn asserts; fault latched |
| RESETn | asserts after FAULTn |
| rail recovers | comparator, deglitch and FAULTn clear |
| +200 ms | RESETn releases |

| Market (rough internal estimate) | Value |
|---|---|
| Volume | 30–80 M/yr India board-level supervisor sockets |
| ASP | $0.2–0.8 commercial; screened grades 10–100x |
| Positioning | attach part: one or more on nearly every PCB |
| Replaces | TI / Maxim supervisors |

## Worked Example
**Tolerance stack on the timing and threshold claims (reconstruction)**
1. Deglitch = 8 µs × 8 MHz = **64 counts**. With the RC oscillator at ±2%, 64 counts span 7.84–8.16 µs, so "<8 µs" needs the counter set to about 62 counts to hold at the fast corner.
2. RESETn delay 200 ms at ±2% = 196–204 ms, assuming the delay counts the same oscillator.
3. Threshold on the 1V2 rail at ±2% = 1.176–1.224 V, a ±24 mV band. The 8 mV hysteresis is one third of that band.
4. Result: the timing claims survive with one counter adjustment; the ±1% headline does not survive on the low rails as drawn.

## Open Questions for Diligence
- Is the 8 MHz oscillator duty-cycled? An always-on 8 MHz RC plus four choppers inside <12 µA is aggressive for 130 nm.
- What screening yield is assumed for the pathfinder lot, and what does stop rule S2 trigger if it fails twice?
- Is the OTP trim done at wafer sort or post-package?

## Key Takeaways
1. Simplest die, first through MIL-883, highest socket count: the sequencing logic is sound.
2. Quote accuracy per rail until the low-rail comparators match the headline.
3. Check deglitch and window timings against the ±2% oscillator.
4. Quiescent current is the spec most likely to move on silicon.

## Connects To
- **ch04**: reference and window-monitor ideas shared with SKU-3.
- **ch13**: "SKU-6 supervises the package" in the SiP.
- **deepgrid-sku-portfolio ch09, ch13**: pathfinder sequencing and stop rule S2.
