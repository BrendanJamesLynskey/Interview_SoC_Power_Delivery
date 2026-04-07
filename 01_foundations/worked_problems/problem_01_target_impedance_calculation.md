# Worked Problem 01: Target Impedance Calculation

## Problem Statement

A mobile SoC has the following specifications for its core voltage rail:

- Nominal supply voltage: Vdd = 0.75 V
- Maximum transient current step: Imax = 25 A
- Allowed voltage ripple: 5% of Vdd
- The PDN must meet the target impedance from DC to 500 MHz

Calculate the target impedance and determine the number of 10 uF MLCCs (ESR = 3 mOhm, ESL = 0.4 nH each) needed on the package substrate to meet the target impedance at the capacitors' effective frequency range.

---

## Worked Solution

### Step 1: Calculate the target impedance

Using the target impedance formula:

```
Ztarget = Vdd * ripple% / Imax
Ztarget = 0.75 * 0.05 / 25
Ztarget = 0.0375 / 25
Ztarget = 1.5 mOhm
```

The PDN impedance must remain below 1.5 mOhm at all frequencies from DC to 500 MHz.

### Step 2: Determine the series resonant frequency of a single 10 uF MLCC

```
f_res = 1 / (2 * pi * sqrt(ESL * C))
f_res = 1 / (2 * pi * sqrt(0.4e-9 * 10e-6))
f_res = 1 / (2 * pi * sqrt(4e-15))
f_res = 1 / (2 * pi * 63.2e-9)
f_res = 2.52 MHz
```

### Step 3: Determine impedance of a single capacitor at resonance

At series resonance, the impedance of a single capacitor equals its ESR:

```
Z_single = ESR = 3 mOhm
```

Since 3 mOhm > 1.5 mOhm (the target), a single capacitor is insufficient even at its optimal frequency.

### Step 4: Calculate the minimum number of parallel capacitors

With N identical capacitors in parallel, the impedance at resonance is ESR/N:

```
ESR / N <= Ztarget
N >= ESR / Ztarget
N >= 3 mOhm / 1.5 mOhm
N >= 2
```

At minimum, 2 capacitors are needed to meet the target at the resonant frequency.

### Step 5: Verify the effective frequency range with N capacitors

With N = 2 capacitors in parallel:

- C_total = 2 * 10 uF = 20 uF
- ESR_total = 3 mOhm / 2 = 1.5 mOhm
- ESL_total = 0.4 nH / 2 = 0.2 nH

The lower bound of the effective range (where capacitive impedance = Ztarget):

```
f_low = 1 / (2 * pi * C_total * Ztarget)
f_low = 1 / (2 * pi * 20e-6 * 1.5e-3)
f_low = 1 / (1.885e-7)
f_low = 5.31 MHz
```

The upper bound (where inductive impedance = Ztarget):

```
f_high = Ztarget / (2 * pi * ESL_total)
f_high = 1.5e-3 / (2 * pi * 0.2e-9)
f_high = 1.5e-3 / 1.257e-9
f_high = 1.19 MHz ... 
```

Wait -- this gives f_high < f_low, which means the effective range is not contiguous. Let us recalculate more carefully. The issue is that f_high as computed (where the inductive impedance crosses the target) must be higher than f_res. Let us compute the inductive impedance at various frequencies:

At f = 10 MHz: Z_inductive = 2 * pi * 10e6 * 0.2e-9 = 12.6 mOhm (exceeds target)
At f = 1.5 MHz: Z_inductive = 2 * pi * 1.5e6 * 0.2e-9 = 1.88 mOhm (exceeds target)

```
f_high = Ztarget / (2 * pi * ESL_total)
f_high = 1.5e-3 / (2 * pi * 0.2e-9)
f_high = 1.19 GHz
```

Correcting the arithmetic (note the units):

```
f_high = 1.5e-3 / (2 * pi * 0.2e-9)
f_high = 1.5e-3 / 1.257e-9
f_high = 1.19e6 Hz = 1.19 MHz
```

This is below f_res = 2.52 MHz, which means the inductive impedance already exceeds the target very close to resonance. This indicates that 2 capacitors is not sufficient for broad frequency coverage. We need more capacitors to reduce ESL_total further.

### Step 6: Determine N for adequate high-frequency coverage

We need the inductive impedance to stay below Ztarget up to at least f_res (and ideally higher). Let us find N such that the inductive impedance at, say, 50 MHz is below target:

```
2 * pi * 50e6 * (0.4e-9 / N) <= 1.5e-3
(0.4e-9 / N) <= 1.5e-3 / (2 * pi * 50e6)
(0.4e-9 / N) <= 4.77e-12
N >= 0.4e-9 / 4.77e-12
N >= 83.9
```

So approximately 84 capacitors of 10 uF are needed to keep the inductive impedance below 1.5 mOhm at 50 MHz.

### Step 7: Verify the complete impedance profile with N = 84

- C_total = 840 uF
- ESR_total = 3 mOhm / 84 = 35.7 uOhm
- ESL_total = 0.4 nH / 84 = 4.76 pH

The resonant frequency is unchanged at 2.52 MHz.

The effective frequency range:

```
f_low = 1 / (2 * pi * 840e-6 * 1.5e-3) = 126 kHz
f_high = 1.5e-3 / (2 * pi * 4.76e-12) = 50.2 MHz
```

The 84 capacitors maintain impedance below 1.5 mOhm from 126 kHz to 50.2 MHz. Coverage from 50 MHz to 500 MHz must be provided by on-die decoupling capacitance.

### Summary

| Parameter | Value |
|-----------|-------|
| Target impedance | 1.5 mOhm |
| Capacitors needed | 84 x 10 uF MLCC |
| Effective range | 126 kHz to 50.2 MHz |
| Remaining gap | 50 MHz to 500 MHz (on-die decaps) |

This result illustrates why modern SoC packages carry dozens to hundreds of decoupling capacitors, and why on-die decoupling is essential for high-frequency noise suppression.
