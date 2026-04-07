# Frequency Domain Analysis

This section covers frequency-domain methods for PDN analysis, including impedance profile construction, anti-resonance identification, and cavity resonance in power planes.

---

### Q1. What is a PDN impedance profile and how is it constructed?

**Answer:**
A PDN impedance profile is a plot of the magnitude of the PDN self-impedance (in ohms, typically milliohms) as a function of frequency, viewed from the load (the switching transistors on the die). It is the most fundamental characterization of PDN quality and directly predicts the voltage noise that will be produced by any current transient.

The profile is constructed by modeling the complete PDN as a network of electrical elements and computing the driving-point impedance at the die load port. The construction involves several steps:

First, each PDN stage is modeled: the VRM as a voltage source with frequency-dependent output impedance, the PCB planes and decoupling capacitors as RLC networks extracted from electromagnetic simulation or lumped models, the package substrate as S-parameters from full-wave extraction, and the on-die grid and decoupling as an RLC mesh from the power integrity tool.

Second, these models are assembled into a single circuit and an AC analysis is performed. The simulator injects a small AC current at the die port and sweeps the frequency, computing the resulting AC voltage at each frequency. The ratio V(f)/I(f) gives Z(f), the impedance profile.

The resulting profile typically shows several distinct features: a low-impedance region at DC and very low frequencies (VRM regulating), a rising impedance as the VRM becomes inductive, dips at each decoupling capacitor group's resonant frequency, anti-resonance peaks between capacitor groups, and a final rise at the highest frequencies above the on-die decoupling effectiveness.

The profile is compared against the target impedance line. Any frequency where the profile exceeds the target represents a potential noise violation that requires design correction.

---

### Q2. How do you interpret anti-resonance peaks in an impedance profile?

**Answer:**
Anti-resonance peaks appear as local maxima (upward spikes) in the impedance profile. They occur at frequencies where the inductance of one PDN stage resonates with the capacitance of an adjacent stage, creating a parallel resonant circuit with high impedance.

Each anti-resonance peak is characterized by its frequency, magnitude, and quality factor (Q, which determines the sharpness of the peak). A narrow, tall peak indicates a high-Q resonance with little damping, while a broad, lower peak indicates a well-damped resonance.

To interpret a peak: identify which two PDN elements are resonating by examining the impedance contributions of each stage at the peak frequency. The stage that is inductive at the peak frequency is the "inductor" of the resonance, and the stage that is capacitive is the "capacitor." Knowing this tells you which elements to modify.

Common anti-resonance locations:
- 10-100 kHz: VRM output inductance resonating with bulk capacitors
- 1-10 MHz: bulk capacitor ESL resonating with ceramic MLCC capacitance
- 10-100 MHz: PCB MLCC ESL resonating with package capacitors or plane capacitance
- 100 MHz - 1 GHz: package spreading inductance resonating with on-die decoupling

Mitigation strategies depend on the specific resonance:
- Add capacitors of intermediate value to fill the frequency gap
- Increase ESR (damping) in one or both resonating elements
- Reduce inductance of the lower-frequency element (lower ESL caps, more parallel caps)
- Increase capacitance of the higher-frequency element

The goal is to keep all anti-resonance peaks below the target impedance line.

---

### Q3. What is cavity resonance in power planes and how is it analyzed?

**Answer:**
Cavity resonance occurs in the parallel-plate waveguide formed by adjacent power and ground planes in a PCB or package substrate. The plane pair behaves as a 2D resonant cavity, similar to a microwave cavity resonator, supporting standing wave patterns at specific resonant frequencies.

The resonant frequencies of a rectangular cavity are:

```
f_mn = (c / (2 * sqrt(epsilon_r))) * sqrt((m/a)^2 + (n/b)^2)
```

where a and b are the plane dimensions, m and n are mode indices, c is the speed of light, and epsilon_r is the dielectric constant.

For a 50 mm x 50 mm PCB plane pair with epsilon_r = 4.2:

```
f_10 = (3e8 / (2*sqrt(4.2))) * (1/0.05) = (7.32e7) * 20 = 1.46 GHz
f_01 = same (square plane)
f_11 = sqrt(2) * f_10 = 2.07 GHz
```

At resonant frequencies, the impedance at certain locations on the plane can be very high (the plane acts as an open circuit at voltage antinodes) while at other locations it can be very low (voltage nodes). If a noise source (such as a switching circuit) is located at a voltage antinode and a sensitive victim (such as a PLL) is also at an antinode, the noise coupling between them is maximized at the resonant frequency.

Analysis methods:
- **Analytical formulas:** For simple rectangular planes, the resonant frequencies and mode shapes can be computed analytically.
- **Cavity model:** The plane pair is modeled as a network of transmission line segments, and the impedance is computed as a sum over all cavity modes.
- **Full-wave simulation:** EM solvers (SIwave, PowerSI) compute the exact impedance at any port location, capturing irregular geometries, cutouts, and via perforations.

Mitigation:
- Decoupling capacitors placed at strategic locations can suppress specific modes
- Lossy dielectric materials damp the resonance
- Plane perforations (from via arrays) increase the effective loss
- Stitching vias around the plane perimeter create boundary conditions that shift or dampen modes

---

### Q4. How do S-parameters relate to PDN impedance analysis?

**Answer:**
S-parameters (scattering parameters) are a frequency-domain characterization of an electrical network in terms of incident and reflected power waves. They are the standard format for representing package and PCB PDN models extracted from electromagnetic simulation.

For PDN analysis, the most important S-parameter is S11, the reflection coefficient at a single port. The PDN impedance at that port is related to S11 by:

```
Z(f) = Z0 * (1 + S11(f)) / (1 - S11(f))
```

where Z0 is the reference impedance (typically 50 ohms for measurement, but often 1 ohm or the target impedance for PDN analysis).

For multi-port PDN analysis (e.g., a package with multiple die bump ports and multiple BGA ports), the full S-parameter matrix captures the impedance at each port and the coupling between ports. The self-impedance at port i is derived from S_ii, and the transfer impedance between ports i and j (which indicates noise coupling) is derived from S_ij.

S-parameters are useful because:
- They accurately capture all frequency-dependent effects (skin effect, dielectric loss, resonance)
- They can be combined with other models in circuit simulators (SPICE can import Touchstone S-parameter files)
- They can be measured on physical hardware using a VNA and compared to simulation

The S-parameter model of the package is typically a multi-port model with ports at each power/ground BGA ball pair and each die bump pad pair. This model is then connected to the PCB model on one side and the die model on the other for complete chip-package-board co-simulation.

---

### Q5. What is transfer impedance and why does it matter for noise coupling?

**Answer:**
Transfer impedance (Z_ij or Z_21) measures how much voltage noise appears at one location on the PDN when a current is injected at another location. It is the off-diagonal element of the impedance matrix:

```
Z_21(f) = V_2(f) / I_1(f)
```

where I_1 is the current injected at port 1 and V_2 is the voltage perturbation at port 2, with all other ports open.

Transfer impedance matters because it quantifies the noise coupling between different circuits that share the same PDN. For example:
- A noisy digital processor core (port 1) generating supply noise
- A sensitive PLL or ADC (port 2) that could be affected by that noise

If the transfer impedance Z_21 is high at a particular frequency, then even moderate current fluctuations from the processor will cause significant voltage noise at the PLL supply.

Transfer impedance depends on the physical distance between the two ports, the PDN topology (plane geometry, via connections), and the decoupling capacitor distribution. At low frequencies, where the plane behaves as a lumped element, the transfer impedance is approximately equal to the self-impedance (the entire plane is at the same voltage). At high frequencies, the plane exhibits distributed behavior, and the transfer impedance can be much lower than the self-impedance for widely separated ports (good) or even higher than the self-impedance at cavity resonance frequencies (bad).

Reducing transfer impedance between sensitive and noisy circuits:
- Place decoupling capacitors near the sensitive circuit to provide a local low-impedance supply
- Increase the physical separation between noisy and sensitive circuits on the die and package
- Use separate power domains (separate planes) for noisy and sensitive circuits
- Add filtering (ferrite beads, LDO post-regulators) between the domains

---

### Q6. How do you perform chip-package-board co-simulation for PDN analysis?

**Answer:**
Chip-package-board co-simulation combines the models of all three PDN stages into a single analysis to capture their mutual interactions and produce an accurate impedance profile at the transistor level.

**Step 1: Extract the die model.** The on-die power grid is modeled in the power integrity tool (RedHawk, Voltus) as an RLC mesh. The model includes the grid resistance and inductance, on-die decoupling capacitance (both intrinsic and intentional), and the bump pad locations. The die model may be exported as a SPICE netlist or as an impedance matrix.

**Step 2: Extract the package model.** The package substrate is modeled in an EM solver (Sigrity PowerSI, ANSYS SIwave) to produce an S-parameter or SPICE model that captures the package planes, vias, bumps, and mounted MLCCs. Ports are defined at each BGA ball and each die bump pad.

**Step 3: Extract the PCB model.** Similarly, the PCB power planes and decoupling network are modeled and extracted as S-parameters or a SPICE netlist. Ports are at the BGA pad locations and the VRM output.

**Step 4: Extract the VRM model.** The VRM output impedance is modeled as a frequency-dependent impedance (often a SPICE model from the regulator vendor or measured data).

**Step 5: Connect the models.** The VRM model drives the PCB model at the VRM output port. The PCB model connects to the package model at the BGA port locations. The package model connects to the die model at the bump pad locations. All connections are made at corresponding power/ground port pairs.

**Step 6: Run analysis.** AC analysis sweeps frequency and computes the impedance at any die port (or all die ports). Transient analysis applies realistic current waveforms at the die ports and computes the time-domain voltage response.

**Challenges:**
- Model sizes can be very large (thousands of ports), requiring model order reduction or adaptive port grouping
- Different tools use different model formats, requiring translation and validation
- The accuracy of the co-simulation is limited by the accuracy of each individual model
- Convergence can be difficult for models with very high Q resonances

Modern PDN analysis tools (ANSYS RedHawk-SC Electrothermal, Cadence Voltus-Fi) integrate chip-package co-simulation within a single environment, simplifying the setup.

---

### Q7. What is the frequency range of interest for SoC PDN analysis?

**Answer:**
The frequency range of interest spans from DC to the highest frequency at which significant current transients occur. For modern SoCs, this range is typically DC to 1-5 GHz.

**DC to 1 kHz:** VRM regulation. The VRM maintains low impedance through active feedback. Current at these frequencies represents slow workload changes.

**1 kHz to 1 MHz:** Low-frequency transients from workload changes, DVFS transitions, and power gating events. Bulk and polymer capacitors on the PCB provide decoupling.

**1 MHz to 100 MHz:** Switching activity at the clock frequency and its low-order harmonics. PCB ceramic caps and package MLCCs provide decoupling. This is often the most challenging range for PDN design due to anti-resonance peaks.

**100 MHz to 1 GHz:** High-frequency switching noise from individual clock edges and logic transitions. Package MLCCs and on-die decoupling provide coverage. The package spreading inductance is the key impedance-limiting factor.

**1 GHz to 5 GHz:** Very high-frequency noise from the fastest signal edges and local switching events. Only on-die decoupling is effective at these frequencies. The on-die grid inductance and decap ESL determine the impedance.

**Above 5 GHz:** Typically not a concern for PDN design because the current spectral content at these frequencies is very small (the amplitude of current harmonics rolls off with frequency). However, some specific concerns (cavity resonance, localized noise hotspots) may extend to higher frequencies.

The exact upper frequency limit depends on the clock frequency, the risetime of the switching events, and the sensitivity of the circuits to high-frequency supply noise. A conservative approach analyzes up to the third harmonic of the clock frequency or up to 1/(pi * t_rise) of the fastest edge, whichever is higher.

---

### Q8. How do you use an impedance profile to predict voltage noise?

**Answer:**
The impedance profile, combined with the current spectrum, predicts the voltage noise spectrum through Ohm's law applied in the frequency domain:

```
V_noise(f) = Z(f) * I(f)
```

where V_noise(f) is the voltage noise spectral density, Z(f) is the PDN impedance, and I(f) is the current spectral density, all as functions of frequency.

To predict the peak time-domain voltage noise from the impedance profile:

**Step 1:** Obtain or estimate the current waveform I(t) at the load. This can be from gate-level simulation, statistical estimation, or a simplified model (e.g., a trapezoidal current pulse with known amplitude, risetime, and duration).

**Step 2:** Compute the Fourier transform I(f) of the current waveform.

**Step 3:** Multiply I(f) by Z(f) at each frequency to get V_noise(f).

**Step 4:** Compute the inverse Fourier transform of V_noise(f) to get the time-domain voltage noise v_noise(t).

**Step 5:** The peak of v_noise(t) is the worst-case voltage droop or overshoot.

For a quick estimate using a step current transient (I_step with risetime t_r):

The current spectrum has significant energy from DC to approximately f_knee = 1/(pi * t_r). For a 40 A step with t_r = 1 ns: f_knee = 318 MHz.

The worst-case noise is bounded by the maximum impedance in the frequency range from DC to f_knee, multiplied by the current step:

```
V_noise_max <= max(Z(f)) * I_step, for f in [0, f_knee]
```

If the impedance profile is flat at 1 mOhm from DC to 318 MHz, then V_noise = 1 mOhm * 40 A = 40 mV. If there is an anti-resonance peak of 3 mOhm at 50 MHz, the noise at that frequency would be 3 mOhm * (I component at 50 MHz), which may or may not exceed 40 mV depending on the current spectrum at 50 MHz.

---

### Q9. What is the effect of probe placement on PDN impedance measurement?

**Answer:**
When measuring PDN impedance on physical hardware, the probe type, position, and connection method significantly affect the measured result.

**Probe type:** VNA (vector network analyzer) measurements using two-port shunt-through method provide the best accuracy for low-impedance measurements (sub-milliohm). One-port reflection measurements lose accuracy below about 100 mOhm because the small reflection coefficient is difficult to measure precisely against the 50-ohm reference.

**Probe position:** The impedance measured at one location on the PDN differs from the impedance at another location, especially at high frequencies where the distributed nature of the planes creates position-dependent impedance. Measuring at the BGA pad gives the PCB+package combined impedance. Measuring at a capacitor pad gives the local impedance at that point. The impedance at the die (the actual operating point) can only be measured using on-die test structures or inferred from models.

**Probe parasitics:** Coaxial probe tips add inductance (typically 0.5 to 2 nH) and resistance that contaminate the measurement. This is particularly problematic for sub-milliohm measurements. Careful calibration and de-embedding techniques are used to remove probe effects.

**Ground loop:** The measurement probe must make a low-inductance connection between VDD and VSS to accurately measure the impedance. A solder connection directly to the pads is ideal. A spring-loaded probe tip with a short ground connection is acceptable. A long ground lead (more than a few millimeters) adds inductance that dominates the measurement at high frequencies.

**Board loading:** The measurement setup may change the PDN behavior if the probe itself draws significant current or if the VNA output impedance interacts with the PDN impedance.

Best practices:
- Use the two-port shunt-through method for sub-milliohm measurements
- Calibrate the VNA to the probe tip plane (SOLT or TRL calibration)
- Minimize ground loop area in the probe connection
- Measure at multiple locations to understand the spatial impedance variation
- Compare measurements to simulation and investigate discrepancies

---

### Q10. How do electromagnetic simulation tools extract the PDN impedance of a package or PCB?

**Answer:**
Electromagnetic simulation tools solve Maxwell's equations for the physical geometry of the package or PCB to compute the frequency-dependent electrical response at defined port locations.

**Geometry import:** The tool imports the physical layout (Gerber files for PCB, substrate design files for package) including all metal layers, dielectric stackup, via locations, and component pad locations.

**Material assignment:** Each metal layer is assigned conductivity (copper: 5.8e7 S/m) and each dielectric layer is assigned permittivity and loss tangent. Temperature-dependent material properties may be specified.

**Port definition:** Ports are defined at the locations where the impedance will be computed or where the model will be connected to other models. For a package, ports are at each BGA ball pair and each die bump pair. Each port is defined between a VDD pad and a nearby VSS pad.

**Meshing:** The tool discretizes the geometry into a mesh of elements. 2D methods (method of moments applied to the plane pair) mesh the plane surfaces. 3D methods (FEM, FDTD) mesh the entire volume. Finer mesh near vias, ports, and edges provides accuracy where the fields vary rapidly.

**Solution:** The tool solves for the fields at each frequency point in the specified range. For method of moments: surface currents are computed. For FEM: the 3D field distribution is computed. The port voltages and currents are extracted from the field solution.

**Post-processing:** The solver computes the impedance matrix (Z-parameters) or scattering matrix (S-parameters) from the port voltages and currents. The self-impedance Z_ii at each port is the PDN impedance at that location. The result is exported as a Touchstone file (.s2p, .snp) for use in circuit simulation.

Common tools and their methods:
- Sigrity PowerSI: hybrid FEM/MoM for package and PCB
- ANSYS SIwave: 2D method of moments for plane pairs
- ANSYS HFSS: 3D FEM for complex 3D structures
- CST Studio: 3D FIT (finite integration technique) and transient solver

The accuracy of the extraction depends on mesh density, frequency sampling, port definition quality, and material model accuracy. Typical accuracy is within 10-20% of measurement for well-modeled structures.
