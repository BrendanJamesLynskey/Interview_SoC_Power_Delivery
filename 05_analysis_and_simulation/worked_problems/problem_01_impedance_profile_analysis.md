# Worked Problem 01: Impedance Profile Analysis

## Problem Statement

A PDN impedance profile measurement shows the following characteristics:

| Frequency | Impedance (mOhm) | Behavior |
|-----------|------------------|----------|
| 1 kHz | 0.3 | Flat (VRM regulating) |
| 50 kHz | 0.5 | Rising |
| 200 kHz | 2.8 | Peak (anti-resonance A) |
| 500 kHz | 0.8 | Dip (bulk cap resonance) |
| 5 MHz | 1.5 | Peak (anti-resonance B) |
| 20 MHz | 0.4 | Dip (MLCC resonance) |
| 80 MHz | 2.2 | Peak (anti-resonance C) |
| 300 MHz | 0.6 | Dip (package cap resonance) |
| 800 MHz | 1.8 | Rising (inductance) |

The target impedance is 1.5 mOhm. Identify the violations, determine which PDN elements cause each anti-resonance, and propose fixes.

---

## Worked Solution

### Step 1: Identify impedance target violations

Compare each data point to Ztarget = 1.5 mOhm:

| Frequency | Z (mOhm) | Status |
|-----------|----------|--------|
| 1 kHz | 0.3 | PASS |
| 50 kHz | 0.5 | PASS |
| 200 kHz | 2.8 | FAIL (1.87x target) |
| 500 kHz | 0.8 | PASS |
| 5 MHz | 1.5 | MARGINAL (at target) |
| 20 MHz | 0.4 | PASS |
| 80 MHz | 2.2 | FAIL (1.47x target) |
| 300 MHz | 0.6 | PASS |
| 800 MHz | 1.8 | FAIL (1.2x target) |

Three problem areas: anti-resonance A at 200 kHz, anti-resonance C at 80 MHz, and the inductive rise at 800 MHz.

### Step 2: Identify the cause of each anti-resonance

**Anti-resonance A (200 kHz):** This is between the VRM output inductance (rising above ~50 kHz, consistent with a VRM bandwidth of ~50-100 kHz) and the bulk capacitors (resonating at ~500 kHz). The VRM output inductor becomes inductive above the control bandwidth, and this inductance resonates with the bulk capacitor bank capacitance.

**Anti-resonance B (5 MHz):** This is between the bulk caps (inductive above 500 kHz) and the ceramic MLCCs (resonating at ~20 MHz). The ESL of the bulk caps resonates with the ceramic cap capacitance. This peak is right at the target (1.5 mOhm) -- marginal but technically passing.

**Anti-resonance C (80 MHz):** This is between the PCB ceramic caps (inductive above 20 MHz) and the package decoupling caps (resonating at ~300 MHz). The mounting inductance of the PCB caps resonates with the package cap capacitance.

**Rising impedance at 800 MHz:** This is the package spreading inductance dominating above the package cap resonance (300 MHz). The on-die decoupling is not providing enough capacitance to keep the impedance below target at 800 MHz.

### Step 3: Propose fixes

**Fix for anti-resonance A (200 kHz, 2.8 mOhm):**

Option 1: Increase VRM bandwidth. If the VRM bandwidth can be extended from ~50 kHz to ~200 kHz, the VRM will actively regulate at 200 kHz, suppressing the peak. This may require loop compensation redesign.

Option 2: Add more bulk caps with intermediate ESR. Adding bulk polymer caps (e.g., 6 additional 100 uF caps with ESR = 10 mOhm each) provides damping at the resonant frequency. The additional ESR damps the peak.

Required impedance reduction: from 2.8 to below 1.5 mOhm, a factor of ~1.87x. Adding 6 more bulk caps (50% more than current) should bring the peak below target through increased damping.

**Fix for anti-resonance C (80 MHz, 2.2 mOhm):**

Option 1: Add intermediate-value MLCCs (e.g., 10 nF) with resonant frequency near 80 MHz. These bridge the gap between the PCB ceramic group and the package decoupling group.

Option 2: Reduce the mounting inductance of the PCB ceramic caps (use via-in-pad, multiple vias) to push their inductive crossover to higher frequency, overlapping with the package caps.

Check Option 1 numerically: at 80 MHz with ESL = 1 nH (PCB mount), each cap has X_L = 2*pi*80e6*1e-9 = 0.503 Ohm = 503 mOhm, and X_C = 1/(2*pi*80e6*10e-9) = 199 mOhm. f_res for 10 nF with 1 nH is 50.3 MHz, so at 80 MHz the caps are inductive: net 304 mOhm per cap, or about 30 mOhm for 10 caps. That is twenty times the 1.5 mOhm target and far above the 2.2 mOhm network impedance, so ten PCB-mounted 10 nF caps would barely change the peak. Matching the target this way would take hundreds of caps: at 80 MHz, 1 nH of mounting inductance, not capacitance value, sets each cap's impedance.

Recommendation: attack the inductance (Option 2: via-in-pad, multiple vias, caps directly under the BGA) and add package-level capacitance close to the die; PCB-mounted 10 nF caps are ineffective at 80 MHz.

**Fix for 800 MHz (1.8 mOhm):**

Increase on-die decoupling capacitance. At 800 MHz:

```
Z = 1 / (2*pi*800e6*C_die) <= 1.5 mOhm
C_die >= 1 / (2*pi*800e6*1.5e-3) = 133 nF
```

If the current on-die capacitance provides 1.8 mOhm at 800 MHz:

```
C_die_current = 1 / (2*pi*800e6*1.8e-3) = 111 nF
```

Need to increase from 111 nF to 133 nF -- about 20% more on-die decoupling. This can be achieved by adding MOS or MOM decap cells.

### Summary of Fixes

| Problem | Frequency | Current Z | Fix | Target Z |
|---------|-----------|-----------|-----|----------|
| Anti-resonance A | 200 kHz | 2.8 mOhm | Add 6 bulk caps or increase VRM BW | < 1.5 mOhm |
| Anti-resonance C | 80 MHz | 2.2 mOhm | Reduce PCB cap mounting inductance; add package-level caps (PCB 10 nF caps are ~30 mOhm per 10 at 80 MHz) | < 1.5 mOhm |
| Inductive rise | 800 MHz | 1.8 mOhm | Add 20% more on-die decaps | < 1.5 mOhm |
