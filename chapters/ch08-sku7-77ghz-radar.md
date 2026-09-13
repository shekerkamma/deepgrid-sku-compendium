# Chapter 8: SKU-7 77 GHz MIMO Radar
*Sheet 8 · FIG 7 · 2 TX / 4 RX MIMO · SiGe HBT front end + 130 nm CMOS baseband*

## Core Idea
An FMCW radar is split into two dies at the ADC:
- an IHP SG13G2 SiGe HBT die (ft 350 / fmax 450 GHz) carrying chirp synthesis, two PAs and four receive chains
- a 130 nm CMOS die running range and Doppler FFTs, CFAR, beamforming and a target list

The sheet prints its own risk: the SiGe front end "has no FPGA equivalent and is proven on silicon or not at all". Its range and resolution figures need reconciling.

## Frameworks Introduced
- **Partition at the ADC** ("Gen-1 automotive partitioning"): RF on the fastest process that meets the physics, digital on the cheapest.
  - When to use: any mixed RF/digital part whose RF needs a specialist process.
  - Failure mode: scheduling the part as though FPGA work reduced RF risk. It reduces only the CMOS half.
- **Beat frequency carries range; phase across RX carries angle** (the figure's note). Range comes from the FFT of each chirp's beat signal, velocity from the FFT across chirps, and angle from phase differences across the receive array.
- **Honesty on the sheet**: the risk statement is printed on the figure, not in an appendix.
- **Two sockets at once**: AD2 mirror-tower radar (captive), defence perimeter and counter-UAS, and automotive AEB. The text calls it the only SKU with captive automotive and defence sockets simultaneously.

## Key Concepts
- **SiGe die · chirp synthesis + transmit**:
  - XTAL 40 MHz → ramp generator (sawtooth/triangle) → fractional-N PLL (1 MHz loop BW) → VCO 38.5 GHz (SiGe HBT) → ×2 to 77 GHz
  - PA 13 dBm on TX1 and TX2
  - LO distribution to the receivers
  - "4 GHz sweep bandwidth → 3.75 cm range resolution"
- **SiGe die · receive array, 4 channels**: LNA (3.5 dB NF) → mixer → IF amp → AAF (10 MHz low-pass) → ADC (12-bit, 40 Msps). As drawn, the ADCs sit inside the SiGe die outline, while the text puts them on the CMOS die.
- **CMOS die · 130 nm · 200 MHz**:
  - range FFT: 1024-point, radix-4, 26 µs per chirp
  - Doppler FFT: 128 points across chirps (velocity bins)
  - range-Doppler map buffer: SRAM 512 KB
  - windowing: Hann/Blackman
- **Detection + output**:
  - CFAR (cell-averaging constant false-alarm rate)
  - angle estimation: digital beamforming, "4 RX → 15° resolution"
  - target list: range, velocity, angle, RCS; up to 64 tracks per frame
  - output to the ECU over CAN-FD or Ethernet
- **FMCW frame**: 128 chirps per frame, 20 ms frame.
- **Schedule (text)**: IHP MPW FY28.

## Mental Models
- Think of **range resolution as bandwidth** (c / 2B) and **maximum range as IF bandwidth and FFT length**. They trade against each other within one chirp profile.
- Use **"virtual array"**: 2 TX × 4 RX = 8 virtual elements, which is what makes ~15° angular resolution plausible. 4 RX alone gives roughly twice as coarse.
- Treat **the SiGe test chip as its own programme** with its own budget (₹1.4–1.8 Cr for the two-die part).

## Anti-patterns
- **Quoting 3.75 cm and 150 m together** (worked example).
- **Drawing the ADCs inside the SiGe outline** while stating the ADC is the boundary. Move them, or say which side owns them.
- **"4 RX → 15°"**: attribute the resolution to the 2×4 MIMO virtual array.
- **The template strip claiming FPGA prototyping** without the sheet's own caveat beside it; here the caveat is correctly present.

## Reference Tables

| Characteristic panel | Axis | Target |
|---|---|---|
| Range profile | 0 → 150 m | 3.75 cm resolution |
| Angular response | −60 → +60° | 15° |
| Chirp linearity | 0 → 40 µs sweep | <50 kHz rms |
| NF over temperature | −40 → +125 °C | 3.5 dB |

| Market (rough internal estimate) | Value |
|---|---|
| Volume | 2–8 M/yr India automotive + security radar |
| ASP | $8–25 front end (business plan table uses $35–60 per set) |
| Addressable | ~$0.15–0.4 B |
| Use cases | blind spot, cross-traffic, AEB; perimeter security and intrusion; level and presence sensing; drone altimetry and obstacle detection (feeds D100) |

## Worked Example
**Do the chirp numbers support 150 m at 3.75 cm? (reconstruction from the sheet's own figures)**
1. Range resolution = c / (2B) = 3×10⁸ / (2 × 4×10⁹) = **3.75 cm**. ✓
2. The sweep lasts 40 µs (chirp-linearity axis), so slope S = 4 GHz / 40 µs = 10¹⁴ Hz/s.
3. Beat frequency f_b = 2RS / c. The 10 MHz anti-alias filter caps f_b, so R_max = f_b · c / (2S) = 10⁷ × 3×10⁸ / (2×10¹⁴) = **15 m**. The ADC's 20 MHz Nyquist limit would allow 30 m.
4. FFT check: 40 µs × 40 Msps = 1,600 samples per chirp. A 1024-point FFT gives 512 usable bins; 512 × 3.75 cm = **19.2 m**.
5. Covering 150 m at 3.75 cm needs 150 / 0.0375 = 4,000 bins, and at this slope f_b = 100 MHz, beyond the AAF and ADC.
6. Frame check: 128 chirps × 40 µs = 5.12 ms of chirping inside a 20 ms frame; range FFTs at 26 µs × 128 ≈ 3.3 ms. ✓

Conclusion: 3.75 cm is a short-range (~15–19 m) profile. A 150 m profile needs roughly 10x less sweep bandwidth, giving about 37.5 cm resolution. Show both chirp profiles, as production radars do.

## Open Questions for Diligence
- Which SiGe blocks (VCO, PA, LNA) have IHP silicon data, and at what output power and NF?
- Is the 13 dBm PA output at the die or at the antenna port after the package transition?
- Does the BEL electronic-warfare specification change the band, and with it the SiGe versus CMOS choice?

## Key Takeaways
1. The partition is right, and the risk statement belongs on the figure.
2. Publish at least two chirp profiles; one cannot deliver both 3.75 cm and 150 m.
3. Credit angular resolution to the MIMO virtual array.
4. Fix the ADC placement in the drawing.
5. Budget the SiGe test chip separately; FPGA work does not touch it.

## Connects To
- **ch11**: drone altimetry and obstacle detection feed D100.
- **ch14**: SDR transceiver (SKU 16) as the SiGe + CMOS follow-on.
- **deepgrid-sku-portfolio ch10**: PIL/DMA radar matches and the BEL EW specification.
