# IR Drop and Electromigration

This section covers static and dynamic IR drop analysis, electromigration design rules, Black's equation, and reliability verification for on-die power delivery.

---

### Q1. What is static IR drop and how is it analyzed?

**Answer:**
Static IR drop is the DC voltage reduction along the resistive path of the power delivery network when a time-averaged current flows. It is computed by solving Kirchhoff's current law (KCL) at every node of the power grid, given the known current sinks (the average current consumed by each standard cell and macro instance) and the known voltage sources (the power bumps, assumed to be at the nominal VDD after accounting for package and board IR drop).

The power grid is modeled as a resistive network: each metal segment between grid intersections is a resistor (R = Rsheet * L / W), each via or via array is a resistor, and each bump is a resistor to the package plane voltage. The current drawn by each cell instance is modeled as a constant current sink connected between VDD and VSS at the cell's power pin location.

The resulting system of linear equations (one KCL equation per node) is solved simultaneously using sparse matrix techniques. The solution gives the voltage at every node, and the IR drop at each point is VDD_nominal - V_node.

Tools like ANSYS RedHawk and Cadence Voltus perform this analysis by:
1. Reading the physical design database (DEF/LEF) to extract the power grid geometry
2. Computing the resistance of every grid segment and via
3. Reading current data from either vector-based simulation or vectorless statistical estimation
4. Solving the resistive network to produce a voltage drop map

The output is typically visualized as a color-coded voltage map overlaid on the die floorplan, with red/hot regions indicating high IR drop and blue/cool regions indicating low IR drop. Sign-off criteria typically require the maximum static IR drop to be below 3 to 5 percent of VDD.

---

### Q2. How does dynamic IR drop differ from static IR drop?

**Answer:**
Dynamic IR drop analysis accounts for the time-varying nature of current demand and includes the effects of both resistance and inductance in the power grid, as well as the charge stored in on-die decoupling capacitors. It provides a more realistic assessment of the actual voltage experienced by transistors during operation.

In dynamic analysis, the current drawn by each instance varies with time based on its switching activity. This time-varying current profile is either extracted from a gate-level simulation with a realistic workload (vector-based analysis) or estimated statistically based on toggle rates and timing windows (vectorless analysis).

The power grid is modeled as an RLC network: each metal segment has both resistance and inductance, and the on-die decoupling capacitance (both intrinsic gate cap and intentional decap cells) is included. The package model (bump resistance and inductance, package plane impedance) is also included to capture the interaction between the die and package.

The simulation proceeds in the time domain, typically over a window of several clock cycles centered on the worst-case activity scenario. The result is the voltage at every node as a function of time. The worst-case dynamic IR drop is the maximum voltage deviation from nominal at any point and any time during the simulation.

Key differences from static analysis:

| Aspect | Static IR Drop | Dynamic IR Drop |
|--------|---------------|-----------------|
| Current model | Time-averaged DC | Time-varying waveforms |
| Grid model | Resistive (R only) | RLC network |
| Decoupling | Not modeled | Included (C and its ESR) |
| Package model | Simple R | RLC or S-parameter |
| Computation | Single matrix solve | Time-domain simulation |
| Result | Spatial voltage map | Spatial + temporal voltage |
| Typical magnitude | 20-50 mV | 50-150 mV |

Dynamic IR drop is typically 2 to 3 times larger than static because it includes the inductive L*di/dt contribution, which is dominant during fast switching events.

---

### Q3. What causes dynamic IR drop hotspots and how are they mitigated?

**Answer:**
Dynamic IR drop hotspots occur at locations where a combination of high transient current demand and inadequate local power delivery capacity creates excessive voltage deviation. The most common causes are:

**Simultaneous switching of dense logic:** When a large cluster of gates switches on the same clock edge (e.g., a wide datapath operation or a vector unit), the aggregate current spike can exceed the local grid's ability to supply current without significant voltage drop.

**Clock tree buffers:** Clock buffers drive large capacitive loads and switch every cycle, creating periodic high-current pulses. Clusters of clock buffers can create significant local current density.

**Memory read/write events:** When many bitlines in an SRAM array are simultaneously activated, the transient current can be very large over a short period.

**Clock ungating events:** When a previously idle clock domain resumes switching, all flip-flops and downstream logic begin switching simultaneously, creating a large current step within a few clock cycles.

**Regions far from power bumps:** Areas that are physically distant from the nearest power bump experience higher grid impedance, amplifying any current transient into a larger voltage drop.

**Mitigation strategies:**

- **Strengthen the local power grid:** Add more power stripes, wider stripes, or additional via stacks in the hotspot area to reduce local grid resistance and inductance.
- **Add local decoupling:** Insert additional decap cells near the hotspot to provide local charge storage.
- **Spread the activity:** If possible, retime or pipeline the logic to spread the switching activity over multiple clock cycles, reducing the peak di/dt.
- **Stagger clock enables:** When ungating multiple sub-blocks, stagger the enable signals over several cycles to avoid a simultaneous current surge.
- **Add power bumps:** If the floorplan allows, add more power bumps in the hotspot region to provide more current injection points.
- **Reduce local logic density:** Spread the placement of high-activity cells to distribute the current demand more evenly.

---

### Q4. What is electromigration and why is it a reliability concern in power grids?

**Answer:**
Electromigration (EM) is the gradual displacement of metal atoms in a conductor due to momentum transfer from conducting electrons. When a high current density flows through a metal wire, the "electron wind" force pushes atoms in the direction of electron flow (opposite to conventional current direction). Over time, this atomic displacement creates voids (where atoms have departed) and hillocks (where atoms accumulate). Voids increase the local resistance of the conductor and can eventually cause an open circuit. Hillocks can cause short circuits to adjacent conductors.

EM is a critical reliability concern in power grids because:

**Constant current stress:** Unlike signal wires that carry bidirectional AC current (which causes less net atomic displacement due to cancellation), power grid wires carry predominantly unidirectional DC current. This unidirectional stress is the worst case for EM.

**High current density:** Power grid wires carry large currents, and the current density (A/um^2 or mA/um) can be close to or exceed the EM limit, especially at narrow via connections and near power bumps where current concentrates.

**Long lifetime requirement:** SoCs are expected to function reliably for 5 to 10 years (or longer for automotive and infrastructure applications) at elevated temperatures. EM is a wearout mechanism that degrades over the entire product lifetime.

**Large number of elements:** A power grid contains millions of metal segments and vias. Even if the probability of EM failure in any single element is very small, the large number of elements means the chip-level failure probability can be significant.

EM is characterized by Black's equation, which relates the mean time to failure (MTTF) to the current density and temperature:

```
MTTF = A * J^(-n) * exp(Ea / (k * T))
```

where A is a material-dependent constant, J is the current density, n is the current density exponent (typically 1 to 2), Ea is the activation energy (0.7 to 0.9 eV for copper), k is Boltzmann's constant, and T is the absolute temperature.

---

### Q5. How are EM design rules applied to the on-die power grid?

**Answer:**
EM design rules specify the maximum allowable current density for each metal layer, via type, and bump type, considering the target product lifetime and operating conditions. These rules are derived from Black's equation and accelerated stress testing during technology development.

**Metal segment rules:** Each metal layer has a maximum DC current density limit (J_max) specified in mA/um of wire width (or equivalently mA/um^2 of cross-sectional area). The limit depends on the metal layer, wire direction, temperature, and required lifetime. Typical values for copper at 105C junction temperature with a 10-year lifetime might be:

- Lower thin metals (M1-M4): 0.5 to 2 mA/um width
- Intermediate metals (M5-M8): 1 to 5 mA/um width
- Upper thick metals (M9-M12): 5 to 20 mA/um width

**Via rules:** Individual vias have a maximum current limit (e.g., 0.1 to 0.5 mA per via). Via arrays carry proportionally more current.

**Bump rules:** C4 bumps have maximum DC current limits, typically 50 to 200 mA per bump depending on the bump size, material (lead-free solder, copper pillar), and temperature.

**Application in power integrity tools:** After solving the power grid for current distribution, the tool computes the current through every metal segment, via, and bump. It then compares each current to the applicable EM limit and reports violations (elements where the current exceeds the limit) and the EM margin (limit/actual) for each element.

**AC and bidirectional EM rules:** For signal wires and some power grid segments that carry bidirectional current (e.g., ground wires in certain configurations), relaxed "AC EM" rules apply because the bidirectional current partially cancels the net atomic displacement. AC EM limits are typically 2 to 5 times higher than DC limits.

**Remediation:** EM violations are fixed by widening the metal stripe, adding parallel stripes, increasing the number of vias in a via array, or redistributing the current by modifying the grid topology.

---

### Q6. Explain Black's equation and its parameters.

**Answer:**
Black's equation is the fundamental model for electromigration lifetime:

```
MTTF = A * J^(-n) * exp(Ea / (k * T))
```

**A (pre-exponential constant):** A material and geometry-dependent constant determined experimentally. It accounts for the cross-sectional area, grain structure, and other physical properties of the conductor.

**J (current density):** The current per unit cross-sectional area of the conductor, typically in A/cm^2 or mA/um^2. Higher current density means more momentum transfer to atoms and shorter lifetime. The relationship is strongly nonlinear due to the exponent n.

**n (current density exponent):** Typically between 1 and 2. A value of n = 1 corresponds to void growth limited by atomic drift (void-growth regime), while n = 2 corresponds to void nucleation-limited failure (nucleation regime). Copper interconnects at advanced nodes typically show n close to 1 for long, narrow lines and closer to 2 for short lines with blocking boundaries.

**Ea (activation energy):** The energy barrier for atomic diffusion. Different diffusion paths have different activation energies:
- Bulk diffusion: ~2.1 eV (negligible at normal temperatures)
- Grain boundary diffusion: ~0.9 eV
- Interface diffusion (top surface, Cu/barrier interface): ~0.7 to 0.8 eV
- Surface diffusion: ~0.5 eV

The dominant diffusion path (lowest Ea) determines the EM lifetime. For copper damascene interconnects, interface diffusion along the Cu/cap or Cu/barrier interface is typically dominant, giving Ea around 0.7 to 0.9 eV.

**k (Boltzmann's constant):** 8.617 x 10^-5 eV/K.

**T (absolute temperature):** The temperature of the conductor in Kelvin. EM lifetime is extremely sensitive to temperature due to the exponential dependence. A 10 to 15 degree increase in temperature can reduce the lifetime by roughly 2x.

**Practical implications:** For design rule development, foundries characterize the EM parameters (A, n, Ea) through accelerated testing at elevated temperatures and high current densities, then extrapolate to normal operating conditions. The resulting J_max limits for each metal layer are the values that ensure a specified MTTF (typically corresponding to a failure rate below a certain threshold, e.g., 0.1% failures at end-of-life).

---

### Q7. What is the difference between void-nucleation and void-growth EM failure modes?

**Answer:**
These two failure modes represent different physical stages of electromigration damage and have different implications for design rules.

**Void nucleation:** Before any void exists, the electron wind creates a tensile stress gradient in the metal. Atoms are swept toward the anode end, creating compression there and tension at the cathode end. When the tensile stress at the cathode exceeds a critical value, a void nucleates (typically at a grain boundary triple point or at the interface between the copper and the barrier/cap layer). The time to nucleation depends on the current density, temperature, line length, and the critical stress threshold. For short lines (shorter than the "Blech length"), the back-stress from the stress gradient can completely oppose the electron wind force, and no void nucleates regardless of time -- this is the Blech effect or short-length effect. The current density exponent n for nucleation-limited failure is typically close to 2.

**Void growth:** After a void nucleates, it grows as atoms continue to be swept away from the cathode region. The void grows along the conductor, increasing the local resistance. If the void spans the entire conductor width, it creates an open circuit and the line fails. The growth rate depends on the current density and temperature, and the time from nucleation to failure depends on the void growth rate and the conductor cross-section. The current density exponent n for growth-limited failure is typically close to 1.

**Design implications:**

For long lines (power grid stripes), void growth is typically the dominant failure mode, and n is close to 1. This means the EM limit scales linearly with J, and halving the current density approximately doubles the lifetime.

For short lines and vias, the nucleation threshold may never be reached if the line is shorter than the Blech length. Foundries may provide relaxed EM rules for short segments (below a specified length) because of this effect.

The Blech length is approximately:

```
L_Blech = sigma_critical * Omega / (Z* * e * rho * J)
```

where sigma_critical is the critical stress for void nucleation, Omega is the atomic volume, Z* is the effective charge number, e is the electron charge, and rho is the resistivity. For copper at typical conditions, the Blech product (J * L_crit) is approximately 3000 to 5000 A/cm.

---

### Q8. How do you perform EM signoff for an SoC power grid?

**Answer:**
EM signoff is the process of verifying that every conductor, via, and bump in the power grid meets the EM lifetime requirement under worst-case operating conditions. The process involves:

**Step 1: Define the operating conditions.** Specify the maximum junction temperature (e.g., 105C or 125C), the target lifetime (e.g., 10 years), and the maximum allowed failure rate (e.g., 100 FIT, where 1 FIT = 1 failure per billion device-hours). These parameters determine the EM current density limits.

**Step 2: Extract the power grid and current distribution.** Run the power integrity tool to solve the grid and determine the current through every element. The current profile should represent the worst-case sustained operating condition (typically the workload that produces the highest average current in each region).

**Step 3: Apply EM rules.** For each metal segment, via, and bump, the tool compares the computed current to the applicable EM limit. The EM limit depends on the metal layer, direction, temperature, and line length (if short-length effects are accounted for).

**Step 4: Report violations and margins.** Elements where the current exceeds the EM limit are flagged as violations. The EM margin (ratio of limit to actual current) is reported for all elements. A margin greater than 1.0 means the element passes; less than 1.0 is a violation.

**Step 5: Fix violations.** Common fixes include:
- Widening the metal stripe (increases cross-sectional area, reduces J)
- Adding parallel stripes on the same or adjacent layers
- Increasing the number of vias in a via array
- Adding more power bumps to reduce per-bump current
- Modifying the grid topology to redistribute current more evenly

**Step 6: Re-verify after fixes.** After making changes, the grid is re-solved and EM re-checked to confirm that all violations are resolved and that the fixes did not create new violations elsewhere.

**Temperature considerations:** EM analysis must be performed at the worst-case temperature. For chips with significant self-heating, the temperature distribution across the die is not uniform, and the EM analysis should use a spatially varying temperature map (from thermal analysis) rather than a single temperature value.

---

### Q9. What is the Blech effect and how does it impact power grid EM design rules?

**Answer:**
The Blech effect (also called the short-length effect) is the observation that metal lines shorter than a critical length do not experience electromigration failure regardless of the current density or stress time, provided the current density times length product (J * L) is below a threshold.

The physics behind the Blech effect is as follows. As electromigration displaces atoms toward the anode end of a line, a mechanical stress gradient builds up: compressive stress at the anode and tensile stress at the cathode. This stress gradient creates a back-diffusion flux that opposes the electron wind-driven flux. For sufficiently short lines, the stress gradient reaches a steady state where the back-diffusion exactly balances the electromigration flux, and no further net atomic transport occurs. The void never nucleates because the tensile stress at the cathode never reaches the critical nucleation threshold.

The critical condition is expressed as the Blech product:

```
(J * L)_critical = sigma_critical * Omega / (Z* * e * rho)
```

For copper interconnects, experimental values of the critical Blech product are typically 3000 to 5000 A/cm (or equivalently, mA-um).

**Impact on power grid design:**

For power grid segments shorter than the Blech length at the operating current density, the EM concern is relaxed or eliminated. This primarily applies to short via landing pads, short jog segments, and connections within bump pad structures.

However, most power grid stripes are much longer than the Blech length (typical Blech lengths are tens of micrometers, while power stripes can extend across the entire die, which is millimeters to centimeters long). Therefore, the Blech effect does not significantly relax the EM constraints on the main power grid stripes. It is most useful for relaxing rules on short segments in the via stack and local grid connections.

Some foundry EM rule decks include short-length corrections that automatically apply relaxed limits to segments below a specified length, simplifying the design process.

---

### Q10. How do temperature gradients across the die affect IR drop and EM analysis?

**Answer:**
Real SoCs have non-uniform temperature distributions due to the spatially varying power density. High-activity blocks (processor cores, GPUs) generate more heat and run hotter than low-activity blocks (memory controllers, I/O rings). Temperature gradients of 20 to 40 degrees Celsius across a die are common.

**Impact on IR drop:** Metal resistivity increases with temperature due to increased phonon scattering. For copper, the temperature coefficient of resistivity is approximately 0.39% per degree Celsius. A 30-degree temperature increase raises the resistivity (and thus the grid resistance) by about 12%. In hot regions, the grid resistance is higher and the IR drop is worse, creating a positive feedback effect: hot blocks draw more current (or experience more IR drop), which increases power loss, which further increases temperature.

Power integrity tools account for this by accepting a temperature map (from thermal analysis tools like ANSYS Icepak or Cadence Celsius) and computing temperature-dependent resistance for each grid segment:

```
R(T) = R(T_ref) * (1 + alpha * (T - T_ref))
```

where alpha is the temperature coefficient.

**Impact on EM:** EM lifetime is extremely sensitive to temperature through the exponential term in Black's equation. A 15-degree increase in temperature roughly halves the EM lifetime (for Ea = 0.7 eV). Therefore, hot spots are the most vulnerable locations for EM failure. An EM analysis that uses a single temperature (e.g., 105C everywhere) is conservative for cool regions but potentially optimistic for hot regions if the actual temperature exceeds 105C locally.

Best practice is to use a per-instance or per-region temperature map for EM analysis. The thermal map is typically generated by a thermal simulation tool that uses the power map (from power analysis) to compute the temperature distribution, accounting for the package thermal resistance, heat sink, and ambient conditions.

**Coupled thermal-electrical analysis:** The most accurate approach is an iteratively coupled analysis where the power map and temperature map are updated in alternating steps until convergence. The IR drop analysis provides the power dissipated in the grid (which is added to the circuit power), the thermal analysis computes the resulting temperature, and the updated temperature is fed back into the IR drop analysis for the next iteration.
