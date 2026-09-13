# Chapter 12: DG SDV Platform
*Sheet 12 · labelled "TRACK B" · "The Architecture Behind It All"*

## Core Idea
A heterogeneous platform reference sits on a 64-bit AXI4 matrix with AXI-REALM monitors on its managers:
- a secure domain (dual lockstep)
- a safe domain (triple-core lockstep)
- a Linux host domain (dual RV64GCH)
- a dynamic scratchpad-memory domain
- vector and integer multi-core accelerator domains

The mature-node SKUs are dies around it: "one architecture, many dies". Its block vocabulary closely matches a published open-source platform, which should be stated.

## Frameworks Introduced
- **One architecture, many dies** (sheet caption): SKU-9 is the mature-node zonal edge of this platform; D100 is its airborne sibling; SKU-3 powers, SKU-6 supervises, SKU-5 connects.
- **Mixed-criticality by domain.** Security, safety, a general-purpose Linux host and accelerators each get their own domain, bus and memory, and meet only at the AXI4 matrix, where AXI-REALM units monitor and regulate each manager's traffic.
  - Why it works: a misbehaving Linux or accelerator workload cannot starve the safety domain of bus access, because its bursts are regulated at the matrix.
- **Private scratchpads for predictability.** The safe domain uses private DSPM/ISPM with ECC, and the accelerators use interleaved SPM banks on a low-latency TCDM bus, avoiding cache timing variation where timing matters.

## Key Concepts
- **DG Secure domain**:
  - peripherals: OTP ctrl, UART, SPIM, lifecycle, timers, alert
  - AON: WDT, reset, power and clock managers
  - JTAG debug; dual lockstep DGI RV32 ×2
  - TL-UL peripheral and system buses
  - crypto DSAs: AES128, SHA2, RNG, OTBN, KMAC, HMAC, key manager, DMA
  - PLIC; OTP memory, BootROM, main SPM
- **DG Safe domain**:
  - DGCV32 RT ×3 with FPU (triple-core lockstep)
  - private DSPM + ECC, private ISPM + ECC
  - OBI system bus; Regbus/APB peripheral bus
  - CLIC, PCRs, generic timer, ECC manager, boot ROM, JTAG debug
- **DG Dynamic SPM**: dynamic addressing switch, bank-group arbiter, bank groups 0 … N−1 of SPM banks + ECC.
- **DG Accelerator domain**:
  - vector multi-core accelerator: interleaved SPM (M banks), TCDM bus, DGS RV32 + DMA, DGS RV32 + DG RVV with FPU0–3, IPU and VRF
  - integer multi-core accelerator: interleaved SPM, TCDM bus, DMA, DGCV32 #0 … #11 in an HMR cluster with an HMR manager, plus a heterogeneous accelerator slot
- **DG Host domain (Linux)**:
  - 2 × DGCVA6RT RV64GCH with MMU and FPU
  - "self-invalidating cache coherence" with L1 D$ and I$
  - partitionable hybrid LLC/SPM, HyperRAM controller, JTAG debug, serial link, DMA, mailbox unit
- **DG Boot & host interfaces**: Regbus/APB with UART, QSPI, PLIC, GPIO ×32, USB, CLINT/CLIC per core, I²C, host PCRs, interrupt router, AXI-REALM guard & config, BootROM.
- **Shared peripherals**: PCRs, CAN ×1, ETH ×1, WDT ×1, PWM timers, generic timers.
- **Legend**: red squares are AXI-REALM units (real-time monitoring and regulation for managers); teal dots are bus protocol adapters.
- **RV64GCH**: 64-bit RISC-V with general extensions, compressed instructions and the hypervisor extension.
- **HMR**: hybrid modular redundancy, runtime-switchable between independent and redundant core execution.

## Mental Models
- Think of **the matrix as a border crossing**: every domain's traffic is inspected and rate-limited on entry.
- Use **"different rules per domain"**: the host domain has caches, an MMU and compressed instructions; the safe domain uses scratchpads and lockstep. This is consistent with DGridRiscV's no-cache rule, which applies to control silicon, not Linux hosts.
- Treat **the platform as a reference**, not a SKU: no market panel, no node, no price.

## Anti-patterns
- **Presenting the platform as original Deepgrid architecture without stating lineage** (worked example).
- **Reusing "Track B" for two things**: the compendium labels this page Track B; the whitepaper uses Track B for the D100 28 nm programme.
- **Implying a node**: the page states none. The honest reading, from the roadmap boundary, is that the host and accelerator domains are 28 nm-and-below territory.

## Reference Tables

| Domain | Cores | Safety/security mechanism | Memory |
|---|---|---|---|
| Secure | DGI RV32 ×2, dual lockstep | crypto DSAs, lifecycle, OTP | main SPM, OTP, BootROM |
| Safe | DGCV32 RT ×3 + FPU, triple lockstep | ECC manager, CLIC | private DSPM/ISPM + ECC |
| Host (Linux) | DGCVA6RT RV64GCH ×2 | MMU, cache coherence | L1 caches, LLC/SPM, HyperRAM |
| Vector accelerator | DGS RV32 + DG RVV, FPU0–3 | n/a | interleaved SPM, TCDM |
| Integer accelerator | DGCV32 #0–#11 | HMR cluster | interleaved SPM, TCDM |
| Dynamic SPM | n/a | ECC per bank | bank groups 0…N−1 |

| Die that attaches | Role |
|---|---|
| SKU-9 | zonal edge |
| D100 | airborne sibling |
| SKU-3 | powers |
| SKU-6 | supervises |
| SKU-5 | connects |

## Worked Example
**Stating the lineage before a reviewer does (diligence walk-through)**
1. Read the block names: CVA6-class RV64GCH host ("DGCVA6RT"), TL-UL buses with OTBN, KMAC, HMAC and a lifecycle controller in the secure domain, an OBI/Regbus safe domain with a triple-core lockstep and CLIC, AXI-REALM, HMR clusters, TCDM accelerator buses, a dynamic SPM and HyperRAM.
2. Compare with published work: this combination matches the open-source Carfield platform from ETH Zürich and the University of Bologna's PULP group, which integrates a CVA6 Linux host, an OpenTitan-derived secure domain, a safety island, vector and integer accelerator clusters, and AXI-REALM.
3. The sheet uses "DG" prefixes and no attribution.
4. Recommended edit: add a line such as "platform reference derived from the open-source Carfield architecture (PULP), with Deepgrid's domains, SKUs and integration". List the licences of the reused IP in the data room.
5. Why it matters: an undeclared open-source base read as original work damages the credibility of every other sheet, while a declared one is a strength (proven RTL, an active community).

## Open Questions for Diligence
- Which blocks are reused open-source RTL, which are modified, and which are Deepgrid-original?
- Which node is this platform intended for, and is any part of it on a 130 nm MPW?
- Is "self-invalidating cache coherence" Deepgrid IP or upstream?

## Key Takeaways
1. The platform explains how the SKUs compose; it is a reference, not a product.
2. Mixed-criticality domains regulated by AXI-REALM are the core idea.
3. State the open-source lineage and licences explicitly.
4. Stop using "Track B" for two different things.

## Connects To
- **ch10**: SKU-9 as the zonal edge.
- **ch11**: D100 as the airborne sibling.
- **ch13**: the SiP as the physical composition of these dies.
