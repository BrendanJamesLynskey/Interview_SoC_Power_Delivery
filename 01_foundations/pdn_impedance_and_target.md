# PDN Impedance and Target Impedance

This section covers the theory and practice of PDN impedance analysis, target impedance derivation, frequency-domain impedance budgeting, and anti-resonance management.

---

### Q1. Derive the target impedance formula and explain each term.

**Answer:**
The target impedance formula establishes the maximum PDN impedance that ensures supply voltage noise remains within specification. It is derived directly from Ohm's law applied to the worst-case scenario.

The specification states that the supply voltage noise (peak deviation from nominal) must not exceed a fraction of the nominal supply voltage:

```
V_noise_max = Vdd * ripple%
```

The worst-case voltage noise occurs when the maximum transient current step Imax flows through the PDN impedance Z:

```
V_noise = Z * Imax
```

Setting V_noise equal to V_noise_max and solving for Z gives the target impedance:

```
Ztarget = Vdd * ripple% / Imax
```

**Vdd** is the nominal supply voltage of the rail being analyzed (e.g., 0.85 V for a core rail). Lower Vdd makes the target tighter because the absolute noise margin is smaller.

**ripple%** is the fractional voltage tolerance, typically 3 to 10 percent depending on the rail and the sensitivity of the circuits it powers. Analog supplies (PLL, SerDes) may require tighter specifications (2 to 3 percent) while digital logic may tolerate 5 to 10 percent.

**Imax** is the maximum expected transient current step. This is not the average current or the peak steady-state current; it is the worst-case change in current that can occur within a short time period. For a processor, this might be the current swing between idle and full activity when a clock domain is ungated.

For a practical example: Vdd = 0.8 V, ripple% = 5% (0.05), Imax = 40 A yields Ztarget = 0.8 * 0.05 / 40 = 1 mOhm. This sub-milliohm target must be maintained from DC through the highest relevant switching frequency, illustrating the severity of the design challenge in modern SoCs.

---

### Q2. How is the PDN impedance profile constructed and interpreted?

**Answer:**
The PDN impedance profile is a plot of the magnitude of the PDN impedance (in ohms or milliohms) as a function of frequency (typically from 1 Hz or 1 kHz up to 1 GHz or beyond), viewed from the load (the switching transistors on the die). It is the single most important characterization of PDN performance.

To construct this profile, the complete PDN -- including the VRM output impedance, PCB planes and decoupling capacitors, package substrate and its decaps, and the on-die grid and decoupling -- is modeled as a network of RLC elements. The impedance seen looking into this network from the die is then computed as a function of frequency, typically using AC analysis in a circuit simulator (HSPICE, Spectre) or an electromagnetic field solver (Sigrity PowerSI, ANSYS SIwave).

The profile typically shows several distinct regions. At very low frequencies (below the VRM bandwidth), the impedance is set by the VRM output impedance and its feedback loop, and is typically very low (milliohms). As frequency increases above the VRM bandwidth, the impedance rises due to the increasing impedance of the VRM output inductor. Bulk capacitors on the PCB then take over, bringing the impedance back down. At higher frequencies, the bulk caps become inductive and the impedance rises again until ceramic MLCCs take over. This pattern of handoffs continues through the package decaps and finally the on-die decaps.

At each handoff between stages, there is a risk of anti-resonance peaks where the inductance of the lower-frequency stage resonates with the capacitance of the higher-frequency stage. These peaks can significantly exceed the target impedance. The goal of PDN design is to keep the entire impedance curve below the target impedance line, paying special attention to these transition regions.

---

### Q3. What causes anti-resonance in a PDN and how is it mitigated?

**Answer:**
Anti-resonance occurs when the parasitic inductance of one decoupling stage forms a parallel resonant (tank) circuit with the capacitance of an adjacent stage. At the resonant frequency, the parallel combination of the inductor and capacitor presents a very high impedance -- theoretically infinite for lossless components and practically limited only by the resistive losses (ESR) in the components.

Consider two capacitors of different values connected in parallel to a power rail. Each has its own ESR and ESL. At low frequencies, the larger capacitor dominates (low impedance). At high frequencies, the smaller capacitor dominates. Between their individual series resonant frequencies, there is a frequency where the larger capacitor has become inductive (above its series resonance) and the smaller capacitor is still capacitive (below its series resonance). The parallel combination of this inductance and capacitance creates an anti-resonance peak.

The impedance at anti-resonance can be estimated as:

```
Z_anti = sqrt(L_large / C_small)
```

where L_large is the ESL of the larger capacitor and C_small is the capacitance of the smaller capacitor. This can easily be tens or hundreds of milliohms, far exceeding a sub-milliohm target.

Mitigation strategies include:

**Adding intermediate capacitor values:** Filling in the frequency gap between two stages with capacitors of intermediate value reduces the impedance at the transition.

**Increasing ESR (adding damping):** Resistive losses in the capacitors damp the resonance. Some designers intentionally use capacitors with moderate ESR or add series resistance to damp anti-resonance peaks, at the cost of slightly higher impedance at the capacitor's series resonant frequency.

**Overlapping frequency coverage:** Choosing capacitor values and quantities so that the effective frequency ranges of adjacent stages overlap ensures there is no frequency gap where impedance is uncontrolled.

**Reducing parasitic inductance:** Using low-inductance capacitor mounting (short, wide traces, many vias) lowers the frequency at which capacitors become inductive, extending their useful range.

---

### Q4. How do you determine the frequency range over which a particular decoupling capacitor is effective?

**Answer:**
A real decoupling capacitor can be modeled as a series RLC circuit: the nominal capacitance C, the equivalent series resistance ESR, and the equivalent series inductance ESL. The impedance of this model is:

```
Z(f) = ESR + j*(2*pi*f*ESL - 1/(2*pi*f*C))
```

The capacitor is most effective (lowest impedance) at its series resonant frequency:

```
f_res = 1 / (2 * pi * sqrt(ESL * C))
```

At this frequency, the inductive and capacitive reactances cancel, and the impedance equals ESR. Below f_res, the capacitor behaves capacitively (impedance decreases with frequency). Above f_res, it behaves inductively (impedance increases with frequency).

The effective frequency range is typically defined as the band over which the capacitor's impedance remains below the target impedance. Below f_res, the impedance is 1/(2*pi*f*C), and this falls below the target at:

```
f_low = 1 / (2 * pi * C * Ztarget)
```

Above f_res, the impedance is 2*pi*f*ESL, and this exceeds the target at:

```
f_high = Ztarget / (2 * pi * ESL)
```

The effective range is approximately f_low to f_high. For a 10 uF MLCC with ESL = 0.5 nH and ESR = 5 mOhm, and a target impedance of 10 mOhm:

- f_res = 1 / (2*pi*sqrt(0.5e-9 * 10e-6)) = 2.25 MHz
- f_low = 1 / (2*pi * 10e-6 * 10e-3) = 1.59 MHz
- f_high = 10e-3 / (2*pi * 0.5e-9) = 3.18 MHz

This relatively narrow effective range illustrates why multiple capacitor values are needed to cover a broad frequency band. Paralleling many identical capacitors reduces the effective ESR and ESL proportionally, widening the effective range.

---

### Q5. How does the number of parallel capacitors affect the PDN impedance?

**Answer:**
Placing N identical capacitors in parallel has the effect of multiplying the capacitance by N while dividing both the ESR and ESL by N. The resulting impedance at any frequency is 1/N times the impedance of a single capacitor:

```
C_total = N * C
ESR_total = ESR / N
ESL_total = ESL / N
```

The series resonant frequency remains unchanged because both L and C scale by the same factor:

```
f_res = 1 / (2 * pi * sqrt((ESL/N) * (N*C))) = 1 / (2 * pi * sqrt(ESL * C))
```

However, the minimum impedance at resonance drops to ESR/N, and both the lower and upper bounds of the effective frequency range shift favorably:

- f_low = 1 / (2*pi * N*C * Ztarget) -- decreases, extending low-frequency coverage
- f_high = Ztarget / (2*pi * ESL/N) = N * Ztarget / (2*pi * ESL) -- increases, extending high-frequency coverage

This is the fundamental reason why high-current SoC designs use many (often hundreds) of decoupling capacitors. For example, a server processor package might have 200 to 400 MLCCs to achieve the required sub-milliohm impedance.

It is important to note that this ideal scaling assumes all capacitors are identically placed with equal current sharing. In practice, placement variations, trace routing, and via inductance cause the actual improvement to be somewhat less than the ideal 1/N scaling. Additionally, above a certain number of capacitors, the mounting inductance (the inductance of the PCB vias and traces connecting each capacitor to the plane) becomes the limiting factor, and adding more capacitors yields diminishing returns.

---

### Q6. What is the difference between self-resonance and parallel resonance (anti-resonance)?

**Answer:**
Self-resonance (series resonance) is the frequency at which a single capacitor's capacitive reactance equals its inductive reactance (from ESL), causing the two to cancel and the impedance to drop to its minimum value of ESR. At series resonance, the capacitor provides maximum decoupling effectiveness. The formula is f_series = 1 / (2*pi*sqrt(ESL*C)). This is a property of a single component.

Parallel resonance (anti-resonance) is a phenomenon that occurs in a network of two or more reactive elements connected in parallel. When the inductive impedance of one element equals the capacitive impedance of another at some frequency, the parallel combination presents a very high impedance at that frequency (limited only by losses). This is an emergent property of the network, not of any single component.

In a PDN context, anti-resonance commonly occurs between: (a) two capacitors of different values connected in parallel, at a frequency between their respective series resonant frequencies; (b) the output inductance of the VRM and the bulk decoupling capacitance; (c) the package spreading inductance and the on-die decoupling capacitance.

The key distinction is that self-resonance is beneficial (it represents the frequency of maximum effectiveness), while anti-resonance is harmful (it represents a frequency of minimum effectiveness, where the PDN impedance spikes). Good PDN design maximizes the benefit of series resonance while controlling anti-resonance through proper capacitor value selection, damping, and ensuring continuous impedance coverage across frequency.

---

### Q7. How is the PDN impedance budget allocated across the VRM, PCB, package, and die?

**Answer:**
Impedance budgeting is the process of assigning a maximum impedance contribution to each stage of the PDN hierarchy such that the total impedance seen at the die remains below the target. Since the stages are effectively in parallel (each can source current independently at certain frequencies), the budget is typically allocated by frequency range rather than by dividing a single impedance value.

A common approach is:

**DC to ~50 kHz (VRM):** The VRM's control loop maintains low output impedance within its bandwidth. The VRM must present impedance below Ztarget at these frequencies. This is typically straightforward for a well-designed multiphase buck converter.

**50 kHz to ~10 MHz (PCB bulk and ceramic caps):** Bulk tantalum/polymer capacitors and large-value MLCCs on the PCB provide low impedance in this range. The impedance budget for this range is the full Ztarget, as the VRM is becoming inductive and the package/die caps are not yet effective.

**10 MHz to ~300 MHz (package decaps and planes):** Package-mounted MLCCs and the package plane capacitance must hold impedance below Ztarget. The combined impedance of PCB caps (now inductive) in parallel with package caps must remain below the target.

**Above 300 MHz (on-die decaps):** On-die MOS/MOM capacitors and intrinsic gate capacitance provide decoupling. The budget here is again Ztarget, with the die-level capacitance responsible for suppressing high-frequency noise.

At each transition frequency, the handoff between stages must be smooth. The inductance of the lower-frequency stage (which increases with frequency) must be low enough that it does not create an anti-resonance peak above the target when combined with the capacitance of the next stage. This is why package spreading inductance is a critical parameter -- it determines the impedance floor for the handoff between package-level and die-level decoupling.

Some designers allocate a margin at each stage (e.g., each stage must achieve 0.5 * Ztarget in its primary frequency range) to provide margin for manufacturing variation, temperature effects, and modeling uncertainty.

---

### Q8. How does the PDN target impedance change with technology scaling?

**Answer:**
Technology scaling has consistently driven PDN target impedance lower, making power delivery more challenging with each process generation. The key trends are:

**Lower supply voltage:** As transistor gate oxides become thinner with each node, the maximum supply voltage decreases (from 1.8 V at 180 nm, to 1.0 V at 65 nm, to 0.75 V at 7 nm, and approaching 0.5 V at 3 nm and below). Since Ztarget is proportional to Vdd, the target impedance decreases proportionally for the same ripple budget and current.

**Higher current:** Smaller transistors are packed at higher density, and despite lower per-transistor current, the total chip current has generally increased (from a few amperes at 180 nm to over 100 A for high-performance designs at 5 nm). Since Ztarget is inversely proportional to Imax, higher current directly reduces the target.

**Higher frequencies:** Faster transistors switch more quickly, pushing the relevant frequency range higher. The PDN must maintain low impedance out to higher frequencies, which is increasingly difficult because parasitic inductance becomes more significant at higher frequencies.

**Quantitatively**, the target impedance has dropped from roughly 10 to 50 mOhm at 180 nm nodes to below 0.5 mOhm at leading-edge 3 nm and 5 nm designs. This represents a roughly 100x reduction in target impedance over about 20 years of scaling.

The response to this trend has been multi-pronged: thicker and wider on-die power grids, more C4 bumps and microbumps dedicated to power, more package-level and PCB-level decoupling capacitors, integrated voltage regulators to bring regulation closer to the load, and sophisticated co-design of the die, package, and board PDN. Despite these efforts, power delivery remains one of the most challenging aspects of advanced SoC design and consumes an increasingly large fraction of the design effort and silicon area.

---

### Q9. What is the role of ESR in controlling anti-resonance peaks?

**Answer:**
ESR (equivalent series resistance) provides resistive damping in the PDN. While ESR is generally undesirable because it increases the minimum impedance of a capacitor at its series resonant frequency (and causes I-squared-R power dissipation), it plays a beneficial role in damping anti-resonance peaks.

At an anti-resonance frequency, the impedance peak is determined by the ratio of inductance to capacitance and is limited by the losses in the circuit. For a parallel resonance between an inductor L and a capacitor C, the peak impedance is:

```
Z_peak = Q * sqrt(L/C)
```

where Q is the quality factor. The quality factor is inversely proportional to the resistance in the circuit:

```
Q = (1/ESR) * sqrt(L/C)
```

Therefore:

```
Z_peak = (1/ESR) * (L/C)
```

Higher ESR means lower Q, which means a lower (better) anti-resonance peak. This is why some PDN designers deliberately choose capacitors with moderate ESR or add small series resistors to specific capacitors to damp problematic resonances.

However, there is a tradeoff. Higher ESR raises the minimum impedance at series resonance, which must still be below the target. The optimal ESR depends on whether the bottleneck is the series resonance impedance or the anti-resonance peak impedance. In many modern designs where the target impedance is extremely low (sub-milliohm), the ESR must also be very low, and damping is achieved instead through careful capacitor value selection that avoids large anti-resonance peaks.

Some specialized capacitors (controlled-ESR or damping capacitors) are designed with ESR values optimized for a specific damping application rather than minimum ESR.

---

### Q10. How do you validate a PDN impedance design before silicon is fabricated?

**Answer:**
PDN impedance validation before tape-out is a multi-step process that combines simulation, modeling, and design rule checking.

**Chip-level power grid analysis:** Tools such as ANSYS RedHawk or Cadence Voltus simulate the on-die power grid using the physical design database (DEF/LEF), including all metal layers, vias, and bump connections. They compute the impedance looking into the die and verify that voltage drop meets specification under both static and dynamic conditions.

**Package model extraction:** The package substrate is modeled using a 3D electromagnetic solver (Cadence Sigrity, ANSYS HFSS/SIwave) or approximated with a distributed RLC network. S-parameter models are extracted that capture the package plane impedance, bump resistance and inductance, and via parasitics.

**Board-level simulation:** The PCB stackup, power plane geometry, and decoupling capacitor network are modeled (Sigrity PowerSI, ANSYS SIwave) and the impedance is extracted from the BGA pad locations.

**Co-simulation:** The die, package, and board models are combined in a single simulation (often in HSPICE or within the power integrity tool's co-simulation framework) to compute the total PDN impedance profile as seen at the transistor level. This is compared against the target impedance curve.

**Sensitivity analysis:** Key parameters (capacitor tolerance, ESR/ESL variation, package inductance uncertainty, VRM output impedance) are varied to ensure the design meets specification across expected manufacturing variation.

**Design rule checks:** Automated checks verify that all metal stripes meet EM current density limits, that power grid coverage meets density requirements, and that decoupling capacitor placement follows guidelines.

**Silicon correlation:** After the first silicon is fabricated, the actual PDN impedance is measured (using VNA-based methods or on-die impedance sensing circuits) and compared to the pre-silicon models. Discrepancies are fed back to improve models for the next design.
