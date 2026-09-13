# Patterns

## Two chips to one die
**When to use**: positioning a mature-node SoC against incumbent chip sets.
**How**: name the incumbent pair (driver + MCU, AFE + MCU, TCON + source driver), then show which blocks merge and which physics lets them merge at 130 nm.
**Trade-offs**: one qualification and one supply line for the customer; a larger mixed-signal die, and more analog risk for Deepgrid. (Ch 2, 3, 9)

## Loop in hardware, supervision in software
**When to use**: control loops faster than interrupt latency (motor PID, fault response).
**How**: build the datapath (PID → CORDIC → inverse Park/Clarke → LUT, or comparator → deglitch → latch) in hardware; let the core set targets through registers.
**Trade-offs**: fixed algorithms in silicon, in exchange for guaranteed timing. (Ch 2, 7)

## Contain faults stage by stage
**When to use**: power trees on hostile buses.
**How**: give each stage its own protection: clamp and UVLO/OVLO at the input, a cycle-by-cycle limit in the pre-regulator, window monitors and foldback per rail, and SEU-hardened sequencing.
**Trade-offs**: more area and trim; faults stay local. (Ch 4)

## Time-diverse lockstep
**When to use**: ASIL-D control, safety islands, flight-critical cores.
**How**: duplicate the core with a 2-cycle skew; compare bus activity and retired instructions; protect the bus and memory with ECC; raise FAULTn within 2 cycles.
**Trade-offs**: twice the core area, which is cheap at 130 nm; the comparator needs its own diagnostics. (Ch 5, 10, 11)

## Partition at the ADC
**When to use**: RF parts whose front end needs a specialist process.
**How**: SiGe die up to the IF and anti-alias filter; CMOS from the ADC onward; FPGA-validate only the CMOS half; budget a separate RF test chip.
**Trade-offs**: two dies and higher NRE; each half on its cheapest adequate process. (Ch 8)

## Hardware failsafe island
**When to use**: autonomous systems that must be certifiable without certifying mission software.
**How**: an isolated power and clock domain with a link monitor, safe-state FSM and a direct actuator path that bypasses all compute.
**Trade-offs**: extra pins, area and routing; a small, auditable safety case. (Ch 11)

## Scope box on the figure
**When to use**: any sheet where readers may assume more capability than claimed.
**How**: print one sentence stating what the part owns and what it does not (SKU-9 zonal layer; radar "proven on silicon or not at all"; roadmap honest boundary).
**Trade-offs**: smaller apparent ambition; much higher reviewer trust. (Ch 8, 10, 14)

## Mixed-criticality domains behind a regulated matrix
**When to use**: platforms combining Linux, safety and accelerators.
**How**: separate secure, safe, host and accelerator domains, each with its own bus and memory, meeting at an AXI matrix with per-manager monitoring and regulation.
**Trade-offs**: complex integration; freedom from interference by construction. (Ch 12)

## Compose dies in a SiP
**When to use**: modules needing HV, analog, control and compute together at low packaging cost.
**How**: one die per process on an organic substrate, wire-bonded, with a flip-chip compute die later; die-to-die control over SPI/UART/GPIO; route high-rate data directly to the die that needs it.
**Trade-offs**: low cost and flexibility; limited die-to-die bandwidth, and yield that multiplies across dies. (Ch 13)

## Shrink by channel count
**When to use**: deciding whether a part leaves 130/180 nm.
**How**: keep parts limited by voltage or analog accuracy; move parts limited by parallel DSP chains to 90–45 nm, keeping analog IO rings at 130 nm; move compute to 28 nm and below.
**Trade-offs**: a multi-node supply chain; each part on its binding constraint. (Ch 14)

## Arithmetic box on every figure
**When to use**: before any sheet reaches an investor or DPSU reviewer.
**How**: recompute the sheet's own numbers: units × ASP against addressable, rate × length, range against FFT bins, LDO dissipation, tolerance stacks, tapeouts against the ramp. Print the correction.
**Trade-offs**: exposes weak figures early; that is the point. (Ch 1, 4, 6, 8, 14)

## Grade evidence by band
**When to use**: quoting any number from a sheet.
**How**: block annotation = target; timing = sketch; characteristic = illustrative target; market = internal estimate unless cited; prototype strip = template.
**Trade-offs**: less quotable copy; no diligence surprises. (Ch 1)
