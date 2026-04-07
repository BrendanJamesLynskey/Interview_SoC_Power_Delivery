# Bulk and Ceramic Decoupling

This section covers the selection and application of bulk and ceramic decoupling capacitors at the PCB level, including tantalum, polymer, electrolytic, and MLCC technologies.

---

### Q1. What types of bulk capacitors are used for PCB-level power delivery?

**Answer:**
Bulk capacitors provide large capacitance values (10 uF to 1000 uF) needed for low-frequency decoupling and energy storage during load transients. The main types are:

**Aluminum electrolytic capacitors:** Very high capacitance (100 uF to 10000 uF) in a relatively large package. High ESR (50 to 500 mOhm) and moderate ESL (5 to 20 nH). They are inexpensive and useful for input filtering on the VRM input rail but are rarely used close to the SoC due to their high ESR and large size.

**Tantalum capacitors:** Solid tantalum capacitors offer 10 to 100 uF in compact surface-mount packages. They have moderate ESR (20 to 200 mOhm) and are reliable at their rated voltage. MnO2-type tantalum caps have a risk of ignition under fault conditions (voltage reversal or surge), so polymer-cathode tantalum caps are preferred for modern designs.

**Polymer capacitors (OS-CON, POSCAP, SP-Cap):** Conductive polymer capacitors use a polymer electrolyte instead of MnO2 or liquid electrolyte. They offer 10 to 500 uF with very low ESR (5 to 30 mOhm) and good high-frequency performance. They are the preferred bulk capacitor for SoC output decoupling because of their low ESR, high ripple current rating, and compact surface-mount form factor.

**Ceramic (MLCC):** High-value MLCCs (10 to 100 uF in 0805 or 1210 packages) blur the line between bulk and ceramic decoupling. They have the lowest ESR (1 to 5 mOhm) and ESL (0.5 to 2 nH with standard mounting) of any discrete capacitor technology. However, they suffer from significant DC bias derating and mechanical sensitivity (cracking under board flex).

For SoC power delivery, the output capacitor bank typically uses a combination of polymer caps (for energy storage and low-frequency decoupling) and high-value MLCCs (for mid-frequency decoupling).

---

### Q2. What is the impedance-versus-frequency characteristic of different capacitor types?

**Answer:**
Each capacitor type has a characteristic impedance profile determined by its capacitance, ESR, and ESL. The profiles differ significantly:

**Aluminum electrolytic (1000 uF, ESR=100 mOhm, ESL=15 nH):**
- Capacitive below ~40 kHz
- Minimum impedance (ESR=100 mOhm) at ~40 kHz
- Inductive above ~40 kHz
- At 1 MHz: Z ~ 94 mOhm (inductive)

**Polymer (100 uF, ESR=10 mOhm, ESL=2 nH):**
- Capacitive below ~350 kHz
- Minimum impedance (ESR=10 mOhm) at ~350 kHz
- Inductive above ~350 kHz
- At 10 MHz: Z ~ 126 mOhm (inductive)

**MLCC (10 uF, ESR=3 mOhm, ESL=1 nH):**
- Capacitive below ~1.6 MHz
- Minimum impedance (ESR=3 mOhm) at ~1.6 MHz
- Inductive above ~1.6 MHz
- At 100 MHz: Z ~ 628 mOhm (inductive)

**Small MLCC (100 nF, ESR=2 mOhm, ESL=0.5 nH):**
- Capacitive below ~22 MHz
- Minimum impedance (ESR=2 mOhm) at ~22 MHz
- Inductive above ~22 MHz
- At 100 MHz: Z ~ 314 mOhm (inductive)

The key insight is that each capacitor type is only effective in a narrow frequency band around its series resonant frequency. Below resonance, it provides capacitive decoupling. Above resonance, it is inductive and actually harmful (adds inductance to the PDN). A complete decoupling strategy uses multiple types to cover the full frequency range.

---

### Q3. What is DC bias derating and how does it affect MLCC selection?

**Answer:**
DC bias derating is the reduction in effective capacitance of an MLCC when a DC voltage is applied across it. This is a property of Class II dielectric materials (X5R, X7R, Y5V) that use ferroelectric ceramics (barium titanate) with high permittivity.

The dielectric constant of these materials decreases as the electric field strength increases. When a DC voltage is applied, the electric field across the dielectric reduces its permittivity, which reduces the capacitance. The magnitude of the derating depends on the DC voltage relative to the rated voltage and the specific dielectric formulation.

For example, a 10 uF X5R MLCC rated at 6.3 V might exhibit:
- At 0 V DC: 10 uF (nominal)
- At 1 V DC: 8 uF (20% reduction)
- At 3 V DC: 5 uF (50% reduction)
- At 6 V DC: 3 uF (70% reduction)

For SoC core voltage rails (0.5 to 1.0 V), the DC bias is relatively low compared to the rated voltage (typically 4 V to 10 V for power delivery caps), so the derating may be modest (10 to 30%). However, for higher-voltage rails (3.3 V, 5 V), the derating can be severe.

**Impact on PDN design:** The effective capacitance at the operating DC voltage must be used in PDN simulation, not the nominal capacitance. Using the nominal value leads to optimistic impedance predictions and potential impedance target violations at the actual operating point.

**Manufacturer data:** MLCC manufacturers provide DC bias curves in their datasheets or online tools (e.g., Murata SimSurfing, TDK SEAT). Designers should always consult these curves when selecting capacitors for power delivery.

**Mitigation:** Choose MLCCs with voltage ratings significantly higher than the operating voltage to minimize derating. Using two 4.7 uF caps rated at 10 V instead of one 10 uF cap rated at 6.3 V may provide more effective capacitance at the operating voltage.

---

### Q4. What is the mounting inductance of a PCB decoupling capacitor and how is it minimized?

**Answer:**
Mounting inductance is the parasitic inductance added by the PCB traces, pads, and vias that connect a discrete capacitor to the power and ground planes. It is often the dominant contributor to the total ESL of a PCB-mounted decoupling capacitor.

A typical PCB mounting adds 0.3 to 2 nH of inductance, depending on the via configuration. The main contributors are:

**Via inductance:** Each via connecting a capacitor pad to a buried power or ground plane has an inductance of approximately 0.2 to 0.5 nH, depending on the via length (board thickness) and diameter. The via is the largest single contributor to mounting inductance.

**Pad and trace inductance:** The PCB pads under the capacitor and any short traces routing from the pad to the via contribute 0.05 to 0.2 nH.

**Minimization techniques:**

Use multiple vias per pad: Two vias per pad instead of one reduces the pad-to-plane inductance by roughly half. Four vias per pad reduce it further.

Short, wide connections: Minimize the trace length and maximize the trace width between the capacitor pad and the via.

Via-in-pad: Place the via directly in the capacitor pad (requires via filling or capping to ensure solderable surface). This eliminates the pad-to-via trace entirely.

Backside mounting: For capacitors directly under the BGA footprint, the via connects straight through to the power plane with minimal lateral routing.

Thin boards: A thinner PCB means shorter vias and lower inductance. However, board thickness is usually set by other constraints.

Reverse-geometry capacitors: Capacitors with terminals on the long sides rather than the short sides (e.g., 0306 instead of 0603) have a wider current path and lower body inductance.

Interdigitated capacitor (IDC) packages: Multiple alternating terminals along the capacitor body reduce both internal and mounting inductance.

For a standard 0402 MLCC mounted with dual vias per pad on a 62 mil (1.6 mm) board, the total mounting inductance is approximately 0.5 to 0.8 nH. With via-in-pad and optimized layout, this can be reduced to 0.3 to 0.5 nH.

---

### Q5. How do you design a decoupling network that avoids problematic anti-resonance peaks?

**Answer:**
Anti-resonance peaks between capacitor groups are one of the most common PDN design problems. A systematic approach to avoiding them involves several strategies.

**Use a continuous range of capacitor values.** Rather than using only two widely spaced values (e.g., 100 uF and 100 nF), use intermediate values (10 uF, 1 uF) to fill the frequency gap. Each intermediate value provides coverage in the transition region where anti-resonance would otherwise occur.

**Overlapping effective ranges.** Choose the values and quantities of each capacitor group so that the inductive crossover of the lower-value group overlaps with the capacitive region of the higher-value group. This ensures continuous coverage without gaps.

**Controlled ESR for damping.** Anti-resonance peaks are damped by resistive losses. If a simulation shows a peak, adding capacitors with slightly higher ESR (or adding a small series resistor to some capacitors) can damp the peak. This raises the impedance at the capacitors' resonant frequency but reduces the anti-resonance peak height. The net result is usually a flatter, lower impedance profile.

**Simulation-driven optimization.** The most reliable approach is to model the complete decoupling network in a circuit simulator and iterate on the capacitor values and quantities until the impedance profile is flat and below the target. Many PDN analysis tools have built-in optimization routines that automatically select the best combination of available capacitor values and quantities to minimize the peak impedance.

**Avoid identical capacitors of the same value.** Manufacturing tolerance causes slight variations in capacitance between individual MLCCs. This natural spread actually helps reduce anti-resonance peaks because the resonant frequencies of individual caps are slightly different, which smears out both the resonance and anti-resonance. In simulation, this effect can be modeled by using a Gaussian distribution of capacitance values.

**Rule of thumb for anti-resonance prevention:** Ensure that adjacent capacitor values differ by no more than a factor of 10 (one decade). A factor of 5 spacing (e.g., 100 uF, 22 uF, 4.7 uF, 1 uF, 220 nF, 47 nF, 10 nF) provides good overlap and manageable anti-resonance.

---

### Q6. What is the role of output capacitors in VRM load transient response?

**Answer:**
Output capacitors are the first responders to a load transient -- they supply charge to the load during the time it takes for the VRM control loop to react and ramp up the inductor current. The output capacitor bank largely determines the voltage droop magnitude during a transient.

When a load current step occurs, the event proceeds in three phases:

**Phase 1 (0 to ~100 ns): ESR-dominated droop.** The initial voltage drop is V = I_step * ESR_total, where ESR_total is the effective series resistance of the entire output capacitor bank. This occurs almost instantaneously because ESR limits the initial current delivery rate. For a 50 A step with ESR_total = 0.5 mOhm: V_esr = 25 mV.

**Phase 2 (~100 ns to response time): Capacitive droop.** After the ESR-limited initial response, the capacitors begin discharging to supply the deficit current. The voltage drops at a rate of dV/dt = I_step / C_total. This continues until the VRM's control loop responds and ramps the inductor current. The total capacitive droop is approximately V_cap = I_step * t_response / C_total, where t_response is the VRM response time (~1/(2*pi*BW)).

**Phase 3 (after response time): Recovery.** The inductor current catches up to the load current, and the capacitors recharge. The voltage recovers toward the new steady-state value (which, with AVP, is lower than the initial voltage by R_LL * I_step).

The total voltage droop is approximately:

```
V_droop = V_esr + V_cap = I_step * ESR + I_step^2 * L / (2 * C * (Vin - Vout))
```

For a typical high-current design, the output capacitor bank might consist of 4 polymer caps (470 uF, 8 mOhm each) and 40 ceramic MLCCs (22 uF, 3 mOhm each):
- Polymer: 1880 uF, ESR = 8/4 = 2 mOhm
- Ceramic: 880 uF (after derating), ESR = 3/40 = 0.075 mOhm
- Combined: 2760 uF, ESR ~ 0.073 mOhm (parallel combination)

This provides excellent ESR-limited droop (50 A * 0.073 mOhm = 3.65 mV) and capacitive energy storage.

---

### Q7. How does the physical size of an MLCC affect its electrical performance?

**Answer:**
MLCC physical size (specified as a standardized code: 0201, 0402, 0603, 0805, 1206, 1210, etc.) affects every electrical parameter.

**Capacitance range:** Larger packages can accommodate more dielectric layers and thus higher capacitance. Maximum capacitance per size:
- 0201 (0.6mm x 0.3mm): up to 1 uF (X5R)
- 0402 (1.0mm x 0.5mm): up to 10 uF
- 0805 (2.0mm x 1.25mm): up to 47 uF
- 1210 (3.2mm x 2.5mm): up to 100 uF

**ESL:** Smaller packages have shorter current paths and thus lower intrinsic ESL. A 0201 MLCC has approximately 50% lower body ESL than a 1206 MLCC. For high-frequency decoupling, smaller packages are preferred.

**ESR:** ESR depends more on the dielectric type and number of layers than on the package size. Smaller packages may have slightly higher ESR due to thinner electrode layers.

**Self-resonant frequency:** Smaller packages (with lower ESL) have higher self-resonant frequencies for the same capacitance value. A 100 nF 0201 resonates at a higher frequency than a 100 nF 0805.

**Current handling:** Smaller packages have higher thermal resistance (less surface area for heat dissipation), limiting the ripple current they can handle.

**Mechanical reliability:** Smaller packages are more resistant to flex cracking because they deform less under board flexure. This is important for large boards that may flex during assembly or thermal cycling.

For power delivery, the trend is toward using many small MLCCs (0201, 0402) rather than fewer large ones (0805, 1210) because the small packages provide lower ESL, better frequency response, and can be placed closer to the BGA footprint.

---

### Q8. What is impedance-frequency overlap and why is it important in decoupling network design?

**Answer:**
Impedance-frequency overlap refers to the frequency range where two adjacent decoupling elements (e.g., a bulk capacitor and a ceramic capacitor) are both simultaneously providing low impedance. In the overlap region, both elements are below their respective inductive crossover points and actively contribute to the parallel impedance.

Without overlap, there is a frequency gap where the lower-frequency element has become inductive (rising impedance) while the higher-frequency element is still capacitive but has not yet reached its low-impedance region. In this gap, the impedance may exceed the target, especially if an anti-resonance peak forms.

With overlap, both elements are effective simultaneously, and the parallel combination maintains low impedance through the transition. The overlap occurs when the inductive crossover frequency of the lower element (f_high_low = Ztarget / (2*pi*ESL_low)) exceeds the capacitive crossover of the higher element (f_low_high = 1 / (2*pi*C_high*Ztarget)).

Ensuring overlap requires:

```
f_high_low > f_low_high
Ztarget * N_low / (2*pi*ESL_low) > 1 / (2*pi * N_high * C_high * Ztarget)
```

Rearranging:

```
N_low * N_high > ESL_low * C_high * (1/Ztarget)^2 * ... 
```

In practice, the number of capacitors in each group must be large enough that their collective frequency ranges overlap. The more capacitors, the wider each group's effective range, and the better the overlap.

This is a quantitative way to verify that a decoupling network provides continuous coverage. If the overlap check fails for any pair of adjacent groups, more capacitors or intermediate values must be added.

---

### Q9. How do temperature and aging affect decoupling capacitor performance?

**Answer:**
Both temperature and aging cause capacitor parameters to drift from their initial values, and PDN design must account for these variations.

**Temperature effects on MLCCs:** Class II dielectrics (X5R, X7R) have specified capacitance variation with temperature: X5R allows +/-15% over -55C to 85C, X7R allows +/-15% over -55C to 125C. Outside these ranges, the change can be larger. ESR generally decreases slightly with temperature. Voltage rating is not affected within the operating range.

**Aging of MLCCs:** Ceramic capacitors exhibit dielectric aging -- a gradual decrease in capacitance over time due to relaxation of the ferroelectric domain structure. The aging rate is typically 2 to 5% per decade of time (e.g., a 5% per decade cap loses 5% from 1 hour to 10 hours, another 5% from 10 hours to 100 hours, etc.). The total aging loss over 10 years can be 10 to 20%. Aging can be partially reversed by heating the capacitor above its Curie temperature (which occurs during solder reflow), after which the aging clock restarts.

**Temperature effects on polymer capacitors:** Polymer caps have a relatively flat capacitance versus temperature curve but ESR increases at low temperatures (polymer conductivity decreases). At -40C, the ESR may be 2 to 3 times the room temperature value.

**Aging of polymer and tantalum capacitors:** These technologies show gradual increase in ESR and slight decrease in capacitance over their lifetime, typically 10 to 20% change over 10 years.

**Design practice:** To ensure the PDN meets specifications over the full product lifetime and operating temperature range, designers apply worst-case derating factors:

| Factor | MLCC (X5R) | Polymer |
|--------|-----------|---------|
| Temperature | -15% to +15% | -5% to +5% |
| DC bias | -10% to -50% | N/A |
| Aging (10 year) | -10% to -20% | -10% |
| Tolerance | +/-10% or +/-20% | +/-20% |
| **Total worst case** | **-50% to -70%** | **-30%** |

Using 50% of nominal MLCC capacitance and 70% of nominal polymer capacitance in PDN simulation provides a conservative design that meets specifications under worst-case conditions.

---

### Q10. How do you select between using many small capacitors versus fewer large capacitors?

**Answer:**
This is a common PDN design tradeoff with implications for impedance performance, cost, board area, and reliability.

**Many small capacitors (e.g., 100 x 100 nF 0201):**
- Very low ESL per cap (short body, short mounting), and collective ESL = ESL_single / N is extremely low
- High self-resonant frequency, effective at higher frequencies
- Distributed placement possible (spread across the board near BGA)
- Low total capacitance (100 x 100 nF = 10 uF) -- may not provide enough charge for slow transients
- Higher assembly cost (more components to place)
- Higher PCB area consumption (100 component footprints plus vias)

**Fewer large capacitors (e.g., 10 x 10 uF 0805):**
- Higher capacitance (100 uF total) -- better for low-frequency decoupling and transient charge
- Higher ESL per cap (larger body) and collective ESL = ESL_single / N is moderate
- Lower self-resonant frequency, effective at lower frequencies
- Fewer components to place (lower assembly cost)
- Less PCB area consumed (10 footprints)
- Cannot be placed as close to BGA (larger footprints need more clearance)

**Optimal approach:** Use a mix of both, allocated by frequency range:
- Large caps (10-22 uF, 0805-1210) for low-frequency decoupling near the VRM
- Medium caps (1-10 uF, 0402) for mid-frequency decoupling near the BGA
- Small caps (100 nF, 0201) for high-frequency decoupling directly under the BGA

This provides continuous frequency coverage while managing cost and board area. The exact split depends on the target impedance, available space, and budget. Simulation-driven optimization tools can determine the most cost-effective combination.
