# PCB Power Planes

This section covers PCB power plane design for SoC power delivery, including stackup considerations, copper weight selection, via stitching, and plane split management.

---

### Q1. How are power and ground planes arranged in a PCB stackup for SoC power delivery?

**Answer:**
The PCB stackup determines the arrangement of copper layers and dielectric materials that form the power delivery path from the VRM to the BGA footprint. A well-designed stackup places power and ground planes in close proximity to minimize the loop inductance and maximize the distributed plane capacitance.

A typical high-performance SoC board might use a 12 to 16 layer stackup. The general principles are: ground planes should be continuous and placed adjacent to signal layers to provide clean reference planes; power/ground plane pairs should be placed on adjacent layers with the thinnest available dielectric between them to maximize plane capacitance; signal layers should be sandwiched between reference planes for controlled impedance.

An example 12-layer stackup for a server-class SoC:
- Layer 1: Top signal + component pads
- Layer 2: Ground plane (GND)
- Layer 3: Signal routing
- Layer 4: VCORE power plane
- Layer 5: Ground plane (GND)
- Layer 6: Signal routing
- Layer 7: Signal routing
- Layer 8: Ground plane (GND)
- Layer 9: VGPU power plane
- Layer 10: Signal routing
- Layer 11: Ground plane (GND)
- Layer 12: Bottom signal + component pads

The VCORE power plane (layer 4) is adjacent to a ground plane (layer 5) with a thin dielectric (2 to 4 mil / 50 to 100 um), forming a low-inductance power/ground plane pair with useful distributed capacitance. Multiple ground planes ensure that every signal layer has an adjacent reference plane.

The key tradeoff is layer count versus performance: more layers allow more dedicated power planes and better shielding, but increase PCB cost. For cost-sensitive designs, a 6 or 8-layer stackup may be used with shared or split power planes.

---

### Q2. How does copper weight affect PCB power delivery performance?

**Answer:**
Copper weight (or copper thickness) directly determines the sheet resistance of the power planes and the current carrying capacity of power traces. Standard copper weights for PCB inner layers are 0.5 oz (17.5 um), 1 oz (35 um), and 2 oz (70 um). For power-intensive designs, heavy copper (3 oz to 6 oz, or 105 to 210 um) is available but adds cost and manufacturing complexity.

The sheet resistance of a copper plane is:

```
Rsh = rho / T
```

For copper (rho = 1.72 uOhm-cm):
- 0.5 oz: Rsh = 1.72e-6 / 17.5e-4 = 0.98 mOhm/sq
- 1 oz: Rsh = 0.49 mOhm/sq
- 2 oz: Rsh = 0.245 mOhm/sq

The plane resistance between the VRM output and the BGA footprint depends on the geometry and the number of effective squares. For a typical SoC board where the VRM is 30 mm from the BGA center, the plane resistance might be:

- 0.5 oz: ~2 mOhm
- 1 oz: ~1 mOhm  
- 2 oz: ~0.5 mOhm

For a 50 A VCORE rail, the IR drop through the PCB planes is:
- 0.5 oz: 50 * 2e-3 = 100 mV (unacceptable)
- 1 oz: 50 * 1e-3 = 50 mV (marginal)
- 2 oz: 50 * 0.5e-3 = 25 mV (better but still significant)

This shows why high-current SoC designs often use 2 oz or heavier copper for power planes. The IR drop through the PCB planes is a significant fraction of the total voltage budget and must be minimized.

Heavier copper also improves thermal performance because the increased copper mass helps spread and dissipate heat. However, it affects the etching resolution (wider trace-to-trace spacing required for thicker copper), impedance control of adjacent signal layers, and overall board thickness.

---

### Q3. What is via stitching and why is it important for power planes?

**Answer:**
Via stitching is the practice of placing an array of vias connecting ground planes on different layers of the PCB. These stitching vias serve multiple purposes in power delivery and signal integrity.

**Low-impedance ground connection:** Multiple ground planes on different layers must be connected with low impedance to ensure they are at the same potential. Without stitching vias, the only connections between ground planes are through component ground pads, which may be sparse. Stitching vias provide many additional parallel connections, reducing the inter-plane resistance and inductance.

**Return current path:** High-speed signals transitioning between layers need a nearby via for the return current to follow. If a signal via passes through a ground plane, the return current must find a path to the next reference plane. A nearby stitching via provides this path with minimal loop area, preserving signal integrity and reducing EMI.

**Plane cavity mode suppression:** Large uninterrupted plane cavities can support resonant modes that amplify noise at certain frequencies. Stitching vias break up the cavity into smaller sub-cavities with higher resonant frequencies, effectively suppressing low-frequency resonance.

**Design guidelines for via stitching:**
- Place stitching vias on a regular grid with spacing no larger than lambda/10 at the highest frequency of concern. At 5 GHz (lambda ~ 30 mm in FR4), the spacing should be less than 3 mm.
- Place stitching vias near every signal via that transitions between reference planes.
- Place stitching vias along the edges of power plane splits to provide return current paths at the split boundaries.
- Place stitching vias around the BGA footprint perimeter to create a low-impedance ground cage.

Via stitching adds to the board cost (more drill operations) and consumes routing space, but the improvements in power integrity, signal integrity, and EMI performance are substantial.

---

### Q4. What problems do power plane splits cause and how should they be managed?

**Answer:**
A power plane split occurs when a single copper layer is divided into separate regions for different voltage domains (e.g., VCORE and VGPU on the same layer). The gap between regions is typically 10 to 20 mils wide.

**Problems caused by splits:**

Signal return current disruption: When a signal trace on an adjacent layer crosses over a plane split, the return current cannot flow directly underneath the trace at the split location. The return current must detour around the split edge, creating a larger current loop. This increases inductance, degrades impedance matching, causes signal reflections, and increases electromagnetic radiation.

Increased power plane inductance: The split forces current to flow around the gap, increasing the effective path length and inductance for power delivery between separated regions.

Electromagnetic coupling: The split gap acts as a slot antenna that can radiate electromagnetic energy, causing EMI problems. It also allows noise coupling between the two voltage domains through capacitive and inductive coupling across the gap.

**Management strategies:**

Route signals on layers that have an unsplit reference plane (ground) on an adjacent layer. Never route high-speed signals across a power plane split unless a continuous ground plane is immediately adjacent.

Use bridge capacitors across the split to provide a high-frequency return current path. Place 100 nF MLCCs at regular intervals (every 5 to 10 mm) across the split boundary to bridge the gap for high-frequency return currents.

Place separate voltage domains on different layers rather than splitting a single layer. Use two inner power layers (one for VCORE, one for VGPU) each with its own adjacent ground plane.

If splits are unavoidable, orient the split so that it runs parallel to (not perpendicular to) the primary signal routing direction, minimizing the number of signals that cross it.

Ensure the ground planes above and below the split power plane are continuous, providing alternative return current paths through the stitching vias.

---

### Q5. How is the PCB PDN modeled for simulation?

**Answer:**
PCB PDN modeling involves characterizing the electrical behavior of the power planes, decoupling capacitors, VRM output, and interconnects, then combining them into a simulation-ready model.

**Plane modeling:** The PCB power/ground plane pair is modeled using either a distributed 2D mesh of RLC elements or a full-wave electromagnetic solver. Tools like Cadence Sigrity PowerSI or ANSYS SIwave import the PCB layout (Gerber files or ODB++ format), extract the plane geometry, and compute the frequency-dependent impedance as S-parameters. The model captures plane spreading resistance, distributed inductance, plane capacitance, and resonant modes.

**Decoupling capacitor modeling:** Each decoupling capacitor is modeled as a series RLC circuit (or a more complex model from the manufacturer's SPICE model). The mounting parasitics (PCB pad, via to plane, trace to component) are included as additional series inductance, typically 0.3 to 1.5 nH depending on the via configuration.

**VRM modeling:** The VRM output is modeled as a voltage source in series with an output impedance. The output impedance is frequency-dependent: at low frequencies (within the control loop bandwidth), it is very low; above the bandwidth, it becomes the impedance of the output capacitor bank. Many VRM manufacturers provide SPICE models of their output impedance.

**BGA ball model:** The BGA solder joints are modeled as RLC elements connecting the PCB planes to the package model.

**Combined simulation:** All these sub-models are assembled in a circuit simulator (HSPICE, Spectre, or within the PDN analysis tool). AC analysis computes the impedance profile, and transient analysis computes the time-domain voltage response to load current steps.

The simulation must include the complete current loop: VDD from the VRM through the PCB planes to the BGA ball, and VSS from the BGA ball back through the ground planes to the VRM. Both the VDD and VSS path impedances contribute to the total PDN impedance.

---

### Q6. What is the effect of PCB dielectric material on power delivery performance?

**Answer:**
The PCB dielectric material affects power delivery through its dielectric constant, loss tangent, and thickness tolerance.

**Dielectric constant (Dk):** The capacitance between power and ground planes is proportional to Dk. Standard FR4 has Dk around 4.0 to 4.5, while low-loss materials (Megtron, IS680) have Dk of 3.2 to 3.8. Higher Dk provides more plane capacitance (beneficial for decoupling) but also reduces signal propagation velocity (affecting signal integrity). For power delivery, higher Dk is slightly beneficial but the effect is small compared to discrete decoupling capacitors.

**Loss tangent (Df):** The dielectric loss provides natural damping of plane resonances. Standard FR4 has Df of 0.02 to 0.03, while low-loss materials have Df below 0.005. For power delivery, higher loss actually helps by damping resonance peaks, but for signal integrity, lower loss is preferred. This creates a tradeoff when selecting materials for boards that must serve both functions.

**Dielectric thickness:** Thinner dielectric between power/ground planes increases plane capacitance (C is inversely proportional to thickness) and reduces spreading inductance (L is proportional to thickness). A 2 mil (50 um) dielectric provides 2x the capacitance and half the inductance of a 4 mil (100 um) dielectric. However, very thin dielectrics increase manufacturing difficulty and may reduce voltage breakdown margin.

**Thermal properties:** The glass transition temperature (Tg) and decomposition temperature (Td) of the dielectric determine the maximum operating and assembly temperature. Standard FR4 (Tg = 130-140C) is adequate for most applications, while high-Tg FR4 (Tg = 170-180C) or polyimide is used for high-temperature applications.

For most SoC board designs, standard high-Tg FR4 is adequate for power delivery. The choice of premium materials (low-loss, controlled-Dk) is driven more by signal integrity requirements for high-speed interfaces (DDR5, PCIe Gen5, 112G SerDes) than by power delivery needs.

---

### Q7. How is current distributed in a PCB power plane?

**Answer:**
Current distribution in a PCB power plane is not uniform -- it follows the path of least impedance from the source (VRM output) to the load (BGA pads), concentrating in some areas and being sparse in others.

At DC and low frequencies, the current distribution is determined by the plane resistance. Current follows the path of least resistance, which means it tends to spread broadly through the plane but concentrates near the shortest path between source and load. Constrictions (narrow necks where the plane is pinched between cuts, vias, or board edges) create current crowding with high local current density.

At higher frequencies, the current distribution is determined by both resistance and inductance. Current tends to flow in the area directly between the VDD plane and the adjacent GND plane (the return current in the GND plane mirrors the VDD current distribution above/below it). The skin effect also plays a role at high frequencies, confining the current to a thin layer on the plane surfaces.

**Factors that affect current distribution:**

Plane geometry: Cutouts, splits, and irregular plane shapes force current to flow around obstacles, creating non-uniform distribution.

Decoupling capacitor placement: Capacitors act as local current sources at high frequencies. Current flows from the nearest capacitor to the nearest load bump, so the distribution depends on the relative positions of caps and BGA balls.

VRM location: The VRM placement determines where current enters the plane. A VRM located far from the BGA footprint requires current to traverse more plane area, increasing resistance. Placing the VRM close to the BGA (ideally adjacent to it) minimizes the plane path length.

Multiple VRM phases: In a multiphase VRM, each phase inductor connects to the plane at a different point. The current from each phase enters the plane and spreads toward the BGA pads. Distributing the phase inductors around the BGA footprint provides more uniform current injection.

DC IR drop simulation tools (Sigrity PowerDC, ANSYS SIwave DC) visualize the current density distribution across the plane, allowing designers to identify hotspots and constrictions that need attention.

---

### Q8. What is the optimal VRM-to-BGA placement strategy on the PCB?

**Answer:**
The physical distance and routing between the VRM output and the SoC BGA footprint directly impacts the PCB PDN impedance. The optimal placement strategy minimizes this path impedance.

**Close placement:** The VRM should be placed as close as possible to the BGA footprint of the SoC. Ideally, the VRM output inductor pads are adjacent to the BGA footprint, with the power plane path being only a few millimeters. This minimizes plane resistance and inductance, and reduces the I-squared-R power loss in the PCB.

**Surrounding placement:** For multiphase regulators, distributing the output inductors on multiple sides of the BGA footprint provides more uniform current injection into the plane. A common configuration places inductors on two or three sides of the BGA, with the controller IC nearby.

**Decoupling capacitor placement:** Bulk and ceramic decoupling capacitors should be placed between the VRM output and the BGA pads, as close to the BGA as possible. The capacitors on the back side of the board (directly under the BGA footprint) are particularly effective because they have short via connections to the power planes.

**Back-side capacitors:** For BGAs with many power balls, placing decoupling MLCCs on the bottom side of the PCB (directly under the BGA footprint, in the spaces between BGA balls) provides the shortest possible current path from capacitor to ball. This requires the BGA ball pitch to be large enough (typically 0.8 mm or larger) to accommodate the MLCC footprint between balls. This technique can reduce mounting inductance from 1.5 nH to below 0.5 nH.

**Thermal considerations:** The VRM dissipates significant power (especially at high current) and generates heat. Placing the VRM very close to the SoC (which is also a major heat source) may create thermal management challenges. Some spacing may be needed for airflow and heat spreading.

**Routing constraints:** The BGA escape routing (signal traces fanning out from the BGA pads to the wider board area) may require specific areas around the BGA footprint. The VRM placement must not block critical signal escape routes.

---

### Q9. How do you determine the number and value of PCB decoupling capacitors?

**Answer:**
The PCB decoupling capacitor network must provide low impedance in the frequency range from the VRM bandwidth (~100 kHz) up to where the package decoupling takes over (~10-50 MHz).

**Step 1: Identify the frequency range.** The PCB caps are responsible for approximately 100 kHz to 50 MHz. Below 100 kHz, the VRM regulates the voltage. Above 50 MHz, the package-mounted MLCCs provide decoupling.

**Step 2: Calculate the target impedance.** Using Ztarget = Vdd * ripple% / Imax. For Vdd = 0.85 V, 5% ripple, Imax = 50 A: Ztarget = 0.85 mOhm.

**Step 3: Select capacitor values.** Choose values whose series resonant frequencies span the target range. Typical PCB cap values and their approximate resonant frequencies (with 1 nH mounting ESL):

| Value | f_res (with 1 nH ESL) |
|-------|----------------------|
| 100 uF polymer | 0.5 MHz |
| 22 uF MLCC | 1.1 MHz |
| 10 uF MLCC | 1.6 MHz |
| 1 uF MLCC | 5.0 MHz |
| 100 nF MLCC | 15.9 MHz |
| 10 nF MLCC | 50.3 MHz |

**Step 4: Determine quantities.** For each value, the number of caps needed to bring the impedance at resonance below Ztarget:

N >= ESR / Ztarget

For bulk polymer caps (ESR ~ 5 mOhm): N >= 5/0.85 = 6.
For ceramic MLCCs (ESR ~ 3 mOhm): N >= 3/0.85 = 4 per value.

In practice, many more caps are used to ensure continuous frequency coverage and manage anti-resonance. A typical PCB decoupling network for a high-performance SoC might include:

- 6 x 100 uF polymer bulk caps
- 10 x 22 uF MLCCs (1206 size)
- 20 x 10 uF MLCCs (0402 size)
- 30 x 1 uF MLCCs (0402 size)
- 20 x 100 nF MLCCs (0201 size)

Total: approximately 86 capacitors on the PCB near the BGA footprint.

**Step 5: Verify with simulation.** The complete PCB PDN (planes + capacitors + VRM model) is simulated to verify the impedance profile meets the target across the entire frequency range. Anti-resonance peaks between capacitor groups are checked and mitigated by adjusting values and quantities.

---

### Q10. What is the role of ferrite beads in PCB power delivery?

**Answer:**
Ferrite beads are passive components that provide frequency-dependent impedance -- low impedance at DC and low frequencies, high impedance at RF frequencies. They are used in power delivery for noise isolation between voltage domains, not for primary power delivery to high-current SoC rails.

**Noise filtering:** Ferrite beads are placed in series with the power supply to low-current, noise-sensitive circuits (PLLs, ADCs, clock generators). The ferrite bead attenuates high-frequency noise on the supply while passing DC current with minimal voltage drop. A typical ferrite bead might have 1 ohm of impedance at 100 MHz and only 50 mOhm of DC resistance.

**Domain isolation:** When a sensitive analog circuit shares a power rail with noisy digital circuits, a ferrite bead can be placed between them to prevent digital switching noise from reaching the analog supply. The ferrite bead is followed by a local decoupling capacitor to form an LC filter.

**Not suitable for high-current rails:** Ferrite beads have significant DC resistance (tens to hundreds of milliohms) and limited current ratings (typically less than 1 A for common SMD ferrites). They are not used in the main power delivery path for high-current SoC rails (VCORE, VGPU) because the IR drop and power dissipation would be unacceptable.

**Saturation consideration:** Ferrite beads use magnetic core materials that saturate at high current. When saturated, the ferrite bead loses its high-frequency impedance and becomes a simple resistor. The component must be selected with a current rating that provides adequate margin against saturation under all operating conditions, including transient peaks.

**Alternative to ferrite beads:** For high-current noise isolation, an LC filter using a small inductor (10 to 100 nH) and capacitors can provide similar filtering with lower DCR. LDO post-regulators are another alternative that provides both voltage regulation and noise rejection.
