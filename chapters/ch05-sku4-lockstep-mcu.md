# Chapter 5: SKU-4 Microcontroller (Lockstep MCU)
*Sheet 5 · FIG 6 · ISO 26262 ASIL-D*

## Core Idea
Two DGridRiscV cores run the same program two clock cycles apart. A bus-and-retire comparator raises FAULTn within two cycles of a mismatch, and ECC protects the AHB end to end. The text panel's point: this lockstep RTL already exists and is fault-injection tested from the D100 programme, so this SKU productises a proven block.

## Frameworks Introduced
- **Time-diverse lockstep.** Core 0 runs "in step"; core 1 runs "+2 cyc skew". A transient that hits both cores at once lands on different instructions, so the comparator sees a difference.
  - Why it works: skew turns a common-cause glitch (clock, supply) into a detectable divergence, at the cost of doubling core area. Older nodes make that affordable.
  - Failure mode: outputs compared only at retire can miss a fault that corrupts memory without changing the retired instruction. Hence bus compare plus retire compare, plus ECC on the bus.
- **Safety at the boundary of the lockstep domain.** Everything outside (system and peripherals) is protected by ECC on the AHB and memories rather than duplication.
- **Productise before you invent.** The sheet's credibility rests on reusing fault-injection-verified RTL, not on a new safety concept.

## Key Concepts
- **Lockstep domain · ASIL-D**: DGridRiscV core 0 and core 1 (RV32IM_Zicsr, cacheless AXI4), each with ALU, MUL/DIV, PMP, CSR + trap. Comparator: bus + retire compare, mismatch → FAULTn in 2 cycles.
- **System**: PLIC (vectored IRQ), DMA (8-channel scatter), DEBUG (JTAG + trace), CLK/RST (PLL, FDIR).
- **Bus**: AHB-Lite 32-bit, ECC end-to-end, 200 MHz, with an AHB→APB bridge to a 32-bit low-speed domain. The cores expose AXI4, so a bridge to AHB-Lite is implied but not drawn.
- **Memory**: FLASH 1 MB ECC, SRAM 256 KB ECC.
- **APB peripherals**: CAN-FD ×2, SPI (quad), ADC 12-bit 2 Msps, TIMER (PWM, QEI), GPIO (5 V tolerant).
- **FDIR**: fault detection, isolation and recovery, attached to clock/reset.
- **Flash caveat (text panel)**: the 1 MB flash maps to an SCL eNVM variant; the open-PDK v1 uses ROM + SRAM + XTS-QSPI.

## Mental Models
- Use **"duplication costs area, not speed"** for why lockstep belongs on 130 nm.
- Think of **the comparator as a shared building block**: the same block reappears in SKU-9's safety island and D100's flight-control pair.
- Treat **99% diagnostic coverage** as the ASIL-D bar the fault-injection campaign must demonstrate, not a result.

## Anti-patterns
- **Showing FLASH 1 MB on the open-PDK version**: the sheet itself says v1 has no flash. Label the variant on the figure.
- **Claiming ASIL-D certification**: the header is a path ("ASIL-D Path" in the text), not an assessment.
- **Quoting ≤2-cycle detection as end-to-end fault latency** (worked example).

## Reference Tables

| Characteristic panel | Axis | Target |
|---|---|---|
| Fault detect latency | by injection class | ≤2 cycles |
| Diagnostic coverage | across test suite | 99% |
| Power modes | run → standby | falling bars, no number |

| Market (rough internal estimate) | Value |
|---|---|
| Volume | 3–10 M/yr India functional-safety MCU sockets |
| ASP | $4–12 ASIL-D MCU |
| Addressable | ~$0.3–0.5 B (2.5–42x units × ASP) |
| Replaces | Microchip / Renesas functional-safety MCUs |
| Use cases | EV BMS and motor-control safety supervision; braking, steering, airbag; industrial safety PLC and robot joints; drone flight-critical redundancy (feeds D100) |

## Worked Example
**What "FAULTn in 2 cycles" means in nanoseconds (reconstruction at the stated 200 MHz)**
1. Clock period at 200 MHz = 5 ns.
2. Core 1 lags core 0 by 2 cycles = 10 ns, so a fault in core 0's instruction at time t is only compared when core 1 retires the same instruction at t + 10 ns.
3. The comparator then raises FAULTn within 2 cycles = 10 ns.
4. Worst case from the leading core's faulty retire to FAULTn ≈ 4 cycles = **20 ns**.
5. At the FPGA's 81.25 MHz the same 4 cycles take ≈ 49 ns. Both are far inside any automotive fault-tolerant time interval; the constraint is the ASIL-D coverage evidence, not latency.

## Open Questions for Diligence
- What fault-injection campaign produced the D100 verification: fault models, injection count and coverage achieved?
- Is the comparator itself duplicated or self-checking? A single comparator is a single point of failure.
- How is the AXI4-to-AHB-Lite crossing protected by ECC?

## Key Takeaways
1. Time-diverse lockstep plus ECC is the whole safety concept, and it is standard practice.
2. The asset is existing fault-injection-tested RTL; lead with that.
3. Label the flash variant; v1 is flash-less.
4. The comparator is reused in SKU-9 and D100; its verification pays three times.

## Connects To
- **ch02**: SKU-4 as the FOC safety brain beside SKU-1.
- **ch10**: SKU-9 safety island (same pair, mismatch → safe state).
- **ch11**: D100 FC/NAV cores.
- **deepgrid-sku-portfolio ch07**: DGridRiscV and why it has no cache.
