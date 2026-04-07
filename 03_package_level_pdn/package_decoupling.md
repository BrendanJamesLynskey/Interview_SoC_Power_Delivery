# Package Decoupling

This section covers the selection, placement, and optimization of decoupling capacitors on IC package substrates, including MLCC characteristics, ESR/ESL considerations, and frequency coverage strategies.

---

### Q1. Why are package-level decoupling capacitors needed?

**Answer:**
Package-level decoupling capacitors bridge the frequency gap between PCB-level decoupling and on-die decoupling. PCB-mounted capacitors (bulk and ceramic) are effective from DC up to roughly 10 to 50 MHz, limited by their mounting inductance (via length through the PCB, trace length to BGA pad). On-die decoupling capacitance is effective from roughly 200 MHz to multi-GHz frequencies. The gap between 50 MHz and 200 MHz must be filled by package-mounted capacitors.

Package-mounted MLCCs are physically closer to the die than PCB capacitors (the current path is shorter), resulting in lower effective inductance. The inductance from a package-mounted MLCC to the die bump pads includes only the package substrate via and plane path (typically 20 to 100 pH), compared to the much longer path from a PCB capacitor through the PCB vias, BGA balls, and package substrate (typically 200 pH to 1 nH).

Without package-level decoupling, the PDN impedance would exhibit a large impedance peak in the 50 to 200 MHz range where neither PCB caps nor on-die caps are effective. This peak could exceed the target impedance by an order of magnitude, causing severe voltage noise at those frequencies.

The number of package-mounted MLCCs required depends on the target impedance. For a target of 1 mOhm and an ESR of 5 mOhm per MLCC, at least 5 capacitors are needed just to meet the target at their resonant frequency. Accounting for frequency coverage and anti-resonance management, high-performance SoC packages typically carry 100 to 400 MLCCs.

---

### Q2. What MLCC characteristics are important for package decoupling?

**Answer:**
The key MLCC parameters that determine its effectiveness as a package decoupling capacitor are:

**Capacitance value:** Determines the low-frequency boundary of the effective range. Larger values extend the useful range to lower frequencies. Typical values for package MLCCs range from 100 nF to 10 uF.

**Equivalent series resistance (ESR):** Determines the minimum impedance at the series resonant frequency. Lower ESR provides lower impedance at resonance but also provides less damping of anti-resonance peaks. Package MLCCs typically have ESR of 2 to 20 mOhm, depending on the size, value, and dielectric type.

**Equivalent series inductance (ESL):** Determines the high-frequency boundary of the effective range. ESL depends primarily on the capacitor body size and mounting geometry. Package-mounted MLCCs have ESL of 50 to 300 pH, much lower than PCB-mounted capacitors (which have 0.5 to 2 nH due to longer via paths). Low-inductance mounting techniques (bottom-terminated, interdigitated terminals, reverse-geometry) further reduce ESL.

**Physical size:** Package MLCCs are typically 0201 (0.6 mm x 0.3 mm) or 0402 (1.0 mm x 0.5 mm) form factors due to the limited space on the package substrate. Smaller sizes have lower ESL (shorter current path) but limited maximum capacitance.

**Dielectric type:** X5R and X7R are common dielectrics for decoupling. X5R is rated to 85C, X7R to 125C. Higher-capacitance MLCCs use higher-permittivity dielectrics (Class II) that exhibit capacitance derating with applied DC voltage (DC bias effect) and temperature variation.

**DC bias derating:** High-permittivity MLCCs lose capacitance when DC voltage is applied. A 10 uF X5R MLCC rated at 6.3 V might provide only 5 uF of actual capacitance at 0.85 V DC bias. The de-rated capacitance must be used in PDN analysis.

**Voltage rating:** Must exceed the maximum supply voltage with margin. For a 0.85 V core rail, MLCCs rated at 4 V or 6.3 V are typical.

---

### Q3. How do you select the optimal mix of MLCC values for a package?

**Answer:**
The MLCC value selection aims to achieve continuous impedance coverage across the target frequency range with no anti-resonance peaks exceeding the target impedance.

**Step 1: Determine the frequency range.** The package MLCCs must cover from where the PCB caps become inductive (typically 20 to 50 MHz) to where on-die decaps take over (typically 200 to 500 MHz).

**Step 2: Calculate the number of capacitors needed at resonance.** At the resonant frequency of N identical capacitors in parallel, the impedance is ESR/N. To meet the target:

```
N >= ESR / Ztarget
```

For ESR = 5 mOhm and Ztarget = 1 mOhm: N >= 5.

**Step 3: Choose a set of capacitor values.** Rather than using only one value, a mix of values extends frequency coverage. A common approach uses a geometric progression of values:

- 10 uF: resonates at low frequency, covers the low end of the package range
- 1 uF: resonates at an intermediate frequency
- 100 nF: resonates at a higher frequency, bridges toward on-die decoupling

**Step 4: Verify with simulation.** The complete PDN (including PCB model, package EM model, selected capacitors, and die model) is simulated to compute the impedance profile. Anti-resonance peaks between capacitor groups are checked.

**Step 5: Adjust for anti-resonance.** If anti-resonance peaks exceed the target, additional capacitors of intermediate values can be added to fill the gap. Alternatively, higher-ESR capacitors can be used to damp the resonance, though this raises the minimum impedance at resonance.

**Step 6: Consider area constraints.** The total number of MLCCs is limited by available space on the package substrate. High-performance packages allocate specific zones for capacitor mounting (top surface, bottom surface, or both). The layout must accommodate the required number within these zones.

A practical allocation for a high-performance SoC might be:
- 50 x 10 uF (0402 size)
- 100 x 1 uF (0201 size)
- 50 x 100 nF (0201 size)

Total: 200 MLCCs on the package substrate.

---

### Q4. What is the ESL of a package-mounted MLCC and what determines it?

**Answer:**
The ESL (equivalent series inductance) of a package-mounted MLCC includes the inductance of the capacitor body itself plus the inductance of the mounting pads, traces, and vias that connect it to the power/ground planes in the package substrate.

**Capacitor body inductance:** The internal inductance of the MLCC depends on its geometry. The current path inside the capacitor flows from one terminal through the interleaved electrode plates to the other terminal. Shorter, wider bodies have lower internal inductance. Typical body inductance for a 0201 MLCC is 50 to 100 pH, and for an 0402 MLCC is 100 to 200 pH.

**Mounting inductance:** The pads, short traces, and vias that connect the MLCC terminals to the power and ground planes in the substrate contribute additional inductance. This depends on the via depth (how many layers the via traverses), the trace length to the via, and the pad geometry. Typical mounting inductance is 20 to 100 pH.

**Total ESL:** The sum of body and mounting inductance is the total ESL. For a 0201 MLCC mounted on a package substrate with optimized via connections:

```
ESL_total = ESL_body + ESL_mount ~ 70 + 30 = 100 pH
```

**Reduction techniques:**

- **Low-inductance capacitor designs:** Reverse-geometry (wide-side terminals) and interdigitated-terminal (IDC) designs reduce body inductance by shortening and widening the current path. An IDC capacitor can have 50% lower body ESL than a standard configuration.

- **Bottom-terminated capacitors:** Capacitors with terminations on the bottom surface (land grid array style) reduce mounting inductance by eliminating solder fillet height.

- **Embedded capacitors:** Capacitors embedded within the package substrate (in a cavity or as a laminated thin-film layer) have the lowest ESL because the current path to the planes is extremely short.

- **Via optimization:** Using multiple vias per capacitor pad and minimizing via stub length reduces mounting inductance.

The ESL directly determines the upper frequency limit of the capacitor's effectiveness: f_high = Ztarget / (2*pi*ESL_total). For ESL = 100 pH and Ztarget = 1 mOhm: f_high = 1.59 GHz. This means a 100 pH-ESL MLCC remains useful to 1.59 GHz, which overlaps well with the on-die decoupling range.

---

### Q5. How does capacitor placement on the package substrate affect decoupling effectiveness?

**Answer:**
The physical location of an MLCC on the package substrate determines the inductance of the current path between the capacitor and the die, which directly affects the capacitor's high-frequency effectiveness.

**Proximity to die:** Capacitors placed directly under or beside the die (in the die shadow region) have the shortest path to the die bumps and thus the lowest mounting inductance. These locations provide the best high-frequency decoupling. However, the die shadow region on the top surface of the package is occupied by the die itself, so capacitors must be placed on the bottom surface (opposite to the die) or on the top surface outside the die footprint.

**Top surface vs. bottom surface:** On the top surface, capacitors are outside the die footprint and must connect laterally through the substrate planes before reaching the die bump area. On the bottom surface, capacitors can be placed directly under the die, providing short vertical via connections to the die-side planes. Bottom-mounted capacitors typically have 30 to 50% lower effective inductance than top-mounted capacitors of the same type.

**Via path length:** The number of substrate layers that the via must traverse between the capacitor pad and the die-side power plane determines the via inductance. A capacitor mounted on the bottom of a 10-layer substrate must traverse more via length than one mounted on a middle layer (in the case of embedded capacitors).

**Current sharing:** The effectiveness of each capacitor also depends on how much of the transient current demand from the die flows through that capacitor versus through other parallel paths (other capacitors, package planes). Capacitors closer to high-current die regions are more heavily utilized.

**Simulation-guided placement:** In practice, package designers use electromagnetic simulation to evaluate different capacitor placement configurations. The tool models the complete package with the proposed capacitor layout and computes the impedance profile. Capacitors are moved and added iteratively until the impedance profile meets the target everywhere. This optimization considers the constraints of available placement zones, signal routing clearances, and assembly manufacturability.

---

### Q6. What is the difference between embedded and discrete package capacitors?

**Answer:**
Discrete package capacitors are standard surface-mount MLCCs soldered onto the package substrate surface, while embedded capacitors are integrated within the package substrate itself.

**Discrete MLCCs:**
- Soldered to top or bottom surface of the package substrate
- Standard components (0201, 0402 sizes) readily available
- Capacitance values from 1 nF to 10 uF per component
- ESL of 50 to 300 pH (body + mounting)
- Require solder pads and via connections on the substrate
- Occupy surface area that might otherwise be used for signal connections
- Can be reworked or replaced if defective

**Embedded thin-film capacitors:**
- Fabricated as a high-permittivity dielectric layer within the substrate stackup
- Provide distributed capacitance across the entire plane overlap area
- Typical capacitance density of 1 to 20 nF/cm^2 depending on dielectric material and thickness
- Extremely low ESL (single-digit pH) because there is no body or mounting inductance
- Cannot be individually adjusted or reworked after substrate fabrication
- Add cost to the substrate manufacturing process
- Limited total capacitance compared to discrete MLCCs

**Embedded discrete capacitors:**
- Individual MLCC components placed in cavities milled into the substrate
- Combine the high capacitance of discrete MLCCs with the low mounting inductance of embedded structures
- More complex substrate fabrication
- Used in some high-performance server processor packages

The choice between embedded and discrete depends on performance requirements and cost. Embedded thin-film capacitors are used in high-performance designs where the lowest possible ESL is critical. Discrete MLCCs are more cost-effective and provide higher total capacitance. Most high-performance packages use both: embedded thin-film layers for high-frequency decoupling and surface-mount MLCCs for mid-frequency decoupling.

---

### Q7. How does temperature affect MLCC performance in package decoupling?

**Answer:**
Temperature affects MLCC characteristics in several ways that must be accounted for in PDN design.

**Capacitance variation:** Class II dielectrics (X5R, X7R) exhibit capacitance change with temperature. X5R is rated for +/-15% variation over -55C to 85C. X7R is rated for +/-15% over -55C to 125C. Outside these ranges, the capacitance can change more dramatically. For package-mounted MLCCs that operate near the die (which can reach 100 to 125C), using X7R dielectric is advisable. Class I (C0G/NP0) dielectrics have near-zero temperature coefficient but are limited to small capacitance values.

**ESR variation:** ESR generally decreases slightly with increasing temperature for ceramic capacitors, which is favorable (lower impedance at resonance). However, the change is small (typically less than 20%) and is usually not a critical design factor.

**DC bias derating with temperature:** The DC bias derating effect (capacitance reduction under DC voltage) worsens at higher temperatures for some dielectric types. The combined effect of DC bias and temperature can reduce the effective capacitance by 30 to 50% from the nominal datasheet value.

**Reliability:** MLCC reliability (resistance to cracking, delamination) is affected by thermal cycling. Package assembly processes subject MLCCs to solder reflow temperatures (260C peak), and the subsequent thermal cycling during operation can stress the capacitor body. Smaller MLCC sizes (0201, 0402) are more resistant to flex cracking than larger sizes.

**Design practice:** PDN analysis should use worst-case capacitance values that account for temperature derating, DC bias derating, and aging effects. A common approach is to derate the nominal capacitance by 30 to 50% for the PDN impedance simulation. For example, a 10 uF MLCC at 0.85 V DC and 100C might provide only 5 to 6 uF of effective capacitance. Using the de-rated value ensures the PDN meets specifications under worst-case conditions.

---

### Q8. What is the optimal strategy for distributing decoupling across multiple frequency decades?

**Answer:**
An effective decoupling strategy provides continuous impedance coverage across the full frequency range by ensuring that each decade of frequency has adequate capacitor coverage with smooth handoffs between adjacent decades.

**Step 1: Define the frequency bands.** Divide the total range (DC to max frequency) into bands corresponding to each PDN stage:
- DC to 100 kHz: VRM control loop
- 100 kHz to 10 MHz: PCB bulk and ceramic caps
- 10 MHz to 100 MHz: PCB ceramic and package MLCCs
- 100 MHz to 1 GHz: Package MLCCs and on-die decaps
- Above 1 GHz: On-die decaps only

**Step 2: Select capacitor values for each band.** Choose values whose series resonant frequency falls within the target band. The resonant frequency is:

```
f_res = 1 / (2 * pi * sqrt(ESL * C))
```

For package MLCCs with ESL = 100 pH:

| Capacitance | f_res |
|------------|-------|
| 10 uF | 5.0 MHz |
| 1 uF | 15.9 MHz |
| 100 nF | 50.3 MHz |
| 10 nF | 159 MHz |
| 1 nF | 503 MHz |

**Step 3: Ensure overlap between bands.** Each capacitor value covers approximately one decade of frequency centered on its resonant frequency. By selecting values spaced approximately one decade apart, the coverage overlaps and anti-resonance peaks are minimized.

**Step 4: Use enough capacitors per value to meet the target.** At each resonant frequency, the impedance is ESR/N (for N identical capacitors). Ensure N is large enough for each value.

**Step 5: Verify anti-resonance peaks.** Simulate the complete decoupling network and check that anti-resonance peaks between adjacent value groups do not exceed the target. If they do, add intermediate values or increase ESR for damping.

**Step 6: Distribute spatially.** Within each value group, distribute capacitors across the available mounting area to ensure uniform decoupling across the die. Concentrating all capacitors in one location leaves other areas poorly decoupled.

This systematic approach, sometimes called the "frequency-domain decoupling design methodology," is widely used in PDN design and is supported by most commercial PDN analysis tools.

---

### Q9. How do you measure the effectiveness of package decoupling in a manufactured system?

**Answer:**
After manufacturing, the package PDN impedance can be measured to verify that it meets the design intent and to correlate simulation models with physical results.

**VNA (Vector Network Analyzer) measurement:** A VNA connected to probes on the package measures the impedance as a function of frequency. Two-port measurements (using a shunt-through configuration) provide accurate impedance measurements down to sub-milliohm levels. Probe points can be at the BGA pads (measuring the package-only impedance) or at the die bumps (if the die has on-chip impedance measurement structures).

**On-die impedance sensing:** Some SoC designs include on-chip test structures that measure the PDN impedance from the die's perspective. These typically consist of a current source that injects a known AC current into the supply, and a voltage sensor that measures the resulting voltage perturbation. The ratio gives the impedance at the injection frequency.

**Time-domain reflectometry (TDR):** A TDR instrument sends a step voltage into the PDN and measures the reflected waveform. The impedance as a function of time (which corresponds to distance into the network) can be computed from the reflection coefficient. TDR provides a time-domain view of the PDN impedance profile.

**Load transient measurement:** An electronic load or an on-chip test circuit applies a current step to the supply, and the resulting voltage transient is captured on an oscilloscope. The voltage droop, overshoot, and settling time are compared to specifications and simulation predictions.

**De-embedding:** Measurements often include the effects of the measurement setup (probe parasitics, cable impedance, test fixture). De-embedding techniques (using calibration standards) remove these artifacts to obtain the true PDN impedance.

Measurement-to-simulation correlation is essential for building confidence in the modeling methodology. Typical agreement between simulation and measurement is within +/-3 dB (factor of 1.4 to 2) for impedance magnitude, with the phase and frequency of resonance peaks matching closely. Larger discrepancies indicate modeling errors (incorrect dielectric properties, missing parasitics, or inaccurate capacitor models) that should be investigated and corrected.

---

### Q10. What are emerging technologies for improved package decoupling?

**Answer:**
Several emerging technologies aim to improve package-level decoupling beyond what conventional MLCCs can provide.

**High-density embedded capacitor layers:** New high-k dielectric materials (barium titanate composites, thin-film PZT) can achieve capacitance densities of 50 to 200 nF/cm^2 when embedded as thin layers in the package substrate. This provides distributed, ultra-low-ESL decoupling across the entire die area.

**Silicon capacitor (trench cap) dies:** Dedicated silicon dies containing deep-trench capacitors (similar to DRAM trench cells) can provide 100 to 1000 nF in a few square millimeters of silicon area. These are mounted as companion dies in the package, adjacent to the SoC die, and connected through the package substrate or an interposer. They offer capacitance density of 200 to 2000 nF/mm^2.

**Integrated passive devices (IPD):** Thin-film passive components fabricated on silicon or glass substrates, containing precision capacitors, resistors, and inductors. They can be embedded in the package as known-good dies.

**3D stacked decoupling:** In 3D IC packages, decoupling capacitor dies or thin-film capacitor layers can be stacked directly on top of or beneath the active die using through-silicon vias. This minimizes the inductance to near-zero because the current path is vertical through a few hundred micrometers of silicon.

**Carbon nanotube and graphene capacitors:** Research-stage technologies that exploit the high surface area of nanostructured carbon materials to achieve very high capacitance density in small volumes. These are not yet commercially available for package decoupling.

**Ferroelectric capacitors:** Materials like hafnium zirconium oxide (HZO) are being explored for high-density embedded decoupling because they have very high permittivity and are compatible with existing semiconductor processing.

These technologies address the fundamental limitation of conventional MLCCs: the tradeoff between capacitance density and parasitic inductance. By placing high-density capacitance very close to the die with minimal interconnect inductance, they promise to extend effective decoupling to higher frequencies and reduce the total number of discrete components needed.
