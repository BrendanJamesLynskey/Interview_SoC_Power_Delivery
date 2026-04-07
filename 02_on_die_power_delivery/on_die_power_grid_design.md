# On-Die Power Grid Design

This section covers the design of on-chip power distribution networks, including grid topologies, metal layer allocation, grid sizing methodology, and voltage uniformity considerations.

---

### Q1. What are the primary on-die power grid topologies and their tradeoffs?

**Answer:**
The two fundamental on-die power grid topologies are stripe-based and mesh-based, with most modern designs using a hybrid approach.

**Stripe topology:** Power (VDD) and ground (VSS) are distributed as parallel stripes running in one direction, typically on the upper metal layers. The stripes alternate between VDD and VSS. Current flows from the bump pads along these stripes and is tapped down through vias to the lower metal layers where the standard cells connect. Stripe-based grids are simple to design and route, and they provide low resistance along the stripe direction. However, they have high resistance in the perpendicular direction (current must flow through the lower-metal connections between stripes), leading to voltage non-uniformity for blocks not directly under a power bump. Stripe grids are common in older process nodes and in less performance-critical designs.

**Mesh topology:** Both VDD and VSS are distributed as orthogonal meshes, with horizontal stripes on one metal layer and vertical stripes on another, connected by vias at every intersection. This creates a two-dimensional grid that provides low-resistance paths in both directions. Current can flow from any bump pad to any load through multiple parallel paths, resulting in much better voltage uniformity and lower effective resistance. The penalty is that the mesh consumes routing resources on two metal layers instead of one, and the via connections at every grid intersection add manufacturing complexity.

**Hybrid approaches** are most common in practice. The uppermost thick metal layers (often called redistribution layers or RDL-like layers in advanced nodes) carry the primary power mesh, while thinner intermediate metal layers carry secondary stripes that connect the mesh down to the standard cell power rails on metal 1 (M1). This balances routing resource consumption with electrical performance.

The choice of topology depends on current density requirements, available metal layers, routing congestion, and the uniformity of current consumption across the die.

---

### Q2. How are metal layers allocated for power grid versus signal routing in a typical SoC?

**Answer:**
In a modern SoC with 10 to 15 metal layers, the allocation follows a general hierarchy from bottom to top:

**Metal 1 (M1):** This is the lowest metal layer and is used for standard cell internal connections and the local VDD/VSS rails that power each row of standard cells. These rails are typically narrow (a few hundred nanometers wide) and run horizontally along each cell row.

**Metal 2 through Metal 5 (lower metals):** These thin metal layers are primarily dedicated to signal routing -- intra-block connections, clock trees, and local interconnect. Occasionally, a portion of these layers is used for secondary power straps that connect the M1 rails up to the higher-level power grid, but this competes directly with signal routing resources.

**Metal 6 through Metal 8 (intermediate metals):** These layers, which are often intermediate in thickness, serve a dual purpose. They carry signal routing for longer-distance connections and also carry secondary power stripes that distribute current from the upper power mesh down to the lower layers. The fraction of each layer dedicated to power versus signals is a key design tradeoff.

**Metal 9 through Metal 12 or above (upper thick metals):** The top two to four metal layers are significantly thicker than the lower layers (often 2 to 5 times thicker, sometimes 10x for the top aluminum or copper RDL layer). These thick layers carry the primary power mesh because their low sheet resistance allows them to handle high currents without excessive IR drop. They also carry global signal buses and clock distribution, but power distribution is the primary consumer.

**Redistribution layer (RDL) or AP (aluminum pad) layer:** In some processes, the topmost layer is an extra-thick aluminum or copper layer used for bump pad connections and wide power distribution.

The exact allocation depends on the process technology, the power budget, and the routing congestion. A typical guideline is that 20 to 40 percent of the upper metal resources are dedicated to the power grid, with higher percentages needed for high-current designs. Power grid planning must be done early in the physical design flow to reserve adequate resources before signal routing begins.

---

### Q3. How do you size the on-die power grid stripes?

**Answer:**
Power grid stripe sizing is driven by two primary constraints: IR drop and electromigration (EM).

**IR drop constraint:** The total resistance from the nearest power bump to the farthest point in the design that draws current from that bump must be low enough that the voltage drop (I * R) stays within budget. For a stripe of width W, thickness T, and length L with resistivity rho:

```
R_stripe = rho * L / (W * T) = Rsheet * L / W
```

where Rsheet is the sheet resistance of the metal layer (in ohms per square). For a given current I flowing through the stripe, the IR drop is:

```
V_drop = I * Rsheet * L / W
```

To meet an IR drop budget V_budget, the minimum stripe width is:

```
W_min_IR = I * Rsheet * L / V_budget
```

**EM constraint:** The current density in the stripe must not exceed the EM limit J_max (typically 1 to 10 mA/um^2 depending on the metal layer, temperature, and required lifetime). The minimum width for EM is:

```
W_min_EM = I / (J_max * T)
```

The final stripe width is the larger of W_min_IR and W_min_EM.

**Practical methodology:** In practice, the grid is designed iteratively. An initial grid is specified based on experience and guidelines (e.g., allocate 25% of the top metal pitch to VDD and 25% to VSS). The design is then analyzed with a power integrity tool (RedHawk, Voltus) that computes the actual voltage at every node. Hotspots where the voltage drops below the budget are identified, and the grid is locally strengthened (wider stripes, additional straps on lower layers, additional vias) until the specification is met everywhere.

Grid sizing must account for both average and peak current scenarios. The peak current determines the EM requirement (which is a DC limit for unidirectional current), while both average and peak currents affect IR drop.

---

### Q4. What is the role of power bumps in on-die power delivery?

**Answer:**
Power bumps (C4 bumps, microbumps, or copper pillars) are the physical connections between the package substrate and the die that carry power and ground current. They serve as the entry points for current into the on-die power grid and are one of the most critical elements in the PDN.

Each bump has a finite resistance (typically 5 to 30 mOhm for a C4 bump, and 10 to 50 mOhm for a microbump depending on size and material) and a small inductance (typically 20 to 50 pH). The total resistance and inductance of the bump array is determined by the number of bumps in parallel.

**Bump allocation:** In a typical flip-chip BGA package, 30 to 60 percent of all bumps may be dedicated to power and ground. For a chip with 2000 total bumps, this means 600 to 1200 power/ground bumps. The ratio is driven by the current requirement: if each bump can carry 100 mA average current (limited by EM), a 50 A rail needs at least 500 bumps.

**Bump placement:** Power bumps are distributed across the die area, not just at the periphery, to provide uniform current injection into the power grid. Areas with high current density (processor cores, memory arrays) need more power bumps per unit area. The bump pattern must be co-designed with the on-die power grid to ensure that every point on the grid is within a reasonable distance of a power bump.

**Bump resistance impact:** The total bump resistance contributes to the DC path resistance of the PDN. With N bumps in parallel, each of resistance R_bump:

```
R_bump_total = R_bump / N
```

For 500 bumps at 20 mOhm each: R_bump_total = 20 mOhm / 500 = 0.04 mOhm. This is small, but the current distribution is not perfectly uniform -- bumps near high-current loads carry more current than bumps in quiet areas, so the effective resistance is higher than the ideal parallel calculation suggests.

**Bump inductance impact:** The parallel inductance of the bump array determines the high-frequency impedance seen between the package and the die. Low bump inductance is essential for effective high-frequency decoupling between package-level capacitors and on-die capacitors.

---

### Q5. How does power grid design differ for digital logic blocks versus memory arrays?

**Answer:**
Digital logic blocks (standard cell regions) and memory arrays (SRAM, register files) have fundamentally different power grid requirements due to their different physical structures and current consumption patterns.

**Standard cell regions:** Standard cells are arranged in rows with M1 VDD and VSS rails running along each row. The upper-level power grid must connect down to these M1 rails through a hierarchy of vias and intermediate metal stripes. The current consumption is distributed relatively uniformly (at a macro level) across the cell area, though hotspots exist at high-activity clusters. The power grid can be a regular mesh or stripe pattern that overlays the cell rows, with via connections at regular intervals. The main challenges are achieving uniform voltage across the entire block and accommodating signal routing congestion in the intermediate layers.

**Memory arrays:** SRAMs and other memory macros have a dense, regular structure with internal power distribution that is typically designed by the memory compiler or macro generator. The memory macro has power pins (usually on the top and/or bottom edges, or on a ring around the periphery) that must be connected to the SoC-level power grid. Memory macros tend to have high peak current density during simultaneous read/write operations across many bitlines. The internal power grid of the macro is optimized by the macro designer, but the SoC-level grid must deliver sufficient current to the macro's power pins without excessive IR drop.

Key differences in grid design:

| Aspect | Standard Cell Logic | Memory Arrays |
|--------|-------------------|---------------|
| Current density | Moderate, distributed | High, concentrated at pins |
| Grid topology | Regular mesh/stripes | Ring or peripheral connection |
| Routing interaction | Must coexist with signals | Self-contained internal grid |
| Decoupling | Distributed decap cells | Internal bitline/wordline cap |
| IR drop sensitivity | Moderate (timing) | High (read margin, write margin) |

Memory macros are particularly sensitive to IR drop because the sensing circuits rely on small voltage differences between bitlines, and supply noise directly reduces these margins. A 30 mV IR drop that might cause a 1% timing impact on logic could cause a read failure in a 6T SRAM operating near minimum voltage.

---

### Q6. What is the concept of effective resistance (Reff) of a power grid?

**Answer:**
The effective resistance of a power grid is a single scalar metric that characterizes the overall quality of the power delivery from the bump pads to the loads. It is defined as the ratio of the average voltage drop across the grid to the total current delivered:

```
Reff = V_drop_avg / I_total
```

Alternatively, it can be computed as the power dissipated in the grid divided by the square of the total current:

```
Reff = P_grid / I_total^2
```

Reff captures the combined effect of all resistive elements in the grid: the bump resistance, the top-metal mesh resistance, the intermediate-metal strap resistance, the via resistance at every level, and the M1 rail resistance. It is a more useful metric than the resistance of any single element because it reflects the actual current distribution through the network.

For a well-designed power grid, Reff is typically in the range of 1 to 5 mOhm for the complete on-die network. This is achieved through massive parallelism: hundreds of bumps, thousands of grid intersections, and millions of vias all contribute parallel paths.

Reff is useful for:
- Comparing different grid design alternatives quickly
- Estimating the average IR drop: V_drop = Reff * I_total
- Estimating the power dissipated in the grid: P_grid = Reff * I_total^2
- Setting design targets: if the IR drop budget allocated to the die grid is 20 mV and the total current is 30 A, then Reff must be less than 20 mV / 30 A = 0.67 mOhm

Note that Reff represents the average behavior. The worst-case IR drop (at the point farthest from bumps with the highest local current density) may be 2 to 3 times the average, so the grid must be designed with sufficient margin.

---

### Q7. How do you handle power grid design in the presence of routing blockages and hard macros?

**Answer:**
Real SoC layouts contain numerous obstacles that interrupt the regular power grid pattern: hard macros (memories, analog blocks, I/O cells), routing blockages, and signal congestion areas. Handling these obstacles while maintaining adequate power delivery requires several strategies.

**Grid around macro boundaries:** When a hard macro occupies a rectangular region of the die, the power grid stripes on the upper metal layers must either terminate at the macro boundary or jog around it. Most power grid generation tools support "blockage-aware" grid creation that automatically terminates stripes at macro edges and creates jogs or alternative paths around the macro. The key requirement is that the grid provides sufficient current injection on all sides of the macro so that the macro's power pins receive adequate current.

**Dedicated macro power rings:** It is common practice to place power rings (VDD and VSS stripes running around all four sides of a macro) to collect current from the global grid and deliver it to the macro's power pins. These rings are typically on intermediate metal layers and connect to the global mesh above through vias and to the macro pins below.

**Via stacking near macros:** The grid may need additional via stacks near macro boundaries to bridge current from layers that are continuous over the macro (if allowed by the macro's routing blockage definition) to layers below.

**Adjusting grid pitch:** In regions with high routing congestion, the power grid pitch may need to be relaxed (wider spacing between stripes) to leave room for signal routes. This reduces the grid density and increases the local IR drop, which must be compensated by wider stripes or additional bumps.

**Floorplanning for power:** The floorplan should be designed with power delivery in mind. Placing high-current blocks near the center of the die (closer to more bumps) and ensuring that critical blocks are not in grid shadow regions (areas starved of current due to obstacles) are important considerations. Power integrity analysis should be run iteratively during floorplanning to catch problems early.

---

### Q8. What is the impact of technology scaling on on-die power grid design?

**Answer:**
Technology scaling from one process node to the next has several effects on power grid design, most of which make the design more challenging.

**Increasing current density:** Smaller transistors enable higher logic density, which generally increases the current consumption per unit die area. Even if per-transistor current decreases, the density increases faster, resulting in higher current density that the grid must support.

**Thinner and narrower metals:** At advanced nodes (7 nm, 5 nm, 3 nm), the lower metal layers become extremely thin and narrow, with significantly increased resistivity due to electron scattering effects at grain boundaries and surfaces. This increases the sheet resistance of the lower metal layers, making it harder to deliver current through the M1 rails and intermediate stripes.

**More metal layers:** Advanced nodes compensate for thinner individual layers by providing more total metal layers (12 to 16 layers at 3 nm versus 6 to 9 layers at 65 nm). This provides more layers for power distribution but also more layers to traverse with vias, each adding resistance.

**Increased via resistance:** As via sizes shrink, individual via resistance increases (inversely proportional to via area). More vias in parallel are needed to maintain low via-stack resistance, consuming additional area.

**Copper resistivity increase:** Below about 30 nm linewidth, copper resistivity increases significantly above the bulk value due to surface scattering, grain boundary scattering, and the increasing fraction of the cross-section occupied by the barrier/liner materials. At 3 nm nodes, the effective resistivity of narrow metal lines can be 2 to 5 times the bulk copper value.

**Alternative metals:** Some advanced nodes are introducing alternative metals (cobalt, ruthenium) for the lowest metal layers, which have higher bulk resistivity than copper but potentially better resistance scaling at very small dimensions due to shorter electron mean free paths.

These trends collectively mean that on-die power grids must be designed more aggressively at each new node: more metal resources dedicated to power, more bumps, and more on-die decoupling. The fraction of die area and routing resources consumed by the power grid has been steadily increasing.

---

### Q9. How do clock gating and power gating events affect on-die power grid design?

**Answer:**
Clock gating and power gating create sudden, large changes in current demand that stress the power grid dynamically.

**Clock gating/ungating:** When a clock domain is gated (disabled), the switching current of all flip-flops and combinational logic in that domain drops to near zero (only leakage remains). When the clock is ungated, the entire domain resumes switching simultaneously, creating a large current step. This step can be 50% or more of the domain's peak current, occurring within a few clock cycles (nanoseconds). The resulting di/dt interacts with the grid inductance to produce voltage droops. The grid must be designed to handle these transients without excessive voltage noise.

Design implications: the power grid must have sufficient on-die decoupling near every clock domain to provide the transient current during ungating. Rush current limiting (gradually increasing clock frequency over several cycles after ungating) can reduce the di/dt and ease the grid requirement.

**Power gating:** Power gating completely shuts off the supply voltage to an inactive block by turning off header switches (PMOS transistors between the always-on VDD rail and the block's virtual VDD rail) or footer switches (NMOS between virtual VSS and ground). When the block powers up, the header switches must charge all the capacitance in the block (gate caps, decaps, wiring caps) from zero to VDD. This creates a large rush current that flows through the power grid and header switches.

Design implications: the rush current during power-up can far exceed the normal operating current of the block. Header switches are typically designed with a programmable turn-on sequence (a "daisy chain" of progressively larger switch segments) to limit the rush current to an acceptable level. The power grid must handle the rush current without drooping below specification on the always-on VDD rail, which powers other active blocks. The virtual VDD rail inside the gated block may droop significantly during power-up, but this is acceptable because the block is not yet performing useful computation.

The grid design must account for the worst-case simultaneous scenario: for example, one large domain ungating while another domain is at peak activity. Power integrity simulation must include these corner cases.

---

### Q10. What are the common metrics used to evaluate power grid quality?

**Answer:**
Several metrics are used to assess and compare power grid designs during the physical design flow:

**Maximum IR drop (static):** The worst-case voltage drop anywhere on the die under average current conditions. Reported as an absolute voltage (mV) and as a percentage of VDD. A typical target is less than 3 to 5 percent of VDD for static IR drop.

**Maximum dynamic voltage drop:** The worst-case instantaneous voltage drop at any point on the die during a dynamic simulation with realistic switching activity. This includes both the resistive and inductive contributions and is typically 2 to 3 times larger than the static IR drop.

**Voltage distribution histogram:** A histogram showing what fraction of the die area experiences each level of voltage drop. A well-designed grid has a tight distribution clustered at low IR drop values, while a poor grid has a long tail extending to high values.

**Effective resistance (Reff):** As described in Q6, the scalar resistance metric of the complete grid.

**Grid coverage (density):** The percentage of routing tracks on each metal layer that are occupied by power stripes. Typical targets are 20 to 40 percent on the upper thick metal layers.

**EM margin:** The ratio of the EM current limit to the actual current in each stripe, via, and bump. All elements must have EM margin greater than 1.0 (or the required derating factor). The minimum EM margin across the entire grid indicates the weakest point.

**Number and distribution of via stacks:** The density of via connections between metal layers affects the local grid resistance. Sparse via connections create bottlenecks.

**Power bump utilization:** The average and peak current per power bump, compared to the bump EM limit. Uneven current distribution across bumps indicates a grid asymmetry.

These metrics are reported by power integrity tools and are tracked throughout the physical design flow. Sign-off criteria typically specify maximum allowed values for IR drop and minimum EM margins.
