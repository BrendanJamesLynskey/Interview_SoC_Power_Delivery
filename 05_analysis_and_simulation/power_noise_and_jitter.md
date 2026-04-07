# Power Noise and Jitter

This section covers the coupling between PDN noise and clock jitter, including supply-induced jitter mechanisms, PLL supply sensitivity, and design techniques to minimize power noise impact on timing.

---

### Q1. How does PDN noise cause clock jitter?

**Answer:**
PDN noise causes clock jitter through several mechanisms that modulate the delay of clock generation and distribution circuits.

**PLL supply sensitivity:** A phase-locked loop generates the SoC clock. The VCO (voltage-controlled oscillator) within the PLL is sensitive to its supply voltage because the transistor switching speed depends on Vdd. When PDN noise modulates the PLL supply voltage, the VCO frequency varies, producing phase modulation of the output clock. This appears as jitter -- variation in the clock period from cycle to cycle.

The supply sensitivity of a VCO is characterized by its power supply sensitivity coefficient (Kvdd), expressed in Hz/V or %/V:

```
delta_f / f = Kvdd * delta_Vdd / Vdd
```

A typical ring oscillator VCO has Kvdd of 5 to 15%/V, meaning a 10 mV supply noise on a 0.85 V supply causes a 0.06 to 0.18% frequency variation, which translates directly to jitter.

**Clock buffer supply noise:** Clock distribution buffers (clock trees) also have supply-dependent delay. When PDN noise modulates the supply of a clock buffer, the buffer delay changes, which shifts the clock edge arrival time at the downstream flip-flops. If different branches of the clock tree experience different supply noise (due to spatial variation in the PDN impedance), the clock skew between branches also varies, contributing to jitter and skew uncertainty.

**Substrate noise coupling:** In bulk CMOS processes, digital switching noise can couple through the substrate to analog circuits (including the PLL). This substrate noise effectively appears as supply noise at the PLL transistors.

The resulting jitter is typically 1 to 10 ps RMS for a well-designed PDN with 10 to 30 mV of supply noise, but can exceed 20 ps if the PDN is poorly designed or the PLL supply is not properly isolated.

---

### Q2. What is supply-induced jitter and how is it quantified?

**Answer:**
Supply-induced jitter (SIJ) is the component of clock jitter caused specifically by voltage fluctuations on the power supply rail. It is distinguished from other jitter sources (intrinsic device noise, phase noise from the reference, deterministic jitter from cross-coupling).

SIJ is quantified in several ways:

**Peak-to-peak jitter (Jpp):** The maximum range of clock edge variation caused by supply noise. For a supply noise amplitude V_noise_pp and a clock buffer with supply sensitivity S_buf (ps/mV):

```
Jpp = S_buf * V_noise_pp
```

**RMS jitter (Jrms):** The root-mean-square variation, which is more meaningful for random supply noise:

```
Jrms = S_buf * V_noise_rms
```

**Period jitter:** The variation in the clock period from one cycle to the next. If the PLL VCO supply has noise at frequency f_noise, the period jitter is:

```
Jperiod = Kvdd * V_noise / (f_clock^2) * f_noise [for f_noise < PLL bandwidth]
```

This is because the PLL loop suppresses low-frequency noise (below its bandwidth) through feedback, so only noise at frequencies above the PLL bandwidth contributes to output jitter.

**Jitter transfer function:** The PLL acts as a high-pass filter for supply noise: noise at frequencies below the PLL bandwidth is suppressed (the PLL adjusts its VCO to compensate), while noise above the PLL bandwidth passes through to the output. The jitter transfer function H_jitter(f) relates the supply noise spectrum to the output jitter spectrum:

```
H_jitter(f) = Kvdd * f / (f^2 + f_PLL_BW^2)  [simplified]
```

The peak jitter sensitivity occurs near the PLL bandwidth frequency. For a PLL with 5 MHz bandwidth, supply noise around 5 MHz produces the most jitter per millivolt of noise.

---

### Q3. How does the PLL bandwidth affect supply-induced jitter?

**Answer:**
The PLL bandwidth determines the boundary between supply noise frequencies that are suppressed by the PLL feedback loop and those that pass through to the output clock.

**Below PLL bandwidth:** Supply noise at these frequencies causes the VCO frequency to vary, but the PLL feedback loop detects the phase error and adjusts the VCO control voltage to compensate. The output jitter is attenuated. The attenuation increases at lower frequencies (the PLL is more effective at correcting slow supply variations).

**Above PLL bandwidth:** Supply noise at these frequencies changes the VCO frequency too quickly for the PLL loop to track. The noise passes through to the output clock as jitter. The jitter contribution increases with the noise frequency (due to the integration effect of phase from frequency), up to a point where the VCO's inherent filtering rolls off the sensitivity.

**At PLL bandwidth:** This is the transition region where the PLL loop gain is approximately unity. There may be peaking in the jitter transfer function (especially if the PLL loop damping is low), meaning noise right at the bandwidth frequency may be amplified rather than suppressed.

**Design implications:**

A higher PLL bandwidth rejects more low-frequency supply noise but passes more high-frequency noise. A lower bandwidth rejects less low-frequency noise but filters high-frequency noise better.

The optimal bandwidth depends on the supply noise spectrum. If the PDN has most of its noise at low frequencies (below 1 MHz, from VRM ripple and bulk cap resonances), a higher PLL bandwidth (5 to 10 MHz) is beneficial. If the PDN noise is dominated by high-frequency components (above 10 MHz, from switching transients), a lower PLL bandwidth (1 to 3 MHz) is better.

In practice, PLL bandwidth is set based on multiple requirements (lock time, reference spur rejection, jitter performance), and the PDN must be designed to minimize noise in the frequency range where the PLL is most sensitive (around and above the PLL bandwidth).

---

### Q4. What design techniques minimize the impact of PDN noise on PLL performance?

**Answer:**
Minimizing the impact of PDN noise on PLLs requires a combination of PDN design, PLL design, and isolation techniques.

**Dedicated PLL supply with LDO post-regulation:** Power the PLL from a separate LDO that is fed by the main supply. The LDO provides high PSRR (60 to 80 dB at low frequencies, 20 to 40 dB at the PLL bandwidth frequency), significantly attenuating the main supply noise before it reaches the PLL. This is the single most effective technique.

**Separate power domain:** Place the PLL on a separate voltage domain with its own decoupling and independent ground connection. This prevents digital switching noise from coupling directly through the shared supply impedance.

**On-die decoupling near the PLL:** Place additional decoupling capacitors (MOS decaps, MOM decaps) in the immediate vicinity of the PLL to suppress local supply noise at the frequencies the PLL is most sensitive to.

**Guard ring isolation:** Surround the PLL with a deep N-well guard ring (in bulk CMOS) or a grounded P+ ring to block substrate noise coupling from digital blocks.

**PLL design for high PSRR:** Design the VCO with differential topology (both supply and ground noise are common-mode rejected) or with supply regulation within the VCO (an internal regulator that further isolates the oscillator from supply variation). Modern PLLs often include an internal LDO for this purpose.

**Minimize PDN impedance at PLL-sensitive frequencies:** Ensure the PDN impedance profile is below the target specifically in the frequency range around the PLL bandwidth (where the PLL is most sensitive). Even if the PDN meets the general target impedance, a small peak at the PLL bandwidth frequency can cause disproportionate jitter.

**Physical separation:** Place the PLL as far as practical from the noisiest digital blocks (processor cores, I/O drivers) on the die floorplan, increasing the transfer impedance of the PDN between noise source and victim.

---

### Q5. How is PDN-induced jitter analyzed in the SoC design flow?

**Answer:**
PDN-induced jitter analysis requires combining PDN noise simulation with PLL sensitivity modeling to predict the jitter at the clock output.

**Step 1: Obtain the supply noise waveform.** Run dynamic IR drop analysis (RedHawk, Voltus) at the PLL supply pins to obtain the time-varying voltage waveform V_supply(t) under a realistic workload.

**Step 2: Extract the noise spectrum.** Compute the FFT of V_supply(t) to obtain the spectral content of the supply noise: V_noise(f).

**Step 3: Apply the PLL jitter transfer function.** Multiply the noise spectrum by the PLL's supply-to-jitter transfer function H_jitter(f) to obtain the jitter spectrum:

```
J(f) = H_jitter(f) * V_noise(f)
```

The transfer function is obtained from the PLL design team (computed from the PLL model parameters).

**Step 4: Compute total jitter.** Integrate the jitter spectrum to obtain the RMS jitter:

```
J_rms = sqrt(integral of |J(f)|^2 df)
```

**Step 5: Compare to specification.** The computed jitter is compared to the jitter budget for the PLL (which allocates a portion of the total allowed jitter to supply-induced jitter, with the remainder allocated to intrinsic noise, reference noise, etc.).

**Step 6: Iterate if needed.** If the supply-induced jitter exceeds its budget, the PDN must be improved (more decoupling at the sensitive frequencies), the PLL must be redesigned (better PSRR, internal regulation), or the isolation must be enhanced (better LDO, separate domain).

Some advanced power integrity tools (RedHawk, Voltus) include built-in PLL sensitivity analysis that automates this flow, allowing the designer to specify the PLL model parameters and directly compute the supply-induced jitter contribution.

---

### Q6. What is the relationship between supply noise and signal integrity?

**Answer:**
Supply noise affects signal integrity through several mechanisms that can degrade the quality of digital and analog signals.

**Timing margin reduction:** Supply noise directly affects gate delay. When the supply voltage drops (droop), gates slow down, increasing delay. This effectively reduces the setup time margin of the downstream flip-flop. If the droop occurs during a critical timing path, a setup violation can result. The timing impact of supply noise can be modeled as:

```
delta_delay / delay ~ -alpha * delta_Vdd / Vdd
```

where alpha is typically 1 to 2 (delay is inversely proportional to approximately Vdd^alpha for sub-threshold to strong inversion operation).

**Simultaneous switching noise (SSN) on I/O:** When many I/O drivers switch simultaneously, the current through the shared ground path inductance creates a ground bounce voltage. This effectively reduces the output voltage swing and the input noise margin of the receiving device. The ground bounce magnitude is:

```
V_bounce = L_ground * N * di/dt
```

where L_ground is the ground path inductance, N is the number of simultaneously switching drivers, and di/dt is the per-driver current slew rate.

**Analog circuit performance:** Supply noise directly degrades the performance of analog circuits: ADC resolution (supply noise adds to the quantization noise), DAC linearity, and amplifier gain accuracy are all affected.

**SerDes jitter:** High-speed serial transceivers (PCIe, USB, Ethernet) have tight jitter budgets. Supply noise on the SerDes transmitter causes data-dependent jitter, and on the receiver causes sampling time uncertainty. Both reduce the link margin and maximum achievable data rate.

The PDN must be designed holistically to ensure that supply noise meets the requirements of all circuits on the chip, not just the worst-case logic timing path.

---

### Q7. How do you budget jitter across different noise sources?

**Answer:**
Jitter budgeting allocates the total allowed jitter among the various independent contributors, ensuring that the root-sum-square (RSS) total does not exceed the specification.

For a clock distribution system, the total jitter is:

```
J_total = sqrt(J_pll^2 + J_supply^2 + J_clock_tree^2 + J_coupling^2 + J_random^2)
```

where:
- J_pll: intrinsic PLL jitter (VCO phase noise, charge pump noise)
- J_supply: supply-induced jitter (PDN noise)
- J_clock_tree: jitter from clock buffer supply noise and load variation
- J_coupling: jitter from crosstalk coupling to clock nets
- J_random: random thermal and shot noise jitter

A typical jitter budget for a 2 GHz processor clock with a total budget of 15 ps RMS might allocate:

| Source | Budget (ps RMS) |
|--------|----------------|
| PLL intrinsic | 5 |
| Supply-induced (PLL) | 6 |
| Clock tree supply | 4 |
| Clock tree coupling | 3 |
| Random | 2 |
| RSS total | ~9.7 (within 15 ps) |

The PDN designer is responsible for ensuring that the supply noise at the PLL and clock buffer supply pins is low enough to keep J_supply and J_clock_tree within their budgets. This means:

- Supply noise at PLL: V_noise < J_supply_budget / S_pll, where S_pll is the PLL supply sensitivity (ps/mV)
- Supply noise at clock buffers: V_noise < J_clock_tree_budget / S_buf

For J_supply = 6 ps and S_pll = 0.5 ps/mV: V_noise_pll < 12 mV. This is the PDN noise specification at the PLL supply pin.

---

### Q8. What is the power supply sensitivity of common circuit blocks?

**Answer:**
Different circuit blocks have different sensitivities to supply voltage variation, which determines how much PDN noise is tolerable for each.

**Ring oscillator VCO:** Highest sensitivity. Frequency varies 5 to 15% per volt of supply change. Even 5 mV of supply noise can produce measurable jitter. This is why PLLs always need the cleanest possible supply.

**LC oscillator VCO:** Lower sensitivity than ring oscillators because the resonant frequency is set primarily by the LC tank values, not the transistor parameters. Sensitivity is typically 0.5 to 2% per volt. LC-VCO PLLs are preferred for jitter-sensitive applications (SerDes, clock generators).

**Clock buffers:** Delay sensitivity is approximately 1 to 2% per volt. A 10 mV supply noise causes about 10 to 20 ps of delay variation on a 1 ns buffer delay.

**Logic gates:** Similar sensitivity to clock buffers. Supply noise affects path delay and timing margins.

**SRAM:** Highly sensitive to supply noise during read operations. The sensing circuits rely on small differential voltages (50 to 100 mV) between bitlines, and supply noise directly adds to the noise margin. A 30 mV supply droop can cause read failures in 6T SRAM at low Vdd.

**ADC/DAC:** Supply noise directly limits the achievable signal-to-noise ratio. For an N-bit ADC with full-scale range Vfs, the LSB is Vfs/2^N. Supply noise larger than the LSB causes errors. A 12-bit ADC with 1 V range has an LSB of 244 uV, requiring supply noise below approximately 100 uV for full accuracy.

**SerDes transmitter:** Supply noise modulates the output eye, adding both amplitude noise and jitter. Typical sensitivity is 1 to 3 ps/mV.

**I/O drivers:** Supply noise modulates the output voltage levels (VOH, VOL) and the driver slew rate, affecting the signal quality at the receiver.

---

### Q9. How does power gating affect PDN noise and jitter?

**Answer:**
Power gating introduces several PDN noise events that can impact jitter and overall power integrity.

**Rush current during power-up:** When a power-gated block is turned on, the header switches connect the discharged virtual-VDD network to the always-on supply. The rush current to charge all the capacitance (gate caps, wiring caps, decaps) in the block can be very large -- potentially several amperes for a large block. This rush current flows through the always-on PDN, causing a voltage droop on the always-on supply that affects all active circuits, including PLLs and clock trees.

The voltage droop on the always-on supply is:

```
V_droop = I_rush * Z_PDN
```

If the rush current is 5 A and the PDN impedance at the rush current frequency is 2 mOhm, the droop is 10 mV, which translates directly to jitter on any PLL powered by that supply.

**Mitigation of rush current impact:**
- Implement gradual turn-on (daisy-chain header switches with progressive sizing)
- Limit the maximum rush current through control of the header switch turn-on rate
- Ensure adequate decoupling on the always-on supply near the power-gated block
- Time power-up events to avoid coinciding with other high-activity periods

**Noise during power-down:** When a block is powered off, the header switches disconnect the supply. The stored energy in the block's capacitance and inductance can cause voltage ringing on both the virtual-VDD (being powered down) and the always-on supply. Isolation cells at the domain boundary prevent the ringing from propagating to active logic, but the impact on the always-on supply noise must be managed.

**Steady-state impact:** While the block is powered off, its decoupling capacitance is disconnected from the always-on supply, reducing the total available decoupling. This must be accounted for in the PDN design -- the always-on supply must meet its impedance target even without the decoupling contribution of power-gated blocks.

---

### Q10. What measurement techniques are used to characterize supply-induced jitter on silicon?

**Answer:**
Characterizing supply-induced jitter on fabricated silicon requires both on-die and off-die measurement techniques.

**On-die jitter measurement circuits:** Many SoCs include built-in self-test (BIST) circuits that measure clock jitter directly on the die. These typically consist of a time-to-digital converter (TDC) that measures the time difference between successive clock edges and reports the distribution of period jitter. By correlating the measured jitter with the supply noise (measured simultaneously), the supply-induced component can be isolated.

**Supply noise injection:** To characterize the PLL supply sensitivity, a controlled noise signal is injected onto the PLL supply through an on-die test port or an external injection point. The resulting jitter is measured as a function of the injected noise frequency and amplitude. This produces the jitter transfer function H_jitter(f).

**Real-time oscilloscope measurement:** The clock output is probed (either at an on-die test pad or at an I/O pin) and captured on a high-bandwidth real-time oscilloscope (>20 GHz bandwidth for multi-GHz clocks). The oscilloscope's jitter analysis software computes the period jitter, cycle-to-cycle jitter, and RMS jitter from the captured waveform.

**Spectrum analyzer:** The clock spectrum is measured using a spectrum analyzer. Supply-induced jitter appears as sidebands around the clock fundamental at offsets equal to the supply noise frequencies. The sideband amplitude relative to the carrier is related to the jitter amplitude.

**Correlation with supply noise measurement:** Simultaneously probing the supply voltage (with a high-bandwidth differential probe) and the clock output (with a low-capacitance probe) allows direct correlation between supply noise events and jitter events. A cross-correlation or coherence analysis quantifies the causal relationship.

**Silicon validation flow:** The measured jitter and supply noise are compared to the pre-silicon predictions. Discrepancies indicate either modeling errors in the PDN or PLL, or measurement artifacts. This feedback loop improves the accuracy of future designs.
