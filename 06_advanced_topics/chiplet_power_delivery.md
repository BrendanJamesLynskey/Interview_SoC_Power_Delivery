# Chiplet Power Delivery

This section covers power delivery challenges and solutions for chiplet-based SoC architectures, including per-chiplet regulation, interposer PDN design, and power TSVs.

---

### Q1. What is a chiplet architecture and how does it affect power delivery?

**Answer:**
A chiplet architecture decomposes a monolithic SoC into multiple smaller dies (chiplets) that are interconnected within a single package. Each chiplet implements a specific function (CPU cores, GPU, I/O, memory) and may be fabricated on different process nodes. The chiplets are connected through an interposer (silicon, organic, or glass) or advanced packaging technology (EMIB, InFO).

Power delivery implications: instead of delivering power to a single monolithic die, the PDN must now deliver power to multiple separate dies, each with its own voltage requirements, current demands, and physical connection to the package. This introduces new challenges: the power must traverse additional interfaces (package to interposer, interposer to chiplet), each adding resistance and inductance; different chiplets may require different supply voltages; the total current and power may be higher than a monolithic die; and the thermal distribution is different (multiple heat sources, potentially non-uniform).

The benefits include: each chiplet is smaller, so the on-die power grid within each chiplet has shorter paths and lower resistance; different chiplets can use different process nodes, optimizing each for its function; and the package-level PDN can be designed with more flexibility in bump and via allocation.

---

### Q2. How are power TSVs (Through-Silicon Vias) used in chiplet power delivery?

**Answer:**
Power TSVs are vertical electrical connections through a silicon interposer or through a stacked die that carry power and ground current. They are a critical element in 2.5D and 3D chiplet power delivery.

**Physical characteristics:** A typical power TSV has a diameter of 5 to 10 um, a height (silicon thickness) of 50 to 100 um, and is filled or lined with copper. The resistance of a single TSV is approximately:

```
R_TSV = rho_Cu * h / (pi * r^2) [for filled TSV]
```

For a 10 um diameter, 100 um deep copper-filled TSV:

```
R_TSV = 1.72e-8 * 100e-6 / (pi * (5e-6)^2) = 1.72e-12 / 7.85e-11 = 21.9 mOhm
```

The inductance is approximately 10 to 30 pH per TSV.

**Quantity needed:** For a chiplet drawing 20 A with a TSV resistance of 22 mOhm each and a target total TSV resistance of 0.1 mOhm: N = 22/0.1 = 220 VDD TSVs. With equal VSS TSVs: 440 power TSVs total per chiplet.

**Area overhead:** Each TSV requires a keep-out zone (typically 5 to 10 um radius around the TSV) where no active transistors can be placed (due to stress effects from the TSV process). For a 10 um TSV with 10 um keep-out: the exclusion area per TSV is pi*(20e-6)^2 = 1257 um^2. For 440 TSVs: total exclusion area = 0.55 mm^2. On a 50 mm^2 interposer or chiplet, this is about 1%.

**TSV placement:** Power TSVs should be distributed uniformly under each chiplet to provide uniform current injection into the chiplet's power grid. Clustering all power TSVs at the chiplet edge creates longer on-die current paths and higher IR drop.

---

### Q3. How does the silicon interposer PDN work in a 2.5D architecture?

**Answer:**
In a 2.5D architecture, multiple chiplets are mounted on a silicon interposer, which is in turn mounted on a package substrate. The interposer contains redistribution layers (RDL) and TSVs that route power (and signals) between the package substrate and the chiplets.

**Power delivery path:** Power flows from the package substrate BGA balls, through the package vias to the interposer microbump pads on the bottom of the interposer, through the interposer TSVs and RDL layers, to the microbump pads on the top of the interposer, and finally through the chiplet microbumps into the chiplet die.

**Interposer PDN elements:**

RDL power planes: The interposer has 2 to 4 metal layers of RDL (typically 1 to 3 um thick copper) that distribute power laterally across the interposer. These planes connect the TSV array to the chiplet microbump array. The sheet resistance is higher than package substrate planes (due to thinner metal), but the distances are shorter.

TSVs: Carry power vertically through the 50 to 100 um silicon interposer. Hundreds to thousands of power TSVs per chiplet.

Interposer decoupling: MIM (metal-insulator-metal) capacitors can be fabricated on the interposer to provide decoupling between the package and chiplet levels. Silicon trench capacitors can also be integrated for higher density.

**Interposer PDN resistance:** The total resistance from the package microbump to the chiplet microbump includes: interposer bottom microbump (15 mOhm per bump, ~100 bumps: 0.15 mOhm), interposer RDL path (0.1 to 0.5 mOhm), TSV resistance (440 TSVs at 22 mOhm: 0.05 mOhm), interposer top microbump (15 mOhm per bump, ~100 bumps: 0.15 mOhm). Total: approximately 0.5 to 1.0 mOhm.

This additional resistance in the delivery path must be added to the package and PCB resistance when computing the total PDN DC resistance. For high-current chiplets, the interposer contribution can be significant.

---

### Q4. What are the challenges of per-chiplet voltage regulation?

**Answer:**
In a multi-chiplet system, each chiplet may operate at a different voltage for DVFS purposes. Providing independent voltage regulation to each chiplet introduces several challenges.

**Number of regulators:** A system with 4 compute chiplets, 2 I/O chiplets, and a memory chiplet may need 7 or more independent voltage regulators. Each requires its own control loop, feedback sensing, and power delivery path. This increases system complexity, board area, and cost.

**Voltage domain isolation:** Each chiplet's supply rail must be isolated from other chiplets' rails to prevent noise coupling and enable independent DVFS. The interposer must route separate power planes for each domain, consuming routing resources and potentially requiring more interposer metal layers.

**Current sharing for identical chiplets:** If multiple identical compute chiplets share a workload, they may benefit from operating at the same voltage and sharing a single regulator (reducing cost). However, if process variation causes one chiplet to be faster than another, a shared voltage must be set for the slowest chiplet, wasting power on the faster ones.

**On-interposer IVR:** One solution is to integrate voltage regulators on the silicon interposer. The interposer has more area than the chiplets and can accommodate LDOs or switched-capacitor converters powered from a shared input supply. Each chiplet gets its own on-interposer regulator, providing independent voltage with short delivery paths. The challenge is designing efficient regulators on the interposer process (which may not be optimized for analog).

**Hybrid approach:** A common architecture uses a small number of off-chip regulators (one per major voltage level) feeding a shared input supply through the package and interposer, with per-chiplet LDOs (either on the chiplet die or on the interposer) providing fine-grained voltage regulation and DVFS.

---

### Q5. How does chiplet power delivery differ from monolithic SoC power delivery?

**Answer:**
Several fundamental differences affect the PDN design:

| Aspect | Monolithic SoC | Chiplet Architecture |
|--------|---------------|---------------------|
| Die-to-package interface | Single bump array | Multiple bump arrays (one per chiplet + interposer) |
| On-die grid | Single large grid | Multiple smaller grids (one per chiplet) |
| Total bump count | Limited by die area | Can be larger (sum of all chiplets) |
| Package substrate | Routes to one die | Routes to interposer (simpler) or multiple dies |
| Voltage domains | 10-30 on one die | Distributed across chiplets |
| IR drop path | Bumps -> die grid | Bumps -> interposer -> TSV -> chiplet grid |
| Additional interfaces | None | Interposer bumps, TSVs, chiplet bumps |
| Decoupling | On-die only | On-chiplet + on-interposer + on-package |
| Thermal | Single hotspot | Multiple hotspots, potentially uneven |

The additional interfaces in a chiplet architecture add resistance and inductance to the PDN. For a 2.5D architecture with a silicon interposer, the interposer adds approximately 0.5 to 1.0 mOhm to the total path resistance. This is modest but must be budgeted.

The advantage is that each chiplet's on-die grid is smaller and simpler than a full monolithic SoC grid. A compute chiplet that is 100 mm^2 (versus a 400 mm^2 monolithic die) has shorter worst-case current paths, fewer grid routing conflicts, and simpler EM management.

---

### Q6. What is the role of decoupling capacitors on the interposer?

**Answer:**
Interposer decoupling capacitors fill the frequency gap between package-level decoupling and chiplet on-die decoupling, similar to how package MLCCs fill the gap between PCB and die decoupling in a conventional architecture.

**Why interposer decoupling is needed:** The TSVs and interposer metal layers add inductance between the package substrate and the chiplet die. Without decoupling on the interposer, this inductance creates an anti-resonance between the package capacitors (which become inductive at the handoff frequency) and the chiplet on-die capacitors. Interposer decoupling bridges this gap.

**Types of interposer decoupling:**

MIM capacitors: Thin dielectric layers (SiN, SiO2, high-k) sandwiched between metal layers on the interposer. Capacitance density of 1 to 20 fF/um^2, providing 1 to 20 nF over a 1 mm^2 area. Very low ESL (single-digit pH) due to the planar structure.

Deep-trench capacitors: High-aspect-ratio trenches etched into the silicon and filled with dielectric and conductor, creating large surface area capacitors. Capacitance density of 100 to 2000 nF/mm^2. This technology (borrowed from DRAM) provides the highest capacitance density for silicon-based decoupling.

Discrete silicon capacitor dies: Small companion dies containing deep-trench capacitors, mounted on the interposer alongside the chiplets. These provide very high capacitance (100+ nF) in a small footprint.

**Effective frequency range:** Interposer decoupling is most effective in the 100 MHz to 2 GHz range, bridging from the package MLCC range (~50-300 MHz) to the chiplet on-die decap range (~500 MHz and above).

---

### Q7. How do 3D stacked architectures affect power delivery?

**Answer:**
3D stacking places multiple active dies vertically, bonded through TSVs or hybrid bonding. This creates unique power delivery challenges because each layer needs power, and the power must traverse all layers below it.

**Cascaded resistance:** In a 3D stack with the bottom die (closest to the package) as Layer 1 and the top die as Layer N, the power to Layer N must pass through the TSVs of all intermediate layers. Each layer adds its TSV resistance to the path. For a 4-layer stack with 0.1 mOhm TSV resistance per layer, the top layer sees 0.4 mOhm just from TSVs.

**Thermal stacking:** Each layer generates heat, and the layers above the bottom are thermally insulated by the silicon and bonding layers below them. The top die in a 3D stack can be 20 to 40 degrees hotter than the bottom die, significantly increasing leakage power and EM stress in the upper layers.

**Current redistribution:** The bottom die carries not only its own current but also the current for all dies above it (which pass through the bottom die's TSVs and power grid). The bottom die's power grid must be designed to handle this transit current without excessive IR drop.

**Per-layer decoupling:** Each die in the stack provides its own on-die decoupling for high-frequency noise. However, the effective loop inductance for charge transfer between adjacent layers (through TSVs) is very small (a few pH for short TSVs), which means adjacent layers can share decoupling effectively at high frequencies.

**Power delivery options:**
- All power enters from the bottom die through the package (conventional)
- Power enters from both the top and bottom (if the top die has backside power delivery)
- Through-mold vias provide supplementary power connections from the package directly to upper layers

---

### Q8. What is backside power delivery and how does it improve chiplet PDN?

**Answer:**
Backside power delivery (BSPD) is an emerging technology where power and ground connections are made to the backside (non-active side) of the die, rather than through the front-side bumps. This separates the power delivery path from the signal path, potentially improving both.

**Conventional front-side delivery:** Power bumps and signal bumps share the same die surface. Power must traverse the entire BEOL metal stack (from top metal down to M1) to reach the transistors. This creates competition for bump real estate and forces power to share the same metal layers as signals.

**Backside power delivery:** Power is delivered through TSVs or nano-TSVs from the wafer backside directly to the transistor level (buried power rails at M0 or below M1). This provides a direct, short path from the backside to the transistors without passing through the BEOL.

**Benefits:**
- More bumps available for signals (or more bumps for power, or both)
- Shorter power delivery path (backside to buried power rail is ~10 um, versus ~10-20 um through the BEOL stack)
- Lower IR drop (shorter path, potentially thicker backside metals)
- Reduced congestion in BEOL (power and signal routing do not compete)
- Potential for higher logic density (buried power rails remove M1 power tracks from the routing layers)

**Challenges:**
- Requires wafer thinning (to 5-10 um for nano-TSV) and backside processing
- New manufacturing steps (backside RDL, backside bonding)
- Thermal management complexity (heat now exits from both sides)
- Still in early production or development phase (Intel, TSMC, Samsung have demonstrated prototypes)

For chiplet architectures, BSPD enables each chiplet to receive power from its backside while using the front side exclusively for chiplet-to-chiplet and chiplet-to-interposer signal connections, simplifying the interposer design.

---

### Q9. How do you model the PDN of a multi-chiplet system?

**Answer:**
Modeling a multi-chiplet PDN requires a hierarchical approach that captures each stage of the delivery path and their interactions.

**Step 1: Model each chiplet individually.** Extract the on-die power grid of each chiplet using the power integrity tool (RedHawk, Voltus) to obtain an equivalent impedance model at the chiplet's bump pads. This can be a SPICE netlist or an impedance matrix.

**Step 2: Model the interposer.** Extract the interposer PDN (TSVs, RDL planes, interposer decoupling) using an EM solver. The result is an S-parameter model with ports at each chiplet's microbump pads and at the package-side microbump pads.

**Step 3: Model the package.** Extract the package substrate PDN (planes, vias, MLCCs) similarly, with ports at the interposer-side pads and the BGA pads.

**Step 4: Model the PCB and VRM.** As in conventional PDN modeling.

**Step 5: Assemble and simulate.** Connect all sub-models in a circuit simulator: VRM -> PCB -> package -> interposer -> chiplets. Run AC impedance analysis at each chiplet's die port and transient analysis with realistic current waveforms for each chiplet.

**Challenges specific to multi-chiplet modeling:**

Large model size: Multiple chiplets and a large interposer result in very large combined models with many ports.

Coupling between chiplets: Current transients in one chiplet affect the supply voltage of neighboring chiplets through the shared interposer and package PDN. The model must capture this inter-chiplet coupling.

Multiple voltage domains: If chiplets operate at different voltages, the model must include the separate power planes for each domain and any shared ground paths.

Thermal coupling: Different chiplets operate at different temperatures, affecting the resistance of the shared PDN elements (interposer RDL, TSVs). A thermally-aware model uses position-dependent resistance.

---

### Q10. What are the emerging trends in chiplet power delivery?

**Answer:**
Several technology trends are shaping the future of chiplet power delivery:

**Integrated voltage regulation on interposers:** Fabricating LDOs or SC converters on the silicon interposer provides per-chiplet regulation without consuming chiplet die area. This leverages the interposer silicon for an active function beyond passive redistribution.

**Backside power delivery for chiplets:** As discussed in Q8, BSPD enables each chiplet to receive power from its backside, freeing the front side for high-density chiplet-to-chiplet interconnects.

**High-density silicon capacitors:** Deep-trench capacitors on the interposer or on dedicated companion dies provide ultra-high decoupling density (>1000 nF/mm^2) with very low ESL, dramatically improving the high-frequency PDN impedance.

**Direct liquid cooling integration:** For high-power chiplet systems (data center GPUs, AI accelerators), direct liquid cooling channels can be integrated into the package or interposer. This enables higher power densities by removing heat more efficiently, relaxing the thermal constraints on the PDN.

**Heterogeneous integration of passive components:** Embedding inductors, capacitors, and resistors into the package substrate alongside the chiplets. Glass interposers (instead of silicon) offer lower cost and better passive component characteristics (higher-Q inductors, lower-loss transmission lines).

**Photonic power delivery:** A research concept where laser light is transmitted through optical fibers or waveguides to the package, where photovoltaic cells convert it to electrical power. This eliminates the resistive delivery path entirely but is far from practical implementation.

**Standardized chiplet interfaces:** Industry efforts (UCIe -- Universal Chiplet Interconnect Express) to standardize chiplet interfaces will also standardize PDN requirements (voltage levels, current limits, bump patterns), enabling interoperable chiplets from different vendors with compatible power delivery specifications.
