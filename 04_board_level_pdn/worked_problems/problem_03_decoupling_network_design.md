# Worked Problem 03: Decoupling Network Design

## Problem Statement

Design a PCB decoupling capacitor network for an SoC VCORE rail with:

- Vdd = 0.85 V, Imax = 40 A, ripple budget = 5%
- Target impedance: Ztarget = 0.85 * 0.05 / 40 = 1.0625 mOhm
- VRM bandwidth: 200 kHz
- Package decoupling effective above: 30 MHz
- PCB caps must cover: 200 kHz to 30 MHz

Available capacitors (all with standard 2-via mounting, ESL_mount = 0.8 nH):

| Type | Value | ESR | Body ESL | Total ESL |
|------|-------|-----|----------|-----------|
| Polymer | 100 uF | 8 mOhm | 1.5 nH | 2.3 nH |
| MLCC | 22 uF | 3 mOhm | 0.3 nH | 1.1 nH |
| MLCC | 1 uF | 2 mOhm | 0.2 nH | 1.0 nH |
| MLCC | 100 nF | 1.5 mOhm | 0.1 nH | 0.9 nH |

Select quantities of each to meet the target impedance from 200 kHz to 30 MHz.

---

## Worked Solution

### Step 1: Calculate series resonant frequency for each type

```
f_res = 1 / (2 * pi * sqrt(ESL * C))
```

| Type | C | ESL | f_res |
|------|---|-----|-------|
| 100 uF polymer | 100 uF | 2.3 nH | 332 kHz |
| 22 uF MLCC | 22 uF | 1.1 nH | 1.02 MHz |
| 1 uF MLCC | 1 uF | 1.0 nH | 5.03 MHz |
| 100 nF MLCC | 100 nF | 0.9 nH | 16.8 MHz |

These resonant frequencies span the target range (200 kHz to 30 MHz).

### Step 2: Determine minimum quantities at resonance

At resonance, Z = ESR/N. Need Z <= 1.0625 mOhm:

| Type | ESR | N_min = ceil(ESR/Ztarget) |
|------|-----|---------------------------|
| 100 uF polymer | 8 mOhm | 8 |
| 22 uF MLCC | 3 mOhm | 3 |
| 1 uF MLCC | 2 mOhm | 2 |
| 100 nF MLCC | 1.5 mOhm | 2 |

### Step 3: Determine quantities for frequency coverage

The ESR minimums are for the resonance point only. We also need enough capacitors so that the inductive impedance remains below target above resonance.

For each group, the frequency where inductive impedance = Ztarget:

```
f_high = Ztarget * N / (2 * pi * ESL)
```

We need f_high of each group to at least reach the resonant frequency of the next group.

**Polymer (f_res = 332 kHz, next group resonance = 1.02 MHz):**

```
N_polymer >= 2 * pi * f_target * ESL / Ztarget
N_polymer >= 2 * pi * 1.02e6 * 2.3e-9 / 1.0625e-3
N_polymer >= 14.74e-3 / 1.0625e-3
N_polymer >= 13.9 -> 14
```

**22 uF MLCC (f_res = 1.02 MHz, next = 5.03 MHz):**

```
N_22uF >= 2 * pi * 5.03e6 * 1.1e-9 / 1.0625e-3
N_22uF >= 34.74e-3 / 1.0625e-3
N_22uF >= 32.7 -> 33
```

**1 uF MLCC (f_res = 5.03 MHz, next = 16.8 MHz):**

```
N_1uF >= 2 * pi * 16.8e6 * 1.0e-9 / 1.0625e-3
N_1uF >= 105.6e-3 / 1.0625e-3
N_1uF >= 99.4 -> 100
```

**100 nF MLCC (f_res = 16.8 MHz, must reach 30 MHz):**

```
N_100nF >= 2 * pi * 30e6 * 0.9e-9 / 1.0625e-3
N_100nF >= 169.6e-3 / 1.0625e-3
N_100nF >= 159.6 -> 160
```

### Step 4: Review the total count

| Type | Quantity | Purpose |
|------|----------|---------|
| 100 uF polymer | 14 | 200 kHz - 1 MHz |
| 22 uF MLCC | 33 | 500 kHz - 5 MHz |
| 1 uF MLCC | 100 | 2 MHz - 17 MHz |
| 100 nF MLCC | 160 | 8 MHz - 30 MHz |
| **Total** | **307** | **200 kHz - 30 MHz** |

This is a large number of capacitors, which is realistic for a high-performance SoC with sub-milliohm target impedance. The 1 uF and 100 nF capacitors dominate because the inductive impedance of a single capacitor at frequencies well above resonance is significant.

### Step 5: Verify at critical frequencies

**At 500 kHz:** Polymer caps are near resonance. Z_polymer = 8/14 = 0.571 mOhm. Passes.

**At 3 MHz:** 22 uF MLCCs are inductive: Z = 2*pi*3e6*(1.1e-9/33) = 629 uOhm = 0.629 mOhm. 1 uF MLCCs are capacitive: Z = 1/(2*pi*3e6*100e-6) = 531 uOhm = 0.531 mOhm. Parallel: 0.288 mOhm. Passes.

**At 10 MHz:** 1 uF caps near resonance (inductive side): Z = 2*pi*10e6*(1.0e-9/100) = 628 uOhm. 100 nF caps capacitive: Z = 1/(2*pi*10e6*160*100e-9) = 99.5 uOhm. Parallel: ~86 uOhm. Passes.

**At 30 MHz:** 100 nF caps inductive: Z = 2*pi*30e6*(0.9e-9/160) = 1.06 mOhm. Marginal -- exactly at target.

### Summary

| Parameter | Value |
|-----------|-------|
| Target impedance | 1.06 mOhm |
| Total PCB caps | 307 |
| Frequency coverage | 200 kHz to 30 MHz |
| Estimated BOM cost | $15-30 (caps only) |
| Board area | ~10 cm^2 |

This problem demonstrates why high-current, low-voltage SoC power delivery requires hundreds of decoupling capacitors on the PCB alone.
