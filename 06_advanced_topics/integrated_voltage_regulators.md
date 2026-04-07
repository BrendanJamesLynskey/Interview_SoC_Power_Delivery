# Integrated Voltage Regulators

This section covers integrated voltage regulator (IVR) architectures, including on-die LDOs, switched-capacitor converters, and inductor-based on-die converters.

---

### Q1. What is an integrated voltage regulator (IVR) and why is it beneficial?

**Answer:**
An integrated voltage regulator is a voltage converter implemented on the same die as the SoC (or within the same package) rather than as a discrete component on the PCB. IVRs bring the regulation point physically closer to the load transistors, dramatically reducing the parasitic impedance of the power delivery path.

Benefits of IVRs include: elimination of the package and PCB parasitic impedance between regulator and load (reducing the effective PDN impedance by 10 to 100x at high frequencies), enabling per-core or per-block voltage regulation for fine-grained DVFS (each core can have its own optimal voltage without requiring a separate off-chip regulator), much faster voltage transitions (microseconds instead of tens of microseconds) because the capacitance to charge/discharge is much smaller, reduction in the number of off-chip components (fewer discrete regulators, inductors, and capacitors), and reduced total board area.

The main challenges are: the IVR consumes silicon area on the die, the efficiency may be lower than discrete regulators (especially for inductor-based designs where on-chip inductors are small and lossy), thermal management of the IVR power dissipation on the die, and design complexity (mixed analog-digital design on a digital process).

IVRs have been implemented in production SoCs, most notably in Intel Haswell (2013) and subsequent processor generations, which use on-die fully integrated voltage regulators to enable per-core DVFS.

---

### Q2. How does an on-die LDO work and what are its tradeoffs?

**Answer:**
An on-die LDO is a linear regulator implemented entirely in the SoC's standard CMOS process. It consists of a large PMOS pass transistor (or an array of pass transistors), an error amplifier, and a voltage reference. The pass transistor conducts current from the input supply (a shared high-voltage rail) to the output supply (a per-block or per-core regulated rail).

The output voltage is regulated by the feedback loop: the error amplifier compares a divided version of the output to a reference and drives the pass transistor gate to maintain the target output voltage.

**Advantages:**
- Very simple design, entirely in digital CMOS process
- No inductors or external capacitors needed
- Excellent PSRR and low output noise
- Very fast transient response (>10 MHz bandwidth possible)
- Compact area (the pass transistor is the main area consumer)
- Enables per-core DVFS with fast transitions (<1 us)

**Disadvantages:**
- Efficiency limited to Vout/Vin. For Vin = 1.0 V and Vout = 0.85 V: eta = 85%. For larger dropout (Vin = 1.2 V, Vout = 0.6 V): eta = 50%.
- Power dissipation: P_loss = (Vin - Vout) * Iout. At 50% efficiency and 10 A: P_loss = 6 W, dissipated on-die.
- Large pass transistor area for high-current designs. A 10 A LDO with 50 mV dropout needs Rdson < 5 mOhm, requiring a very wide PMOS (tens of millimeters of total width).
- The input supply must be delivered to the die at a voltage higher than the maximum output, adding its own delivery challenge.

On-die LDOs are best suited for moderate-current applications (1 to 10 A per domain) where the input-output differential is small (100 to 300 mV) and fast response is critical.

---

### Q3. What is a switched-capacitor voltage converter and how is it implemented on-die?

**Answer:**
A switched-capacitor (SC) converter uses a network of switches and capacitors to transfer charge from input to output, achieving voltage conversion without inductors. The switches alternately connect the capacitors in different configurations to achieve the desired conversion ratio.

**Basic operation (2:1 step-down):** Two capacitors are alternately connected in series (charged from Vin) and in parallel (discharged to Vout). In the series configuration, each capacitor charges to Vin/2. In the parallel configuration, they discharge in parallel to the output at Vin/2. This achieves an ideal 2:1 conversion ratio.

**On-die implementation:** The switches are CMOS transistors (PMOS for high-side, NMOS for low-side). The capacitors are implemented using MOS capacitors (similar to decap cells) or MOM capacitors in the metal layers. The controller generates the switching signals at a frequency of 10 to 500 MHz.

**Advantages for on-die integration:**
- No inductors (which are difficult to integrate on-die at useful values)
- Capacitors are readily available using MOS or MOM structures
- Scalable -- adding more capacitor stages improves current capacity
- Compatible with standard CMOS process

**Disadvantages:**
- Efficiency depends on the ratio of actual conversion to ideal conversion ratio. A 2:1 converter providing Vout = Vin/2 (exactly) is efficient, but providing Vout = 0.4*Vin (less than 1/2) wastes energy in the resistive losses.
- The output impedance of an SC converter is inversely proportional to f_sw * C_fly, where C_fly is the flying capacitor value. Large capacitors and high switching frequency are needed for low output impedance.
- Voltage conversion ratios are limited to rational fractions (1/2, 2/3, 1/3, 3/4, etc.). Arbitrary voltage conversion requires multiple stages or a combination with an LDO.

**Efficiency:** For an ideal SC converter at its target conversion ratio: eta_ideal = Vout_actual / Vout_ideal_ratio. In practice, switch resistance, capacitor ESR, and control overhead reduce efficiency to 80 to 90% for well-designed converters.

---

### Q4. How are inductors integrated for on-package or on-die voltage regulators?

**Answer:**
Inductor-based switching converters require inductors, which are the most challenging component to integrate due to the need for magnetic materials and large physical volume for useful inductance values.

**On-package inductors:** Inductors can be embedded in the package substrate or mounted on the package surface. Thin-film magnetic materials (NiFe, CoZrTa) are deposited on the substrate and patterned to form solenoid or toroidal inductors. Typical values: 1 to 50 nH with Q of 5 to 20 at the switching frequency. The inductors are small (0.5 to 2 mm per side) and can be integrated into the package without significantly increasing its size.

**On-die air-core inductors:** Spiral inductors fabricated in the top metal layers of the die, using the BEOL metallization. These achieve only 0.5 to 5 nH due to the thin metal layers and lack of magnetic core material. The quality factor is limited to 5 to 15 due to resistive losses in the thin metal. However, at very high switching frequencies (100 to 500 MHz), these small inductors can support useful voltage conversion.

**On-die magnetic-core inductors:** Research and advanced development have explored depositing thin magnetic films (NiFe permalloy, CoFeB) on the die surface to create planar magnetic-core inductors. These achieve higher inductance density than air-core inductors (5 to 50 nH in a small area) but require additional process steps and careful control of the magnetic material properties.

**Tradeoffs:**

| Inductor Type | L (nH) | Area | Q | Process Impact | Maturity |
|--------------|--------|------|---|---------------|----------|
| On-package embedded | 1-50 | 1-4 mm^2 | 10-20 | Package process | Production |
| On-die air-core | 0.5-5 | 0.1-1 mm^2 | 5-15 | None (BEOL only) | Production |
| On-die magnetic-core | 5-50 | 0.1-0.5 mm^2 | 10-25 | Magnetic film deposition | R&D/early production |

The trend is toward higher switching frequencies (100 MHz to 1 GHz) to reduce the required inductance value, making on-die and on-package inductors more practical.

---

### Q5. What are the efficiency tradeoffs of different IVR topologies?

**Answer:**
The three main IVR topologies (LDO, switched-capacitor, inductor-based buck) have fundamentally different efficiency characteristics.

**On-die LDO:**
- Efficiency: eta = Vout / Vin (inherent)
- Best case: small dropout (Vin = 0.95 V, Vout = 0.85 V, eta = 89%)
- Worst case: large step-down (Vin = 1.2 V, Vout = 0.5 V, eta = 42%)
- Independent of load current
- Efficiency does not vary with switching frequency (no switching)

**Switched-capacitor:**
- Ideal efficiency determined by conversion ratio matching: eta = Vout / (Vin * M/N)
- For 2:1 conversion at Vin = 1.0 V, Vout = 0.5 V: eta_ideal = 100%
- For 2:1 conversion at Vin = 1.0 V, Vout = 0.4 V: eta_ideal = 80% (remaining 20% lost in switches)
- Practical efficiency: 75 to 92% including switch and capacitor losses
- Efficiency peaks at discrete conversion ratios and drops between them

**Inductor-based buck:**
- Efficiency: 80 to 95% in principle, but on-die implementations suffer from low inductor Q and high switching losses
- Practical on-die/on-package efficiency: 75 to 90%
- Continuous efficiency over the full voltage range (no discrete conversion ratio constraint)
- Efficiency degrades at very high switching frequencies (>500 MHz) due to switching losses

**Comparison for a typical application** (Vin = 1.0 V, Vout range 0.5 to 0.85 V):

| Topology | eta at 0.85V | eta at 0.65V | eta at 0.5V | Area | Complexity |
|----------|-------------|-------------|-------------|------|------------|
| LDO | 85% | 65% | 50% | Small | Low |
| SC (2:1) | ~90% | ~80% | ~92% | Medium | Medium |
| Buck | ~88% | ~85% | ~82% | Large | High |

The optimal choice depends on the application: LDOs for small dropout, high-response applications; SC for fixed-ratio conversions; buck for wide-range, high-efficiency needs.

---

### Q6. How does an IVR affect the PDN impedance profile?

**Answer:**
An IVR fundamentally changes the PDN impedance profile by bringing the regulation point from the PCB onto the die (or package), eliminating the intermediate PDN stages.

**Without IVR (conventional):** The PDN impedance includes the VRM output impedance (effective up to ~200 kHz), PCB plane and decoupling (200 kHz to 50 MHz), package and package decaps (50 MHz to 300 MHz), and on-die decoupling (above 300 MHz). Each handoff creates potential anti-resonance peaks.

**With on-die IVR:** The IVR output is on the die, directly adjacent to the load. The PDN impedance includes only the IVR output impedance (effective up to its bandwidth, potentially several MHz for an LDO) and the on-die decoupling (above the IVR bandwidth). The PCB and package PDN only need to deliver the IVR input supply, which has more relaxed impedance requirements because the IVR input voltage is higher and the current ripple is smoother.

**Impedance improvement:** The IVR eliminates the package spreading inductance from the regulated supply path. For a conventional PDN, the package spreading inductance (20 to 100 pH) sets a minimum impedance of 60 to 300 uOhm at 500 MHz. With an on-die IVR, the impedance at 500 MHz is determined by the on-die decoupling capacitance and the IVR output impedance, potentially achieving 10 to 100 uOhm.

**Input supply considerations:** The IVR input supply still needs a conventional PDN (PCB + package delivery), but the requirements are relaxed because: the input voltage is higher (wider noise margin), the input current is lower (for a step-down converter), and the input current waveform is smoother (the IVR input draws relatively constant current with small ripple).

---

### Q7. What are the thermal implications of on-die IVRs?

**Answer:**
On-die IVRs dissipate power directly on the silicon die, adding to the already significant thermal challenge of modern SoCs.

**Power dissipation:** An IVR supplying 10 A at 85% efficiency dissipates: P_loss = P_out * (1/eta - 1) = 8.5 W * (1/0.85 - 1) = 1.5 W. For an LDO at 70% efficiency: P_loss = 8.5 * (1/0.7 - 1) = 3.6 W. This heat is generated in a small area (the pass transistor or switch array), creating a localized hotspot.

**Thermal impact on surrounding circuits:** The IVR hotspot raises the temperature of nearby logic blocks, increasing their leakage power and potentially degrading their timing margins. The thermal coupling depends on the distance and the die's thermal conductivity.

**Co-design with thermal management:** The IVR placement must consider both electrical (close to the load for minimum impedance) and thermal (spread the heat, avoid stacking hotspots) requirements. Some designs place the IVR at the periphery of the core it serves, distributing the heat along the edge rather than concentrating it in the center.

**Efficiency at elevated temperature:** IVR efficiency may degrade at high temperature due to increased MOSFET Rdson (higher resistance = more conduction loss) and increased leakage (more power wasted in off-state switches). This creates a positive thermal feedback loop that must be managed.

**Thermal throttling interaction:** The SoC's thermal management system may need to throttle the IVR output (reduce voltage/frequency) if the IVR temperature exceeds a safe limit, even if the overall chip thermal budget has headroom. This requires thermal sensors near the IVR.

---

### Q8. How does Intel's FIVR (Fully Integrated Voltage Regulator) work?

**Answer:**
Intel introduced the Fully Integrated Voltage Regulator (FIVR) in its Haswell processor generation (2013) and has used variations in subsequent generations. FIVR integrates the voltage regulator onto the processor die to enable per-core DVFS.

**Architecture:** FIVR is a multi-phase inductor-based buck converter. The power switches (PMOS high-side, NMOS low-side) are fabricated on the processor die using the standard logic process. The output inductors are embedded in the package substrate using thin-film magnetic technology. The controller, gate drivers, and current sensing are also on-die.

**Key features:**
- Switching frequency: ~140 MHz (much higher than discrete VRMs at 0.5-2 MHz)
- Multiple phases per core for current sharing
- Per-core output voltage with independent DVFS control
- Input voltage: ~1.8 V (delivered from a discrete first-stage converter on the motherboard)
- Output voltage: 0.5 to 1.2 V (core DVFS range)
- Efficiency: ~85 to 90% at typical operating points

**Package inductors:** The inductors are fabricated as thin-film solenoids embedded in the package substrate. Each inductor is approximately 1 to 2 nH, suitable for the 140 MHz switching frequency. The magnetic core material is a thin-film NiFe alloy deposited on the package substrate.

**Benefits realized:** FIVR enabled Intel processors to implement per-core Turbo Boost with voltage regulation granularity that was not possible with motherboard-mounted VRMs. Each core can independently boost its voltage and frequency when other cores are idle, improving single-threaded performance without exceeding the chip's power budget.

**Challenges encountered:** The initial FIVR implementation (Haswell) had lower efficiency than the discrete VRM it replaced, resulting in slightly higher total system power at high load. Subsequent generations improved efficiency through process and design optimizations. The package inductors added cost and complexity to the package manufacturing process.

---

### Q9. What is a hybrid IVR architecture?

**Answer:**
A hybrid IVR architecture combines multiple converter topologies to achieve better efficiency and performance than any single topology can provide alone.

**SC + LDO hybrid:** A switched-capacitor converter performs the coarse voltage step-down (e.g., 2:1 from 1.0 V to 0.5 V) with high efficiency, and an LDO provides fine regulation and DVFS adjustment around the SC output. The LDO only needs to handle a small dropout (e.g., 0.5 V +/- 100 mV), so its efficiency is high (>80%). The combined efficiency is the product: eta_total = eta_SC * eta_LDO = 0.90 * 0.85 = 76.5%. While lower than a standalone buck converter, this architecture requires no inductors, making it fully integrable on-die.

**SC + buck hybrid:** A switched-capacitor first stage provides a high-efficiency coarse conversion (e.g., 3:1 from 1.2 V to 0.4 V), and a low-frequency buck converter with a small on-die or on-package inductor provides fine regulation. The buck operates with a small input-output differential, allowing a small inductor and fast transient response.

**Multi-ratio SC:** An SC converter with multiple selectable conversion ratios (1/2, 2/3, 3/4, etc.) can adapt its ratio to the DVFS operating point. By always operating near an ideal ratio, the efficiency remains high across the DVFS range. A small LDO at the output fine-tunes the voltage between the discrete SC ratios.

**Buck + LDO cascade:** A fast on-die LDO is placed after a slower off-chip or on-package buck converter. The buck provides high-efficiency coarse regulation, and the LDO provides fast transient response and noise rejection. This is essentially the conventional architecture with an on-die LDO added, and is the simplest form of IVR.

Hybrid architectures are an active area of research and product development, with each major SoC vendor exploring different combinations optimized for their specific voltage ranges, current levels, and process technologies.

---

### Q10. What are the key design challenges for IVRs at advanced process nodes?

**Answer:**
Implementing IVRs at 7 nm, 5 nm, and 3 nm process nodes introduces several unique challenges.

**Thin gate oxide limitations:** The maximum voltage that can be applied to a standard transistor decreases with each node (from ~1.0 V at 16 nm to ~0.75 V at 3 nm). This limits the IVR input voltage if only standard devices are used. Thick-oxide I/O devices can handle higher voltages but are larger and slower, reducing efficiency.

**Reduced analog performance:** Advanced digital processes optimize for digital switching speed, not analog accuracy. Transistor gain (gm/Id), output impedance, and matching are degraded compared to older nodes. This makes it harder to design accurate voltage references, high-gain error amplifiers, and precision current sensors on-die.

**Increased leakage:** Higher transistor density and thinner oxides increase leakage current, which represents wasted power in the IVR switches, capacitors, and control circuits. At 3 nm, the leakage of a large pass transistor array can be several hundred milliwatts.

**Interconnect resistance:** As metal layers become thinner and narrower, the on-die routing of high-current IVR signals (input supply, output supply, switch node) becomes more resistive. This increases conduction losses and IR drop within the IVR itself.

**Passive component quality:** On-die inductors (air-core spirals in BEOL metal) have lower Q at advanced nodes due to thinner metals and higher resistance. MOS capacitors for SC converters have higher leakage (thinner oxide) and may have reliability concerns (TDDB) under the continuous DC stress of SC operation.

**Electromagnetic interference:** High-frequency switching (100 MHz to 1 GHz) of large currents on-die creates electromagnetic radiation that can couple to nearby sensitive circuits (PLLs, ADCs, SerDes). Careful shielding and layout isolation are required.

**Design flow integration:** IVR design requires mixed-signal design expertise (analog circuits on a digital process) and tight integration with the PDN design flow. The IVR affects and is affected by the power grid, bump allocation, package design, and thermal management.

Despite these challenges, the benefits of IVRs (per-block DVFS, reduced PDN impedance, faster voltage transitions) are compelling enough that major SoC vendors continue to invest in IVR development for each new process node.
