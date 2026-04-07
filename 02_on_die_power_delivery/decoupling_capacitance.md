# Decoupling Capacitance

This section covers on-die decoupling capacitance, including intrinsic gate capacitance, intentional MOS and MOM decap cells, thin-oxide reliability considerations, and decap optimization strategies.

---

### Q1. What are the sources of on-die decoupling capacitance?

**Answer:**
On-die decoupling capacitance comes from three primary sources:

**Intrinsic gate capacitance:** Every transistor in the design has a gate capacitance that provides inherent decoupling. When a CMOS gate is in a stable state (input either high or low), some transistors have their gates connected to VDD or VSS through the logic network, and the gate oxide capacitance between the gate and the channel acts as a small decoupling capacitor between the supply rails. The total intrinsic gate capacitance of a modern SoC can be substantial -- for a chip with billions of transistors, the cumulative gate capacitance can reach 100 to 500 nF. However, this capacitance varies with the logic state and switching activity, so it is not fully available at all times. Typically, 50 to 70 percent of the intrinsic gate capacitance is considered available for decoupling at any given instant.

**Intentional MOS decap cells:** These are standard cells specifically designed to provide decoupling capacitance. A typical MOS decap cell consists of a large NMOS transistor (or PMOS, or both) with its gate connected to VDD and its source and drain connected to VSS (for an NMOS decap). The gate oxide capacitance of this intentionally large transistor provides a dense source of decoupling. MOS decaps achieve the highest capacitance density of any on-die structure because they leverage the thin gate oxide for maximum capacitance per unit area. At advanced nodes, a thin-oxide MOS decap can provide 10 to 20 fF/um^2 of silicon area.

**MOM (metal-oxide-metal) capacitors:** Also called metal finger capacitors or vertical parallel plate (VPP) capacitors, MOM decaps use interdigitated metal fingers on multiple metal layers to create capacitance through the inter-metal dielectric. They do not use the gate oxide and therefore have no thin-oxide reliability concern, but they have lower capacitance density (typically 2 to 5 fF/um^2) than MOS decaps. MOM decaps are placed in the metal routing layers above the standard cells, potentially using routing resources that might otherwise carry signals.

A fourth, less common source is wiring capacitance -- the capacitance between power and ground wires that are routed adjacent to each other. This is usually a small contribution.

---

### Q2. How does a MOS decap cell work and what determines its capacitance?

**Answer:**
A MOS decap cell exploits the gate oxide capacitance of a MOSFET. In the simplest form, an NMOS transistor has its gate connected to VDD and its source, drain, and body connected to VSS. When VDD is applied to the gate, the NMOS operates in strong inversion (assuming VDD > Vth), and the gate-to-channel capacitance is approximately:

```
C_decap = Cox * W * L = (epsilon_ox / t_ox) * W * L
```

where Cox is the oxide capacitance per unit area, epsilon_ox is the permittivity of the gate dielectric (SiO2 or high-k), t_ox is the oxide thickness, W is the channel width, and L is the channel length.

For modern high-k metal gate processes at 7 nm, the equivalent oxide thickness (EOT) might be 0.7 to 1.0 nm, giving Cox approximately 30 to 50 fF/um^2 of gate area. A decap cell that is 5 um wide and 1 um long (5 um^2 of gate area) provides roughly 150 to 250 fF.

In practice, the effective decap capacitance is somewhat less than the theoretical Cox * W * L because:

- Part of the gate area is occupied by source/drain contacts and their spacing requirements
- The channel resistance (Ron) creates a distributed RC network that reduces the effective capacitance at high frequencies
- At very low VDD (near threshold), the transistor may not be fully inverted, reducing the capacitance

The frequency-dependent behavior is important: at low frequencies, the full C_decap is available. At high frequencies, the distributed RC nature of the channel causes the effective capacitance to roll off. The cutoff frequency depends on the channel length -- shorter channels have lower channel resistance and higher cutoff frequency. This is why decap cells use minimum or near-minimum channel lengths.

PMOS decap cells (gate to VSS, source/drain/body to VDD) are also used, often in combination with NMOS decaps to balance the use of both NFET and PFET device types and to avoid N-well spacing issues.

---

### Q3. What is the thin-oxide reliability concern with MOS decap cells?

**Answer:**
MOS decap cells have their gate oxide continuously stressed at the full VDD voltage. Unlike logic transistors whose gate voltage toggles between 0 and VDD, the decap transistor gate is always at VDD (for NMOS decaps) or always at VSS (for PMOS decaps), meaning the oxide experiences a constant DC stress.

The two primary reliability mechanisms are:

**Time-dependent dielectric breakdown (TDDB):** Under constant voltage stress, the gate oxide gradually degrades as traps and defects accumulate in the dielectric. Eventually, a percolation path of defects forms through the oxide, causing a hard breakdown (short circuit between gate and channel). The time to breakdown follows a Weibull distribution and depends strongly on the oxide thickness, applied voltage, temperature, and area. Because MOS decap cells have very large total gate area (potentially square millimeters of oxide across the chip), the probability of at least one decap experiencing breakdown is significantly higher than for individual logic transistors.

**Gate oxide leakage (tunneling current):** Thin gate oxides have significant quantum mechanical tunneling current that flows continuously through the decap. This leakage contributes to standby power consumption and generates heat. At advanced nodes with EOT below 1 nm, the tunneling current density can be 10 to 100 A/cm^2, and the total leakage from all decap cells can be a significant fraction of the chip's total leakage power.

**Mitigation strategies:**

- Using thick-oxide (IO) transistors for decaps: These have lower capacitance density but much better reliability and lower leakage. The tradeoff is 3 to 5 times less capacitance per unit area.
- De-rating the decap voltage: Designing the decap with a slightly higher threshold voltage so the oxide stress is Vdd - Vth instead of the full Vdd.
- Limiting total decap area: Budgeting the total thin-oxide decap area to meet the chip-level TDDB reliability target (e.g., less than 100 FIT for the entire chip).
- Using MOM decaps instead: MOM capacitors have no oxide reliability concern because the dielectric is the thick inter-metal oxide, but they have lower capacitance density.
- Hybrid approach: Using a combination of thin-oxide MOS decaps (for maximum density) and MOM decaps, with the total thin-oxide area controlled to meet reliability targets.

---

### Q4. How do MOM (metal-oxide-metal) decap cells differ from MOS decap cells?

**Answer:**
MOM capacitors create capacitance using the electric fields between closely spaced metal conductors on the BEOL (back-end-of-line) metal layers, rather than using the gate oxide of a transistor.

A typical MOM decap cell consists of interdigitated metal fingers on one or more metal layers. Alternating fingers are connected to VDD and VSS. The capacitance arises from three components: lateral coupling between adjacent fingers on the same layer, vertical coupling between fingers on different layers, and fringe fields. As metal pitch has decreased with technology scaling, the lateral coupling component has become dominant, and MOM capacitance density has improved.

| Property | MOS Decap | MOM Decap |
|----------|-----------|-----------|
| Capacitance density | 10-20 fF/um^2 | 2-5 fF/um^2 |
| Reliability risk | TDDB, high leakage | None (thick dielectric) |
| Leakage current | Significant (tunneling) | Negligible |
| Placement | In standard cell rows | In metal layers above cells |
| Routing impact | Uses silicon area | Uses metal routing tracks |
| Frequency range | Excellent (very low ESL) | Good (slightly higher ESL) |
| Process variation | Sensitive to Vth, tox | Sensitive to metal pitch, spacing |

MOM decaps are particularly attractive at advanced nodes where thin-oxide reliability is a growing concern and where the tighter metal pitches improve MOM capacitance density. Some designs use MOM decaps exclusively for reliability reasons, accepting the lower density and compensating with greater area allocation.

In many modern designs, both types are used: MOS decaps provide high-density decoupling in the standard cell area, while MOM decaps provide additional capacitance in the metal layers without consuming silicon area. The combination maximizes total on-die decoupling while managing reliability risk.

---

### Q5. How much on-die decoupling capacitance does a typical SoC need?

**Answer:**
The required on-die decoupling capacitance depends on the target impedance and the frequency range that must be covered by on-die decaps (above the effective range of the package-level capacitors).

Starting from the target impedance, the minimum on-die capacitance to meet the target at a given frequency f is:

```
C_die >= 1 / (2 * pi * f * Ztarget)
```

If the package decoupling is effective up to 200 MHz and on-die decaps must take over above that:

```
C_die >= 1 / (2 * pi * 200e6 * Ztarget)
```

For Ztarget = 1 mOhm:

```
C_die >= 1 / (2 * pi * 200e6 * 1e-3) = 796 nF
```

This is approximately 800 nF of on-die capacitance -- a substantial amount that requires significant silicon area.

In practice, modern high-performance SoCs have 100 to 500 nF of on-die decoupling capacitance (both intrinsic and intentional). A rough breakdown might be:

- Intrinsic gate capacitance: 50 to 200 nF (comes for free with the logic transistors)
- Intentional MOS decap cells: 50 to 300 nF (occupies 5 to 15 percent of the die area)
- MOM decap cells: 10 to 50 nF (uses metal routing resources)

The silicon area overhead of intentional decaps is substantial. At 15 fF/um^2, providing 200 nF of MOS decap requires:

```
Area = 200e-9 / 15e-15 = 13.3e6 um^2 = 13.3 mm^2
```

For a 100 mm^2 die, this is 13.3 percent of the die area dedicated to decoupling alone, which is a significant cost. The optimization of decap placement -- putting decaps where they are most needed and minimizing the total area while meeting the impedance target -- is an important part of physical design.

---

### Q6. Where should on-die decap cells be placed for maximum effectiveness?

**Answer:**
The placement of on-die decap cells directly determines their effectiveness, because a decap only helps if the current path from the decap to the switching circuits it supplies has low impedance (primarily low inductance at high frequencies and low resistance at low frequencies).

**Near high-activity regions:** Decap cells should be concentrated near blocks with high switching activity and large transient current demand, such as processor ALUs, cache arrays, and clock distribution buffers. These areas create the largest and fastest current transients.

**Distributed throughout the design:** While concentrating near hotspots is important, a baseline distribution of decap cells throughout the entire design is also necessary. Every region of the die has some switching activity, and even moderate current transients can cause problems if there is no nearby decoupling.

**In whitespace between macros:** After placement, there is typically unused space between standard cell blocks and hard macros. This whitespace is ideal for decap cell insertion because it does not displace logic cells or increase die area.

**Close to power bumps is less critical:** Decap cells near power bumps are less valuable because the bump already provides a low-impedance connection to the package-level capacitors. Decaps are most needed in areas far from bumps where the grid impedance is highest.

**Fill cells:** Most design flows use decap cells as filler cells to fill gaps in standard cell rows after placement. This automatically distributes decaps throughout the design. The standard cell library typically includes decap filler cells of various widths to efficiently fill gaps of different sizes.

**Optimization tools:** Advanced power integrity tools can perform decap optimization, which analyzes the voltage drop map and iteratively adds or removes decap cells to minimize the worst-case voltage drop for a given total decap area budget. This optimization considers the current density distribution, the grid topology, and the bump locations to determine the most effective decap placement.

---

### Q7. What is the ESR and ESL of on-die decoupling capacitors?

**Answer:**
On-die decoupling capacitors have extremely low parasitic resistance and inductance compared to discrete package or PCB capacitors, which is why they are essential for high-frequency decoupling.

**ESR of MOS decap cells:** The series resistance comes from the MOSFET channel resistance (Rds_on), the contact resistance between the metal and the source/drain, and the metal routing resistance within the cell. For a well-designed MOS decap cell, the ESR is typically 0.1 to 1 ohm per individual cell. However, when thousands of decap cells are distributed across the die and connected in parallel through the power grid, the effective total ESR is very low -- typically 0.1 to 10 mOhm for the aggregate on-die decoupling.

**ESL of MOS decap cells:** The inductance of an on-die decap is determined by the current loop area between VDD and VSS within the cell and its immediate grid connections. Because the cell is only a few micrometers tall and the metal layers are only a few micrometers apart, the loop area is extremely small. Individual cell ESL is on the order of 0.1 to 1 pH. The aggregate ESL of all on-die decaps in parallel is in the sub-femtohenry range, which means on-die decaps remain capacitive (below self-resonance) well into the multi-gigahertz range.

**MOM decap ESR and ESL:** MOM capacitors have similar ESL to MOS decaps (determined by the metal routing parasitics) and slightly higher ESR because the current path through the interdigitated fingers is longer. However, the values are still far lower than any discrete capacitor.

The extremely low ESL of on-die decaps is the key property that makes them essential. The series resonant frequency of an on-die decap cell with C = 100 fF and ESL = 0.5 pH is:

```
f_res = 1 / (2 * pi * sqrt(0.5e-12 * 100e-15)) = 1 / (2 * pi * sqrt(5e-26)) = 1 / (2 * pi * 7.07e-14) = 2.25 THz
```

This is far above any relevant switching frequency, confirming that on-die decaps behave as pure capacitors throughout the entire frequency range of interest.

---

### Q8. How does DVFS affect on-die decoupling requirements?

**Answer:**
Dynamic voltage and frequency scaling (DVFS) changes the operating voltage of a power domain, which directly affects the available decoupling capacitance and the noise budget.

**Voltage-dependent capacitance:** MOS decap capacitance depends on the gate-to-source voltage. When VDD is reduced during DVFS, the MOS decap is driven closer to its threshold voltage, and the inversion charge in the channel decreases. Below threshold, the capacitance drops dramatically because the transistor is no longer inverted. For designs that DVFS down to near-threshold voltages (0.4 to 0.5 V), the MOS decap capacitance can be 30 to 50 percent less than at the nominal VDD. This must be accounted for in the decoupling budget at the lowest voltage operating point.

**Tighter noise margin at lower voltage:** At reduced VDD, the absolute noise budget (Vdd * ripple%) is smaller. For example, at VDD = 0.5 V with 5% ripple, the budget is only 25 mV compared to 42.5 mV at 0.85 V. Although the current is also typically lower at reduced voltage (because frequency is reduced and activity decreases), the net effect on target impedance depends on the specific current and voltage at each DVFS operating point.

**Transient during voltage transitions:** When the DVFS controller changes the voltage setpoint, the regulator must either charge or discharge the total capacitance on the rail (including all decoupling capacitance). The transition time is:

```
t_transition = C_total * delta_V / I_charge
```

For a 500 nF total on-die capacitance and a 200 mV voltage step with a 2 A charge current:

```
t_transition = 500e-9 * 200e-3 / 2 = 50 ns
```

This is fast, but during the transition, the voltage is changing and the circuit operation may need to be paused or operated at a reduced frequency to avoid timing violations.

**Design implication:** The on-die decoupling must be verified at every DVFS operating point, not just the nominal voltage. The worst case may be the lowest voltage (tightest noise margin, least MOS decap capacitance) or the highest voltage (largest absolute current transients, highest EM stress).

---

### Q9. How do you estimate the on-die decoupling capacitance from the intrinsic gate capacitance of the design?

**Answer:**
The intrinsic gate capacitance is the sum of the gate capacitances of all transistors in the design, weighted by the probability that each transistor provides effective decoupling at any given time.

**Total gate capacitance:** This can be computed from the netlist and the technology parameters. For each transistor, the gate capacitance is approximately Cox * W * L (simplified; the actual value depends on the operating region and terminal voltages). Summing over all transistors gives the total gate capacitance of the design.

Power integrity tools (RedHawk, Voltus) extract this information from the design database. They read the placed and routed design, identify all transistor instances, compute their gate capacitance, and model the time-varying effective decoupling based on the switching activity of each gate.

**Effective decoupling fraction:** Not all gate capacitance is available for decoupling at any instant. A transistor provides decoupling when it is in a state where its gate capacitance bridges VDD and VSS through its channel. Specifically:

- An NMOS with gate at VDD (turned on): provides C_gate between VDD (gate) and VSS (source/drain), effective decoupling.
- An NMOS with gate at VSS (turned off): gate capacitance is very small (depletion cap), minimal decoupling.
- A PMOS with gate at VSS (turned on): provides C_gate between VDD (source/drain) and VSS (gate), effective decoupling.
- A PMOS with gate at VDD (turned off): minimal decoupling.

On average, about half the transistors in a random logic design are "on" at any time, but the fraction varies with the logic state. A conservative estimate is that 50 to 70 percent of the total gate capacitance is available as effective decoupling at any given instant.

For a design with 5 billion transistors at an average gate capacitance of 0.1 fF per transistor:

```
C_total_gate = 5e9 * 0.1e-15 = 500 nF
C_effective = 0.6 * 500 = 300 nF
```

This 300 nF of intrinsic decoupling is often the largest component of the total on-die decoupling budget.

---

### Q10. What is the impact of decoupling capacitor leakage on total chip power?

**Answer:**
MOS decap cells have gate oxide leakage current that flows continuously, contributing to the chip's static (standby) power consumption. This leakage is a direct consequence of quantum mechanical tunneling through the thin gate oxide.

The leakage current density depends exponentially on the oxide thickness and the applied voltage:

```
J_leak ~ A * (V/t_ox)^2 * exp(-B * t_ox / V)
```

where A and B are material-dependent constants, V is the gate voltage, and t_ox is the physical oxide thickness.

For a high-performance process at 7 nm with an EOT of approximately 0.8 nm, the gate leakage current density might be 10 to 50 A/cm^2 at nominal VDD. If the total MOS decap area is 10 mm^2 (0.1 cm^2):

```
I_leak_decap = 20 A/cm^2 * 0.1 cm^2 = 2 A
P_leak_decap = 2 A * 0.85 V = 1.7 W
```

This 1.7 W of decap leakage can be a significant fraction of the total chip leakage power, especially for mobile SoCs where total power budgets are 3 to 5 W. In some designs, decap leakage has been observed to be 20 to 30 percent of the total chip leakage.

**Mitigation strategies:**

- **Thick-oxide decaps:** Using I/O-voltage transistors (thicker oxide) for decaps reduces leakage by orders of magnitude but at the cost of 3 to 5 times lower capacitance density. This may be acceptable for blocks with moderate decoupling requirements.
- **MOM decaps:** No gate leakage at all, making them leakage-free alternatives. The lower density is offset by zero leakage contribution.
- **Power-gated decaps:** In power-gated domains, the decaps are powered off with the rest of the block, so their leakage does not contribute to standby power.
- **High-Vth decap cells:** Using high-threshold transistors for decaps slightly reduces the effective capacitance (because the overdrive voltage is lower) but significantly reduces leakage (exponential sensitivity to Vth).
- **Leakage-density tradeoff optimization:** The physical design tool can be configured to use a mix of standard-Vth and high-Vth decap cells, or a mix of MOS and MOM decaps, to optimize the tradeoff between decoupling density and leakage power.
