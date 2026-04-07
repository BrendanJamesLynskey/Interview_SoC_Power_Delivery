# Bump and Via Allocation

This section covers C4 bump and microbump design for power delivery, power/ground bump allocation ratios, via resistance management, and optimization of the bump-to-die interface.

---

### Q1. What types of bump interconnects are used for SoC power delivery?

**Answer:**
Several bump technologies are used to connect the die to the package substrate, each with different electrical and mechanical characteristics.

**C4 (Controlled Collapse Chip Connection) bumps:** Traditional solder bumps used in flip-chip packaging. They consist of solder balls (lead-free SnAg or SnAgCu) deposited on under-bump metallization (UBM) pads on the die, which collapse and bond to matching pads on the package substrate during reflow. C4 bumps have pitches of 100 to 200 um, diameters of 75 to 150 um after collapse, and resistance of 5 to 25 mOhm per bump. They are the standard interconnect for mainstream flip-chip BGA packages.

**Copper pillar bumps:** An evolution of C4, copper pillar bumps use a tall copper post with a solder cap. The copper pillar provides lower resistance than a pure solder bump (copper resistivity is much lower than solder) and better electromigration resistance. Copper pillars can be fabricated at finer pitches (50 to 100 um) than traditional C4.

**Microbumps:** Very small bumps used in advanced packaging (2.5D and 3D). Microbumps have pitches of 20 to 55 um and diameters of 10 to 30 um. Each microbump has higher resistance (20 to 80 mOhm) than a C4 bump due to its smaller size, but the much higher density allows many more parallel connections. Microbumps are used to connect dies to silicon interposers or to stack dies in 3D configurations.

**Hybrid bonding:** A direct copper-to-copper bonding technology that eliminates solder entirely. Copper pads on the die are bonded directly to copper pads on the substrate or another die at temperatures of 200 to 300C with applied pressure. Hybrid bonding enables pitches below 10 um with very low resistance per connection (less than 10 mOhm). It is used in the most advanced 3D stacking applications.

| Technology | Pitch | Resistance/bump | Inductance/bump | Application |
|-----------|-------|-----------------|-----------------|-------------|
| C4 solder | 100-200 um | 5-25 mOhm | 20-50 pH | Mainstream FC-BGA |
| Cu pillar | 50-130 um | 3-15 mOhm | 10-40 pH | Advanced FC-BGA |
| Microbump | 20-55 um | 20-80 mOhm | 10-30 pH | 2.5D/3D packaging |
| Hybrid bond | 1-10 um | 1-10 mOhm | 1-5 pH | Advanced 3D stacking |

---

### Q2. How do you determine the number of power and ground bumps needed?

**Answer:**
The number of power/ground bumps is determined by three constraints: electromigration current limit, IR drop budget, and inductance (target impedance) requirement. The constraint that requires the most bumps is the binding one.

**EM constraint:** Each bump has a maximum DC current rating (determined by EM testing, typically 50 to 200 mA per C4 bump at operating temperature). The minimum number of VDD bumps for EM is:

```
N_vdd_em = I_total / I_max_per_bump
```

For a 50 A rail with 150 mA per bump: N_vdd = 50/0.15 = 334 bumps. An equal number of VSS bumps is needed for the return path.

**IR drop constraint:** The total bump resistance (R_bump/N) must be a small fraction of the total IR drop budget:

```
N_vdd_ir = R_bump * I_total / V_drop_budget_bumps
```

If R_bump = 15 mOhm, I_total = 50 A, and the bump IR drop budget is 5 mV:

```
N_vdd_ir = 15e-3 * 50 / 5e-3 = 150 bumps
```

**Inductance constraint:** The total bump inductance (L_bump/N) must keep the inductive impedance below the target at the critical frequency:

```
N_vdd_L = 2 * pi * f * L_bump / Ztarget
```

At f = 500 MHz, L_bump = 40 pH, Ztarget = 1 mOhm:

```
N_vdd_L = 2 * pi * 500e6 * 40e-12 / 1e-3 = 126 bumps
```

The binding constraint is EM: 334 VDD bumps. Adding a 20% margin: 400 VDD bumps.

**Total bump count:** With 400 VDD and 400 VSS bumps, that is 800 power/ground bumps. If the die has 2500 total bumps, the power/ground fraction is 800/2500 = 32%. Typical power/ground fractions range from 30% to 60% depending on the current requirements.

---

### Q3. How should power and ground bumps be distributed across the die?

**Answer:**
The distribution of power and ground bumps across the die area is critical for uniform voltage delivery and current sharing.

**Uniform distribution:** The simplest approach is to distribute VDD and VSS bumps uniformly across the die in a regular pattern (e.g., a checkerboard of VDD and VSS bumps). This ensures that every region of the die has nearby power access points, preventing localized IR drop hotspots.

**Proportional to current density:** A more optimized approach allocates more power bumps to regions with higher current density. For example, a processor core that draws 60% of the total current might receive 60% of the power bumps, even if it occupies only 40% of the die area. This requires knowledge of the spatial current distribution, which is available from power analysis tools.

**Paired VDD/VSS:** VDD and VSS bumps should be placed in close proximity (adjacent bumps) to minimize the inductance of the current loop. When a VDD bump and its nearest VSS neighbor are close together, the current loop area (and hence inductance) is small. A checkerboard pattern naturally achieves this.

**Avoid clustering:** Grouping all power bumps in one area and all signal bumps in another creates a problem: the power-dense region has very low IR drop but the signal-dense region has no nearby power access, leading to high IR drop. The interleaving of power and signal bumps across the die is essential.

**Consider signal routing:** Power bumps interrupt the signal routing channels on the package substrate. Too many power bumps in a dense signal area can create routing congestion. The bump assignment must balance power delivery needs with signal routing requirements.

**Hard macro considerations:** Large hard macros (memories, analog blocks) may have their power pins at specific locations along their periphery. The bump map must align power bumps with these pin locations to provide direct current injection.

---

### Q4. What is the resistance of a C4 bump and what factors affect it?

**Answer:**
The total resistance of a C4 bump includes contributions from the under-bump metallization (UBM) on the die, the solder ball, and the bond pad on the package substrate.

**UBM resistance:** The UBM stack (typically Ti/Cu/Ni or TiW/Cu layers) has a total resistance of 1 to 5 mOhm, depending on the layer thicknesses and the pad diameter.

**Solder resistance:** The solder (SnAg, resistivity approximately 12 uOhm-cm) forms the bulk of the bump. For a collapsed bump with height h and cross-sectional area A:

```
R_solder = rho * h / A
```

For h = 50 um and diameter = 100 um (A = pi*(50)^2 = 7854 um^2):

```
R_solder = 12e-6 * 50e-4 / 7854e-8 = 7.6 mOhm
```

**Bond pad and intermetallic resistance:** The interface between the solder and the substrate pad adds 1 to 3 mOhm due to intermetallic compound (IMC) formation.

**Total bump resistance:** Summing the contributions: 3 + 7.6 + 2 = approximately 12.6 mOhm for a typical C4 bump with 100 um diameter and 50 um height.

Factors that affect bump resistance:
- **Bump diameter:** Larger diameter reduces resistance (larger cross-section). R is inversely proportional to the square of the diameter.
- **Bump height:** Taller bumps have higher resistance. R is proportional to height.
- **Solder composition:** Lead-free solders have higher resistivity than leaded solders.
- **Temperature:** Solder resistivity increases with temperature (approximately 0.4%/C).
- **Aging:** Intermetallic growth over time can increase resistance slightly.
- **Current crowding:** At high frequencies or with non-uniform current distribution, the effective resistance can be higher than the DC value due to current crowding at the bump periphery.

---

### Q5. How does via resistance in the package substrate affect power delivery?

**Answer:**
Package substrate vias connect the different metal layers within the substrate, routing power current from the BGA-side planes up to the die-side planes. Their resistance contributes to the total PDN DC resistance.

The resistance of a single cylindrical via is:

```
R_via = rho * h / (pi * r^2)
```

For a copper-plated via (resistivity ~ 2 uOhm-cm) with drill diameter 75 um and substrate thickness 800 um:

If the via is plated (hollow), the conducting area is the annular ring:

```
A = pi * (r_outer^2 - r_inner^2)
```

For a 75 um drill with 20 um plating thickness: r_outer = 37.5 um, r_inner = 17.5 um:

```
A = pi * (37.5^2 - 17.5^2) = pi * (1406.25 - 306.25) = pi * 1100 = 3456 um^2
R_via = 2e-8 * 800e-6 / 3456e-12 = 1.6e-11 / 3.456e-9 = 4.6 mOhm
```

For a filled (solid) via with 75 um diameter:

```
A = pi * (37.5)^2 = 4418 um^2
R_via = 2e-8 * 800e-6 / 4418e-12 = 3.6 mOhm
```

High-current power rails use many vias in parallel. If 100 vias connect the VDD planes:

```
R_via_total = 4 mOhm / 100 = 0.04 mOhm
```

This is typically a small contribution to the total PDN resistance. However, the via inductance (10 to 50 pH per via) is often more significant than the resistance for high-frequency performance.

Key design considerations:
- Maximize the number of power vias within routing constraints
- Place vias in groups (via arrays) under power bumps and capacitor pads
- Use filled vias for mechanical strength and lower resistance
- Account for via anti-pad clearance requirements that reduce plane connectivity

---

### Q6. What is current crowding in bump connections and how does it affect reliability?

**Answer:**
Current crowding occurs when the current distribution within a bump is non-uniform, with higher current density at certain locations. This is particularly important for EM reliability because the local current density at the crowding point can be several times the average current density.

**Cause of current crowding:** In a C4 or microbump connection, current enters from the UBM pad on one side and exits through the bond pad on the other side. If the UBM and bond pad are not perfectly aligned (offset) or if the current approaches from one direction (due to the trace routing on the substrate or the die metal grid), the current concentrates at one edge of the bump rather than distributing uniformly across the entire cross-section.

Additionally, at the interface between the UBM and the solder, the current transitions from a thin metal film to a bulk solder volume. This transition creates a geometric constriction that concentrates current at the periphery of the UBM pad.

**Impact on EM:** The current density at the crowding point can be 3 to 10 times the average current density. Since EM lifetime depends on J^(-n), this concentration dramatically reduces the local EM lifetime. Voids tend to nucleate first at current crowding locations, which are typically at the interface between the UBM and the solder or at the die-side corner of the bump.

**Mitigation:**
- **Center-fed bumps:** Design the UBM and die-side metal routing so that current enters the bump as uniformly as possible from all directions, rather than predominantly from one side.
- **Larger UBM pads:** A larger UBM pad distributes the current over a larger area, reducing the peak current density.
- **Copper pillar bumps:** The tall copper column distributes current more uniformly than a short solder bump because the pillar acts as a current-spreading structure.
- **Multiple via connections:** Connecting the bump to the die metal grid through multiple vias distributed around the bump pad reduces current concentration.

---

### Q7. How do you optimize the power bump map for a multi-voltage-domain SoC?

**Answer:**
Multi-voltage-domain SoCs require separate bump allocations for each voltage rail, adding complexity to the bump map design.

**Step 1: Budget bumps per rail.** For each voltage domain, calculate the required number of VDD bumps (based on current, EM, IR drop, and inductance constraints). Also allocate VSS bumps -- VSS is typically shared across domains but may also need dedicated bumps for noise-sensitive analog domains.

Example for a 3-domain SoC with 3000 total bumps:

| Domain | Current (A) | VDD bumps needed | VSS bumps |
|--------|------------|-----------------|-----------|
| VCORE | 40 | 300 | 300 |
| VGPU | 20 | 150 | 150 |
| VIO | 5 | 40 | 40 |
| Signal I/O | -- | -- | -- |
| Total P/G | -- | 490 | 490 |
| Remaining for signals | -- | 2020 | -- |

**Step 2: Define bump regions.** Align the bump regions with the die floorplan. VCORE bumps are placed over the CPU cores, VGPU bumps over the GPU area, etc. This minimizes the lateral current path on the die from bump to load.

**Step 3: Handle shared VSS.** If VSS is shared across domains, the VSS bumps can be distributed uniformly and shared by all VDD domains. If separate analog ground (AVSS) is needed, dedicate specific bumps in the analog block region.

**Step 4: Buffer zones between domains.** At the boundary between two VDD domains on the die, there should be VSS bumps to provide a clean ground reference and to avoid coupling between the domains through shared bump parasitics.

**Step 5: Package routing feasibility.** The bump map must be routable on the package substrate. Each VDD domain needs its own plane or plane region in the substrate, and the BGA-to-bump routing must not create bottlenecks. Signal bumps must be routable to their corresponding BGA balls without crossing power regions in ways that create integrity problems.

**Step 6: Iterate with simulation.** Run power integrity analysis for each domain to verify that the bump allocation provides adequate IR drop and EM margins. Adjust the bump map as needed.

---

### Q8. What is the inductance of a C4 bump and how is it calculated?

**Answer:**
The inductance of a C4 bump arises from the current loop formed by the VDD bump and its nearest VSS return bump. This is properly described as the loop inductance or partial mutual inductance.

For a single VDD bump, the partial self-inductance can be approximated as a short cylindrical conductor:

```
L_self ~ (mu_0 / (2*pi)) * h * [ln(2*h/r) - 1 + r/(2*h)]
```

For h = 60 um and r = 50 um:

```
L_self ~ (4*pi*1e-7 / (2*pi)) * 60e-6 * [ln(120/50) - 1 + 50/120]
       = 2e-7 * 60e-6 * [0.875 - 1 + 0.417]
       = 12e-12 * 0.292
       = 3.5 pH
```

However, the relevant quantity for PDN analysis is the loop inductance -- the inductance of the VDD-VSS bump pair that the current must traverse. The loop inductance depends on the separation between the VDD and VSS bumps:

```
L_loop = (mu_0 / pi) * h * [ln(d/r) - 1 + r/d]
```

where d is the center-to-center spacing between the VDD and VSS bumps.

For d = 200 um (bump pitch), h = 60 um, r = 50 um:

```
L_loop = (4e-7 / pi) * 60e-6 * [ln(200/50) - 1 + 50/200]
       = 1.273e-7 * 60e-6 * [1.386 - 1 + 0.25]
       = 7.64e-12 * 0.636
       = 4.9 pH
```

For a closer-spaced bump pair (d = 100 um):

```
L_loop = 1.273e-7 * 60e-6 * [ln(100/50) - 1 + 0.5]
       = 7.64e-12 * [0.693 - 1 + 0.5]
       = 7.64e-12 * 0.193
       = 1.5 pH
```

With N VDD/VSS bump pairs in parallel, the total loop inductance is L_loop/N.

This calculation shows why bump pitch matters: halving the pitch from 200 um to 100 um reduces the per-pair inductance by more than 3x (from 4.9 pH to 1.5 pH), and the higher bump density also increases N.

---

### Q9. How do thermal bumps interact with the power delivery network?

**Answer:**
Thermal bumps (also called thermal balls or dummy bumps) are bump connections that are not electrically functional but serve to conduct heat from the die to the package substrate. In some designs, thermal bumps are connected to the power or ground network, serving a dual role.

**Dual-purpose thermal/power bumps:** If thermal bumps are connected to the VDD or VSS rails, they provide additional parallel resistance and inductance paths for power delivery while also conducting heat. This is a common and efficient approach because it uses bumps that would otherwise be electrically wasted.

**Impact on PDN:** Adding thermal bumps to the power grid increases the effective number of power bumps, reducing the total resistance and inductance of the bump array. If 100 thermal bumps are connected to VSS in addition to 400 dedicated VSS bumps, the effective VSS bump count becomes 500, reducing the bump resistance by 20%.

**Current distribution:** Thermal bumps may be located in areas that do not have direct current demand (e.g., over analog blocks that draw very little current). The current flowing through thermal bumps for power delivery purposes may be minimal, but they still contribute to the parallel impedance and provide alternative current paths during transients.

**Thermal considerations:** The primary purpose of thermal bumps is heat removal. The heat conducted through these bumps flows from the die through the solder to the package substrate, where it spreads to the BGA balls and ultimately to the PCB and heat sink. If thermal bumps carry significant power current, the I^2*R heating in the bumps is additional heat that must be managed, but this is typically negligible compared to the die's active power.

**Design practice:** Most modern SoC bump maps attempt to minimize unused (floating) bumps. Any bump that does not carry a signal is connected to either VDD or VSS, serving as both a power delivery element and a thermal conduit.

---

### Q10. What are the tradeoffs between C4 bumps and microbumps for power delivery?

**Answer:**
The choice between C4 bumps and microbumps involves tradeoffs in resistance, density, cost, and total current capacity.

**Resistance per bump:** C4 bumps have lower resistance per bump (5-25 mOhm) compared to microbumps (20-80 mOhm) because of their larger cross-sectional area. However, this advantage is offset by the higher density of microbumps.

**Density and total bumps:** Microbumps at 50 um pitch provide 4x the density of C4 bumps at 100 um pitch (and 16x the density of 200 um pitch C4). For a 10 mm x 10 mm die:
- C4 at 150 um: ~4400 bumps
- Microbump at 50 um: ~40000 bumps

**Total resistance:** With more bumps in parallel, the total resistance can be lower for microbumps despite the higher per-bump resistance:
- C4 (1500 VDD bumps at 15 mOhm): total = 15/1500 = 0.01 mOhm
- Microbump (12000 VDD bumps at 50 mOhm): total = 50/12000 = 0.0042 mOhm

**Total inductance:** Similarly, the total inductance benefits from the higher bump count. Additionally, microbumps at finer pitch have smaller VDD-VSS loop area, reducing per-pair inductance.

**EM capacity:** Total current capacity = N * I_max_per_bump. Even if each microbump has a lower current limit, the larger number of bumps provides higher total capacity.

**Cost and complexity:** Microbump assembly requires more precise alignment, more expensive substrates with finer features, and more complex underfill processes. The package substrate must have routing tracks and via pads at the microbump pitch, which may require more substrate layers.

**Reliability:** Microbumps have less solder volume and smaller interfaces, making them potentially more susceptible to cracking under thermal cycling stress. However, the copper pillar + solder cap design used for most microbumps provides good reliability.

For high-performance SoCs, the trend is toward finer bump pitch (microbumps) because the power delivery and signal density benefits justify the increased packaging cost. For cost-sensitive designs, C4 bumps at 100-150 um pitch remain the mainstream choice.
