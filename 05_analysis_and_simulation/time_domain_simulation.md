# Time Domain Simulation

This section covers time-domain simulation methods for PDN analysis, including step load response, droop and overshoot characterization, and transient simulation methodology.

---

### Q1. What is a step load response and what does it reveal about PDN performance?

**Answer:**
A step load response is the time-domain voltage waveform observed at the load when the load current changes abruptly from one level to another (typically from low to high current or vice versa). It is the most intuitive and physically meaningful characterization of PDN transient performance.

When a positive current step occurs (current increases suddenly), the voltage response typically shows: an initial fast droop determined by the ESR of the nearest decoupling capacitors, followed by a slower droop as the capacitors discharge, a minimum voltage point (maximum droop), and then a recovery as the VRM increases its inductor current to match the new load level. The voltage may oscillate around the final value before settling.

Key parameters extracted from the step response: peak droop magnitude (must be within specification), peak overshoot (for step-down transients), settling time (time to reach within a specified error band of the final value), and the presence of ringing (which indicates poorly damped resonances in the PDN).

The step response reveals the interaction between all PDN stages in the time domain. A well-designed PDN shows a smooth, monotonic droop followed by a clean recovery with minimal overshoot. Poorly designed PDNs show excessive droop, ringing at the frequency of anti-resonance peaks, or slow settling due to inadequate VRM bandwidth.

The step response is related to the impedance profile through the inverse Fourier transform: a flat, low impedance profile produces a small, well-behaved step response, while impedance peaks produce ringing at the corresponding frequencies.

---

### Q2. How do you set up a time-domain PDN simulation?

**Answer:**
A time-domain PDN simulation requires a complete circuit model of the PDN, a realistic current stimulus, and a transient circuit simulator.

**PDN model setup:** Assemble the cascaded PDN model including the VRM output impedance (frequency-dependent model or a SPICE subcircuit), PCB plane model and decoupling capacitors, package S-parameter model (imported as a frequency-dependent element), and on-die grid model with decoupling.

**Current stimulus:** Define the load current waveform that represents the worst-case transient event. Common stimuli include: a step current (e.g., 0 to 40 A in 10 ns) representing clock ungating, a pulse current (step up and then step down) representing a burst of activity, and realistic current waveforms extracted from gate-level simulation of the actual SoC operating under a specific workload.

**Simulation settings:** Choose an appropriate time step (typically 0.1 to 1 ns for resolving GHz-range dynamics), simulation duration (long enough to observe the full transient and VRM recovery, typically 10 to 100 us), and convergence tolerance.

**Run the simulation:** The circuit simulator (HSPICE, Spectre) solves the circuit equations at each time step, computing the voltage at every node as a function of time.

**Post-processing:** Extract the minimum voltage (maximum droop), maximum voltage (overshoot), settling time, and RMS noise from the voltage waveform at the die load point. Compare these against specifications.

For vectorless dynamic IR drop analysis in power integrity tools (RedHawk, Voltus), the current stimulus is generated automatically from the design's switching activity profile. The tool applies a statistical model of worst-case simultaneous switching to create a realistic current waveform, then simulates the power grid response in the time domain.

---

### Q3. What causes voltage droop during a load transient and what are its components?

**Answer:**
Voltage droop during a positive load step (increasing current) has three distinct components, each dominated by a different physical mechanism and time scale.

**First droop (resistive, 0 to ~1 ns):** The instantaneous voltage drop caused by the current step flowing through the ESR of the on-die decoupling capacitors and the resistance of the die power grid. This occurs at the speed of electromagnetic wave propagation in the grid and is essentially instantaneous. Magnitude: V1 = I_step * (R_grid + ESR_die).

**Second droop (capacitive, 1 ns to ~1 us):** After the initial resistive drop, the on-die and package decoupling capacitors begin discharging to supply the transient current. The voltage drops at a rate proportional to I_step / C_local. This continues until the PCB capacitors and VRM can respond. The second droop is typically the largest component and is limited by the total local capacitance (on-die + package). Magnitude: V2 approximately equals I_step * L_spread / C_die (where L_spread is the package spreading inductance that limits the current delivery rate from package to die).

**Third droop (VRM response, 1 us to ~50 us):** After the on-die and package capacitors are partially depleted, the PCB bulk capacitors supply current. Eventually, the VRM control loop responds and ramps the inductor current to match the new load. Until the inductor current fully catches up, the output voltage remains below nominal. The third droop depends on the VRM bandwidth, inductor value, and output capacitance. Magnitude: V3 depends on the VRM response time and capacitor charge depletion.

The total droop is the sum of these three components, and the worst-case (minimum) voltage typically occurs at the transition between the second and third droop phases. In a well-designed PDN with adaptive voltage positioning, the total droop is managed to stay within the specified voltage window.

---

### Q4. What is settling time and what factors affect it?

**Answer:**
Settling time is the duration from the onset of a load transient until the output voltage permanently enters and remains within a specified error band around the final value (typically 1% or 2% of Vdd). It characterizes how quickly the PDN returns to a stable state after a disturbance.

Factors affecting settling time:

**VRM control loop bandwidth:** Higher bandwidth enables faster response, reducing settling time. A 200 kHz bandwidth VRM settles approximately 5x faster than a 40 kHz bandwidth VRM.

**Loop phase margin:** Adequate phase margin (>50 degrees) ensures a well-damped response with minimal overshoot and ringing after the droop. Low phase margin causes the output to ring for many cycles before settling, extending the settling time even if the initial droop is acceptable.

**Output capacitance and ESR:** Larger output capacitance reduces droop but may slow recovery because the VRM must charge a larger capacitor back to the nominal voltage. The ESR provides damping that reduces ringing.

**Anti-resonance in the PDN:** Underdamped anti-resonance peaks cause ringing at specific frequencies. If the damping is low, this ringing takes many cycles to decay, extending the settling time. The ringing frequency corresponds to the anti-resonance frequency in the impedance profile.

**Load-line (AVP) implementation:** With load-line regulation, the final settled voltage is different from the initial voltage (lower at higher load), so the settling criteria must be defined relative to the new target voltage. A well-implemented AVP reduces the settling time by reducing the total voltage excursion.

Typical settling times for modern SoC PDN: 5 to 50 microseconds for VRM-level settling (third droop recovery), and 10 to 100 nanoseconds for die-level settling (first and second droop recovery from on-die and package decaps).

---

### Q5. How do you model a realistic current transient for PDN simulation?

**Answer:**
Realistic current transients are essential for accurate PDN simulation because idealized step functions overestimate the high-frequency content and may produce overly pessimistic results.

**Gate-level simulation extraction:** The most accurate approach runs a gate-level simulation of the SoC with a realistic workload (benchmark, stress test) and extracts the time-varying current consumption of each power domain. The current waveform captures the actual switching patterns, including clock gating events, pipeline stalls, cache misses, and instruction-dependent activity. This waveform is used directly as the current stimulus in PDN simulation.

**Statistical activity estimation:** When gate-level simulation is impractical (too slow for large designs or early design stages), vectorless methods estimate the current waveform statistically. The power integrity tool uses the toggle rates, timing windows, and power per instance from static timing analysis and power analysis to generate a probabilistic current waveform that represents the worst-case simultaneous switching.

**Simplified waveform models:** For early-stage analysis and hand calculations, simplified current waveforms are used:

Trapezoidal step: A current step from I_low to I_high with a finite risetime t_r. The risetime is estimated from the clock period (typically 1 to 5 clock cycles for a clock ungating event).

```
I(t) = I_low + (I_high - I_low) * t / t_r,  for 0 < t < t_r
I(t) = I_high,  for t >= t_r
```

Pulse train: A periodic rectangular current pulse at the clock frequency, with amplitude equal to the dynamic current and duration equal to half the clock period. This represents the repetitive switching current.

**Current source model:** In SPICE simulation, the current waveform is applied as an ideal current source connected between VDD and VSS at the die port of the PDN model. For distributed analysis, multiple current sources at different die locations represent the spatial distribution of current demand.

---

### Q6. What is the relationship between time-domain and frequency-domain PDN analysis?

**Answer:**
Time-domain and frequency-domain analyses are mathematically equivalent descriptions of the same physical system, connected by the Fourier transform. Each has advantages depending on what information is needed.

**Frequency-domain analysis** (impedance profile) is best for understanding the broadband behavior of the PDN, identifying resonant and anti-resonant frequencies, designing the decoupling network, and comparing the PDN to the target impedance specification. It directly shows where in the frequency spectrum the PDN is weak and what modifications will improve it.

**Time-domain analysis** (transient simulation) is best for computing the actual voltage waveform under realistic operating conditions, determining the peak droop and overshoot, verifying the settling time, and assessing the impact on circuit timing.

The connection between them is:

```
v(t) = IFFT[Z(f) * I(f)]
```

where v(t) is the time-domain voltage noise, Z(f) is the impedance profile, I(f) is the Fourier transform of the current waveform, and IFFT is the inverse fast Fourier transform.

In practice, a PDN that meets the target impedance across all frequencies will produce acceptable time-domain noise for any current waveform with spectral content within the analyzed range. Conversely, a time-domain simulation that passes for one specific current waveform does not guarantee that the PDN will pass for all possible waveforms -- a different workload might excite a frequency where the impedance is too high.

Therefore, best practice is to use both: frequency-domain analysis for design and optimization of the decoupling network, and time-domain analysis for final verification with realistic workload-specific current waveforms.

---

### Q7. How do power integrity tools perform dynamic IR drop analysis?

**Answer:**
Dynamic IR drop analysis in tools like ANSYS RedHawk or Cadence Voltus combines the on-die power grid model with time-varying current waveforms and the package/board model to compute the voltage at every grid node as a function of time.

**Grid extraction:** The tool reads the physical design database (DEF, LEF, technology files) and constructs an RLC network model of the on-die power grid. Each metal segment is represented by a resistance (from sheet resistance and geometry) and an inductance. Via connections are resistors. On-die decoupling capacitors (MOS decap cells, intrinsic gate cap) are identified and included as capacitance elements.

**Instance power model:** Each standard cell and macro instance is associated with a dynamic current waveform. This waveform is either extracted from a gate-level simulation (VCD file) or estimated statistically from the cell's internal power, switching power, and leakage power, combined with the signal toggle rates and timing windows.

**Package and board model:** A simplified lumped model or imported S-parameter model of the package and PCB is connected at the bump pads to capture the off-die impedance.

**Transient simulation:** The tool performs a time-domain simulation of the entire RLC network with the time-varying current sources, typically over a window of 5 to 20 clock cycles centered on the worst-case activity period. The simulation uses specialized sparse matrix solvers optimized for the regular grid structure.

**Output:** The result is a time-varying voltage map of the entire die. The tool reports the worst-case dynamic voltage drop at each instance, identifies hotspots, and generates visualizations (voltage maps, waveform plots). Instances where the voltage drops below the specified minimum are flagged as violations.

This analysis is typically part of the power integrity signoff flow and is performed late in the design cycle when the physical design is near-final and detailed activity data is available.

---

### Q8. What is the difference between vectored and vectorless dynamic analysis?

**Answer:**
Vectored and vectorless analyses differ in how the switching activity of the design is specified.

**Vectored analysis** uses switching activity data extracted from a gate-level simulation of the design running a specific workload. The simulation produces a VCD (Value Change Dump) or FSDB (Fast Signal Database) file that records the toggling of every signal in the design over a time window. This data is imported into the power integrity tool, which computes the current drawn by each instance at each time step based on the actual switching pattern.

Advantages: most accurate representation of real operating conditions; captures temporal correlations between blocks (e.g., cache miss followed by memory controller activity); produces realistic worst-case current waveforms.

Disadvantages: requires a gate-level simulation (time-consuming for large designs); the result is specific to the chosen workload and may miss worst cases for other workloads; the VCD/FSDB file can be very large.

**Vectorless analysis** estimates the switching activity statistically without running a gate-level simulation. The tool uses the toggle rates (switching frequency) of each signal, the timing windows (when each signal can toggle relative to the clock), and the instantaneous power of each cell to estimate a probabilistic current waveform. The tool identifies the worst-case combination of simultaneously switching instances using statistical methods (e.g., identifying the time window where the maximum number of high-current instances can switch simultaneously).

Advantages: fast, does not require gate-level simulation; can identify worst cases that may not occur with any specific workload; suitable for early design stages when RTL simulation is not yet available.

Disadvantages: may be pessimistic (overestimate the worst case) because it assumes statistically worst-case simultaneous switching; does not capture workload-specific temporal correlations; accuracy depends on the quality of the toggle rate estimates.

Best practice: use vectorless analysis for early design exploration and iteration, then verify with vectored analysis using representative workloads before signoff.

---

### Q9. How do you interpret a dynamic IR drop voltage map?

**Answer:**
A dynamic IR drop voltage map is a 2D visualization of the worst-case instantaneous voltage drop at every point on the die during a transient simulation. It is typically displayed as a color-coded overlay on the die floorplan.

**Color scale:** Blue/green indicates low voltage drop (good), yellow indicates moderate drop, and red indicates high drop (hotspot). The scale is usually set so that the maximum color corresponds to the specified IR drop budget (e.g., 10% of Vdd).

**Spatial patterns to look for:**

Hotspots near high-activity blocks: CPU cores, GPU shader arrays, and wide datapaths that switch heavily during the simulation window appear as red regions. This is expected and indicates that the grid must be strengthened or more decoupling added in these areas.

Hotspots far from power bumps: Points equidistant from multiple bumps (centers of bump grid cells) may show higher drop because they have the longest grid path to the nearest current injection point.

Hotspots near macros: Hard macros can block the power grid, creating current bottlenecks at their boundaries. If the grid must detour around a large memory macro, the area on the far side of the macro may show elevated drop.

Uniform, low drop: A well-designed grid shows relatively uniform, low voltage drop across the entire die, with minor variations near the highest-activity areas.

**Temporal dimension:** The voltage map represents the worst-case instant in time. Different regions may reach their worst case at different times (e.g., one core is at peak activity while another is idle). Some tools provide time-resolved maps that show the voltage evolution over the simulation window.

**Action items from the map:** Hotspots exceeding the budget require remediation: widening local grid stripes, adding via stacks, inserting decap cells near the hotspot, adding power bumps (if feasible), or reducing the local activity (spreading the load through physical design optimization).

---

### Q10. What are the computational challenges of large-scale PDN transient simulation?

**Answer:**
Transient simulation of a modern SoC PDN is computationally demanding due to the enormous size of the problem and the wide range of time scales involved.

**Problem size:** A modern SoC power grid has millions of nodes (intersections of metal stripes) and millions of RLC elements. The resulting circuit matrix has millions of rows and columns. Each time step requires solving this large sparse linear system.

**Time scale range:** The simulation must resolve nanosecond-scale dynamics (on-die switching events) while also capturing microsecond-scale VRM response. This requires small time steps (0.1 to 1 ns) over a long simulation window (10 to 100 us), resulting in 10,000 to 1,000,000 time steps.

**Memory requirements:** Storing the circuit matrix, the solution vector, and the time-history data for all nodes requires tens to hundreds of gigabytes of RAM.

**Runtime:** A full dynamic IR drop simulation of a large SoC can take 12 to 48 hours on a powerful server with 100+ GB of RAM. Multiple runs may be needed for different corners and workloads.

**Mitigation techniques:**

Model order reduction (MOR): The full RLC grid is reduced to a much smaller equivalent circuit that preserves the input-output behavior at the ports of interest. This can reduce the matrix size by 100x or more.

Hierarchical simulation: The design is partitioned into blocks, each simulated separately with approximate boundary conditions, then combined.

Multi-rate simulation: Different parts of the circuit are simulated at different time steps -- fine time steps for the die grid (where fast transients occur) and coarser time steps for the VRM and PCB (where dynamics are slower).

GPU acceleration: Some modern power integrity tools offload matrix operations to GPUs, providing 5 to 20x speedup over CPU-only computation.

Distributed computing: The simulation is split across multiple servers, each handling a portion of the die, with inter-server communication for boundary conditions.

Despite these optimizations, dynamic IR drop analysis remains one of the most computationally intensive steps in the SoC signoff flow.
