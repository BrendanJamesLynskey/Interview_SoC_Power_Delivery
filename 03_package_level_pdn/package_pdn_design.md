# Package PDN Design

This section covers the design of power delivery networks in IC packages, including package plane design, resonance phenomena, and modeling techniques.

---

### Q1. What are the main components of a package-level PDN?

**Answer:**
The package-level PDN consists of several interconnected physical elements that form the power delivery path between the PCB BGA pads and the die bump pads.

**BGA (Ball Grid Array) pads and solder balls:** These are the connections between the package and the PCB. Power and ground BGA balls are distributed across the package footprint. Each ball has a resistance of approximately 1 to 5 mOhm and an inductance of 50 to 200 pH.

**Package substrate power/ground planes:** The package substrate is a multi-layer laminate (typically 4 to 12 layers for high-performance packages) with copper planes dedicated to power and ground distribution. These planes carry current laterally from the BGA balls to the via connections leading up to the die. The plane pairs also form a distributed capacitor (plane capacitance) that provides some decoupling, typically 1 to 10 nF depending on the plane area and dielectric thickness.

**Through-substrate vias:** Vertical vias connect the BGA-side planes to the die-side planes through the substrate. Each via has a resistance of 1 to 10 mOhm and an inductance of 10 to 50 pH. High-current rails use many vias in parallel.

**C4 bumps or microbumps:** These connect the package substrate to the die. As discussed in section 02, they have resistance of 5 to 30 mOhm and inductance of 20 to 50 pH each.

**Package-mounted decoupling capacitors (MLCCs):** Discrete ceramic capacitors soldered onto the package substrate (either on the top surface, bottom surface, or embedded within the substrate) provide decoupling in the frequency range of approximately 10 MHz to 500 MHz.

The total impedance of the package PDN is the series-parallel combination of all these elements. The spreading inductance of the package -- the inductance of the current loop from the die bumps through the substrate planes to the BGA balls -- is typically the most critical parameter, as it determines the impedance floor between the package decoupling and on-die decoupling frequency ranges.

---

### Q2. What is package plane resonance and why is it important?

**Answer:**
Package plane resonance occurs when the power and ground plane pair in the package substrate behaves as a resonant cavity. The plane pair forms a parallel-plate transmission line structure, and at frequencies where the plane dimensions are comparable to the wavelength of the electromagnetic wave in the dielectric, standing wave resonances develop.

The resonant frequencies of a rectangular cavity are given by:

```
f_mn = (c / (2 * sqrt(epsilon_r))) * sqrt((m/a)^2 + (n/b)^2)
```

where c is the speed of light, epsilon_r is the relative permittivity of the dielectric, a and b are the plane dimensions, and m and n are integer mode indices (m, n = 0, 1, 2, ..., with at least one nonzero).

For a typical package substrate with dimensions 30 mm x 30 mm and epsilon_r = 4:

```
f_10 = (3e8 / (2 * sqrt(4))) * sqrt((1/0.03)^2) = (1.5e8) * 33.3 = 5.0 GHz
```

The fundamental resonance at 5 GHz is generally above the frequency range of concern for PDN design. However, for larger packages (50 mm x 50 mm) or substrates with higher dielectric constant, the fundamental resonance can drop below 3 GHz, which may be within the operating frequency range.

More importantly, the anti-resonance between the package plane capacitance (acting as a distributed capacitor) and the package spreading inductance can occur at much lower frequencies (100 MHz to 1 GHz). At these anti-resonance frequencies, the impedance can spike significantly above the target.

Package plane resonance is mitigated by:
- Adding decoupling capacitors at locations that suppress the dominant resonant modes
- Using lossy dielectric materials that damp the resonance (higher loss tangent)
- Segmenting large planes with splits (though this must be done carefully to avoid creating inductance barriers)
- Adding resistive termination at plane edges

---

### Q3. How is the package spreading inductance determined and why is it critical?

**Answer:**
Package spreading inductance (also called loop inductance or partial self-inductance of the package) is the inductance of the current loop from the die bump pads through the package substrate planes and vias to the BGA balls. It is called "spreading" inductance because the current must spread laterally through the plane pair to reach the BGA connections.

The spreading inductance depends on:
- The distance between the die bump pads and the BGA pads (both lateral and vertical)
- The thickness of the dielectric between power and ground planes
- The number and location of vias connecting the planes
- The plane area and geometry

For a simple parallel-plate model, the inductance of a plane pair per unit area is:

```
L_per_area = mu_0 * d
```

where d is the dielectric thickness between power and ground planes and mu_0 is the permeability of free space. For d = 50 um:

```
L_per_area = 4*pi*1e-7 * 50e-6 = 62.8 pH/mm^2... 
```

Actually, the inductance per square is:

```
L_sheet = mu_0 * d = 4*pi*1e-7 * 50e-6 = 62.8 pH per square
```

The total spreading inductance from the die center to the package edge depends on the current distribution pattern and is typically computed using electromagnetic simulation tools (Sigrity PowerSI, ANSYS SIwave, Cadence Allegro).

Typical package spreading inductance values range from 10 to 100 pH for high-performance packages with many power vias and tight plane spacing, to 200 to 500 pH for lower-cost packages with fewer layers and wider plane spacing.

Spreading inductance is critical because it determines the impedance floor in the frequency range between package decoupling and on-die decoupling. The inductive impedance is:

```
Z_L = 2 * pi * f * L_spread
```

At 200 MHz with L_spread = 50 pH:

```
Z_L = 2 * pi * 200e6 * 50e-12 = 62.8 uOhm = 0.063 mOhm
```

This is comfortably below a 1 mOhm target. But at 1 GHz:

```
Z_L = 2 * pi * 1e9 * 50e-12 = 314 uOhm = 0.314 mOhm
```

Still below 1 mOhm but approaching it. For packages with higher spreading inductance (200 pH), the impedance at 1 GHz would be 1.26 mOhm, exceeding a 1 mOhm target.

---

### Q4. What are the different package types used for SoCs and their PDN implications?

**Answer:**
Different package types offer different PDN characteristics:

**Wire-bond BGA:** Traditional wire-bond packages connect the die to the substrate using bond wires from the die periphery to bond fingers on the substrate. The bond wires have high inductance (1 to 5 nH each) and moderate resistance (50 to 200 mOhm). Multiple wires in parallel reduce these values, but wire-bond packages inherently have much higher interconnect inductance than flip-chip packages. This limits their use for high-current, high-frequency SoCs.

**Flip-chip BGA (FC-BGA):** The die is flipped face-down and connected to the substrate through an array of C4 solder bumps across the entire die area. This provides hundreds to thousands of parallel power connections with very low total inductance and resistance. FC-BGA is the standard package for high-performance SoCs (processors, GPUs, networking chips).

**Flip-chip with microbumps:** Used with advanced packaging technologies (2.5D and 3D), microbumps are smaller than C4 bumps (typically 25 to 50 um pitch versus 100 to 200 um for C4). They provide higher connection density but each bump has higher resistance due to its smaller size.

**Fan-out wafer-level packaging (FOWLP):** The die is embedded in an epoxy mold compound and the package redistribution layers (RDL) are built directly on the wafer. FOWLP provides very thin packages with short interconnect paths but limited power and ground plane area, making it suitable for mobile SoCs with moderate power requirements.

**2.5D with silicon interposer:** Multiple dies are mounted on a silicon interposer that contains through-silicon vias (TSVs) and fine-pitch RDL. The interposer provides an intermediate PDN between the package substrate and the individual dies. Power TSVs in the interposer have very low resistance and inductance but consume silicon area.

| Package Type | Typical L_spread | Max Current | Best for |
|-------------|-----------------|-------------|----------|
| Wire-bond BGA | 1-10 nH | 5-15 A | Low-cost, low-power |
| FC-BGA | 20-200 pH | 50-300 A | High-performance |
| FOWLP | 50-500 pH | 5-30 A | Mobile, thin form factor |
| 2.5D interposer | 10-100 pH | 50-200 A | Multi-die, HPC |

---

### Q5. How are package power planes designed and how do plane splits affect PDN performance?

**Answer:**
Package power planes are continuous copper layers within the package substrate dedicated to carrying power or ground current. A well-designed package has one or more plane pairs (VDD plane adjacent to VSS plane, separated by a thin dielectric) that provide low-impedance current paths and built-in plane capacitance.

The plane design considers:

**Plane assignment:** Each voltage domain requires its own plane or plane region. A package with three supply rails (VCORE, VGPU, VIO) might have separate VDD planes for each, plus one or more continuous VSS (ground) planes. Ground planes are usually continuous across the entire package area to provide the lowest possible impedance return path.

**Plane splits:** When multiple voltage domains share the same layer, the plane must be split into separate regions for each domain. Plane splits create discontinuities in the current return path and can cause several problems: increased inductance for signals crossing the split (their return current must detour around the split), radiation and coupling between domains at the split edges, and reduced plane capacitance.

To minimize the impact of plane splits:
- Route power planes on inner layers where they are shielded by continuous ground planes on adjacent layers
- Avoid routing high-speed signals across plane splits
- Place decoupling capacitors near split edges to provide an alternative current return path
- Keep split gaps as narrow as the manufacturing process allows
- Use stitching capacitors to bridge splits for high-frequency return currents

**Copper thickness and coverage:** Package substrate copper layers are typically 12 to 35 um thick (0.5 to 1 oz equivalent). Thicker copper reduces resistance but increases manufacturing cost and limits fine-feature routing. The copper coverage (percentage of the layer occupied by copper) should be maximized for power planes while observing clearance rules to adjacent nets.

**Plane capacitance:** The capacitance between adjacent power and ground planes is:

```
C_plane = epsilon_0 * epsilon_r * A / d
```

where A is the overlap area and d is the dielectric thickness. For a 25 mm x 25 mm plane pair with epsilon_r = 4 and d = 50 um:

```
C_plane = 8.854e-12 * 4 * 625e-6 / 50e-6 = 443 pF
```

This 443 pF is modest but provides some decoupling at frequencies around the plane resonance (several GHz).

---

### Q6. How do you model a package PDN for simulation?

**Answer:**
Package PDN modeling can be done at various levels of complexity, depending on the analysis needs and computational resources.

**Lumped RLC model:** The simplest model represents the entire package as a single inductor (spreading inductance) in series with a resistor (total DC resistance), with a capacitor (plane capacitance) in parallel. This model is useful for quick hand calculations and early-stage exploration but does not capture frequency-dependent effects or spatial variations.

**Distributed lumped model:** The package is divided into a grid of cells, each represented by a local RLC circuit. Adjacent cells are connected by ladder networks of R and L elements. This captures spatial variation in impedance across the package and can model the effect of bump and via placement. Tools like SPICE can simulate these models.

**S-parameter model:** The package is characterized by its scattering parameters (S-parameters) extracted from a full-wave electromagnetic simulation. The S-parameter model accurately captures all frequency-dependent effects, including plane resonance, skin effect, and dielectric loss. It can be imported into circuit simulators for co-simulation with the die and PCB models. This is the most accurate approach for high-frequency analysis.

**Full-wave electromagnetic simulation:** Tools like Cadence Sigrity PowerSI, ANSYS SIwave, or ANSYS HFSS solve Maxwell's equations for the complete 3D package geometry. This provides the most accurate results but is computationally expensive. The output is typically an S-parameter model or an impedance matrix that can be used in circuit simulation.

The modeling flow typically proceeds as:
1. Import the package layout (substrate geometry, layer stackup, via locations)
2. Assign port locations at the BGA pads and die bump pads
3. Run electromagnetic extraction to generate S-parameters
4. Import the S-parameter model into a circuit simulator alongside the die and PCB models
5. Run AC impedance analysis to compute the PDN impedance at the die
6. Run transient analysis to verify time-domain voltage noise

---

### Q7. What is the role of keep-out zones in package PDN design?

**Answer:**
Keep-out zones are regions on the package substrate where certain physical features (traces, vias, components) are prohibited. They are defined for manufacturing and electrical reasons and have significant implications for PDN design.

**Component keep-out zones:** Areas around the die attach region, package edge, and existing components where additional decoupling capacitors cannot be placed. These zones limit the number and location of package-mounted MLCCs, reducing the available decoupling. Designers must prioritize capacitor placement in the remaining available space.

**Via keep-out zones:** Regions where vias cannot be drilled due to proximity to other features (BGA pads, component pads, or other vias). Via restrictions can limit the number of power vias between planes, increasing the via resistance and inductance.

**Routing keep-out zones:** Areas where signal traces cannot be routed, often above or below critical analog circuits or in regions reserved for power plane access. These zones can force signal routes to cross plane splits or take longer paths, affecting signal integrity.

**Die shadow region:** The area directly above or below the die is typically reserved for power/ground connections (BGA balls, vias, and potentially embedded capacitors). Signal routing in this region may be restricted to minimize noise coupling.

PDN design must work within these constraints. The impact of keep-out zones on decoupling capacitor placement is particularly important: if capacitors cannot be placed close to the die, their effectiveness is reduced because the additional inductance of the longer current path raises their effective ESL. Embedded capacitor technologies (thin dielectric layers within the substrate, or discrete capacitors embedded in cavities within the substrate) can partially address this by placing decoupling inside the substrate, closer to the die.

---

### Q8. How does the package PDN interact with the PCB PDN?

**Answer:**
The package PDN and PCB PDN are connected through the BGA solder joints and function as a cascaded network. Their interaction determines the impedance profile in the mid-frequency range (approximately 1 MHz to 100 MHz) and creates potential anti-resonance issues.

**Series connection:** The package inductance (spreading inductance from BGA to die bumps) is in series with the PCB plane inductance and decoupling network. From the die's perspective, the total impedance is the sum of the package impedance and the PCB impedance (at frequencies where neither the package decaps nor the PCB decaps are effective).

**Anti-resonance between PCB capacitors and package inductance:** The PCB-mounted decoupling capacitors provide low impedance up to their inductive crossover frequency. Above that, the PCB capacitors become inductive, and this inductance (plus the BGA ball inductance) resonates with the package plane capacitance and package-mounted capacitors. This anti-resonance can create an impedance spike in the 50 to 200 MHz range.

**Anti-resonance between package capacitors and die capacitance:** Similarly, the package MLCCs become inductive above their resonant frequency, and this inductance resonates with the on-die decoupling capacitance, creating an impedance peak in the 200 MHz to 1 GHz range.

**Co-design importance:** The package and PCB PDN must be designed together (co-designed) to ensure that the impedance profile across all frequency handoffs remains below the target. This requires:
- Sharing S-parameter models between the package and PCB design teams
- Running chip-package-board co-simulation to verify the combined impedance profile
- Selecting capacitor values and quantities that minimize anti-resonance peaks at the handoff frequencies
- Adding damping (through capacitor ESR or intentional resistors) at problematic frequencies

In practice, the package and PCB are often designed by different teams or even different companies (the SoC vendor designs the die and package, the system integrator designs the PCB). Clear PDN specifications and model exchange are essential for successful co-design.

---

### Q9. What is the impact of bump pitch on package PDN performance?

**Answer:**
Bump pitch (the center-to-center distance between adjacent bumps) directly affects the number of bumps available, the current per bump, and the inductance of the bump array.

**More bumps at finer pitch:** A smaller bump pitch allows more bumps in the same die area. For a die area of 100 mm^2, the number of bumps at different pitches:

| Pitch | Bumps per edge (10 mm) | Total bumps | VDD bumps (40%) |
|-------|----------------------|-------------|-----------------|
| 200 um | 50 | 2500 | 1000 |
| 150 um | 67 | 4489 | 1796 |
| 100 um | 100 | 10000 | 4000 |
| 50 um | 200 | 40000 | 16000 |

**Lower current per bump:** More bumps sharing the same total current means lower current per bump, reducing EM stress and allowing higher total current capacity.

**Lower total inductance:** The inductance of N bumps in parallel scales as L_single/N. More bumps dramatically reduce the total bump array inductance, lowering the high-frequency impedance between the package and die.

**Lower total resistance:** Similarly, total bump resistance scales as R_single/N, reducing the DC IR drop contribution from bumps.

**Tradeoffs:** Finer bump pitch requires more advanced packaging technology (microbumps at 50 um pitch require different assembly processes than C4 bumps at 150 um pitch), increases package substrate routing density requirements, and may require more package substrate layers. The cost increases significantly with finer pitch.

The trend in high-performance SoCs is toward finer bump pitch (moving from 150-200 um C4 pitch to 100 um or even 50 um microbump pitch) driven by both power delivery and signal I/O density requirements. Each bump pitch reduction improves PDN performance but increases package cost.

---

### Q10. How are package PDN designs verified before manufacturing?

**Answer:**
Package PDN verification involves a multi-step process combining electromagnetic simulation, circuit simulation, and comparison against specifications.

**Step 1: Electromagnetic extraction.** The complete package substrate layout (all metal layers, vias, BGA pads, bump pads) is imported into an EM solver (Sigrity PowerSI, ANSYS SIwave). The solver extracts the frequency-dependent impedance matrix or S-parameters for the package, capturing all plane effects, via parasitics, and distributed effects.

**Step 2: DC IR drop analysis.** A DC analysis (Sigrity PowerDC or equivalent) computes the static voltage drop from each BGA power ball to each die bump pad under the specified current load. This verifies that the total package DC resistance is within budget and identifies any current crowding at vias or narrow plane sections.

**Step 3: AC impedance analysis.** The extracted S-parameter model is used to compute the self-impedance at each die bump pad port, which represents the impedance the die sees looking into the package. This is plotted as a function of frequency and compared to the target impedance.

**Step 4: Co-simulation.** The package model is combined with a PCB model (from the PCB designer) and a die model (from the chip designer) in a circuit simulator. The combined impedance profile and time-domain transient response are analyzed to verify the complete PDN meets specifications.

**Step 5: Decoupling capacitor optimization.** If the impedance profile shows peaks above the target, the number, values, and locations of package-mounted MLCCs are adjusted. The EM extraction and impedance analysis are re-run iteratively until the target is met.

**Step 6: Sensitivity analysis.** Key parameters (dielectric constant, conductor resistivity, capacitor tolerance) are varied to ensure the design is robust across manufacturing variation.

**Step 7: Correlation with silicon.** After the first prototypes are assembled, the actual PDN impedance is measured (typically using a VNA connected to test points or through on-die impedance sensing) and compared to the simulated results. Discrepancies are used to improve the modeling methodology for future designs.
