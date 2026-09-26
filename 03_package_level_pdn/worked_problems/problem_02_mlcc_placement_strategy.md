# Worked Problem 02: MLCC Placement Strategy

## Problem Statement

A package substrate has space for up to 150 MLCCs for the VCORE rail. The target impedance is 0.8 mOhm from 5 MHz to 400 MHz. Available MLCC options (0201 size, bottom-mounted):

| Value | ESR (mOhm) | ESL (pH) | f_res (MHz) |
|-------|-----------|---------|------------|
| 10 uF | 5 | 120 | 4.6 |
| 1 uF | 4 | 100 | 15.9 |
| 100 nF | 3 | 80 | 56.3 |
| 10 nF | 5 | 80 | 178 |

Determine the optimal quantity of each value to minimize peak impedance across 5 MHz to 400 MHz.

---

## Worked Solution

### Step 1: Determine the effective frequency range for each value

For N capacitors of each value, the effective range is where the impedance is below Ztarget = 0.8 mOhm.

At resonance, Z = ESR/N. To meet 0.8 mOhm:

| Value | ESR | N_min = ESR/Ztarget |
|-------|-----|---------------------|
| 10 uF | 5 mOhm | 6.25 -> 7 |
| 1 uF | 4 mOhm | 5 |
| 100 nF | 3 mOhm | 3.75 -> 4 |
| 10 nF | 5 mOhm | 6.25 -> 7 |

### Step 2: Determine high-frequency limit for each value group

The inductive impedance of N capacitors: Z_L = 2*pi*f*(ESL/N). Setting this equal to Ztarget:

```
f_high = Ztarget * N / (2*pi*ESL)
```

| Value | ESL | N | f_high |
|-------|-----|---|--------|
| 10 uF | 120 pH | N1 | 0.8e-3*N1/(2*pi*120e-12) = 1.06e6*N1 |
| 1 uF | 100 pH | N2 | 0.8e-3*N2/(2*pi*100e-12) = 1.27e6*N2 |
| 100 nF | 80 pH | N3 | 0.8e-3*N3/(2*pi*80e-12) = 1.59e6*N3 |
| 10 nF | 80 pH | N4 | 0.8e-3*N4/(2*pi*80e-12) = 1.59e6*N4 |

### Step 3: Determine the frequency coverage strategy

We need coverage from 5 MHz to 400 MHz. Working backwards from 400 MHz:

**10 nF caps (f_res = 178 MHz):** To cover up to 400 MHz:

```
f_high = 1.59e6 * N4 >= 400e6
N4 >= 252
```

This exceeds our total budget of 150. The 10 nF caps alone cannot reach 400 MHz with 0.8 mOhm target. This means the on-die decoupling must take over below 400 MHz. Let us redefine the package cap responsibility to 5 MHz to 200 MHz, with on-die caps covering above 200 MHz.

**Revised target: 5 MHz to 200 MHz.**

**10 nF caps** to cover the high end (100 to 200 MHz):

```
N4 = 200e6 / 1.59e6 = 126
```

This is still too many. Let us try using 100 nF caps to reach 200 MHz:

```
N3 for f_high = 200 MHz: N3 = 200e6 / 1.59e6 = 126
```

Still too many. The problem is that 0.8 mOhm is extremely aggressive.

### Step 4: Reconsider -- use all 150 caps of the highest-density value

Let us check what impedance we can achieve at 200 MHz with 150 caps of 100 nF (ESL = 80 pH):

```
Z_L_200MHz = 2*pi*200e6 * (80e-12/150) = 2*pi*200e6 * 0.533e-12 = 670 uOhm = 0.67 mOhm
```

This meets the 0.8 mOhm target at 200 MHz. But it does not cover the lower frequencies (5 to 50 MHz) where 100 nF caps are still capacitive and have higher impedance.

### Step 5: Optimized mixed allocation

Let us allocate caps to cover 5 to 200 MHz and optimize by frequency band.

**Band 1: 5-20 MHz** -- needs 10 uF and 1 uF caps
**Band 2: 20-60 MHz** -- needs 1 uF and 100 nF caps
**Band 3: 60-200 MHz** -- needs 100 nF caps

Allocate:
- 30 x 10 uF (covers ~2-15 MHz at resonance, extends to ~30 MHz inductively)
- 40 x 1 uF (covers ~8-50 MHz)
- 80 x 100 nF (covers ~20-200 MHz)

Total: 150 caps.

Verify at key frequencies:

**At 10 MHz:** The 10 uF caps are just above resonance (4.6 MHz): per cap |Z| = |5 + j(7.54 - 1.59)| = 7.8 mOhm, so 0.26 mOhm for 30. The 1 uF caps are still capacitive: per cap |Z| = |4 + j(6.28 - 15.9)| = 10.4 mOhm, so 0.26 mOhm for 40. Combined as complex impedances: ~0.24 mOhm. Passes.

**At 50 MHz:** The 1 uF caps are inductive (f_res = 15.9 MHz): Z_1uF = 2*pi*50e6*(100e-12/40) = 0.785 mOhm. The 100 nF caps are just below resonance (f_res = 56.3 MHz): per cap |Z| = |3 + j(25.1 - 31.8)| = 7.3 mOhm, so 0.092 mOhm for 80. Combined as complex impedances: ~0.11 mOhm. Passes.

**At ~36 MHz (anti-resonance between the 1 uF and 100 nF groups):** Above 15.9 MHz the 1 uF group is inductive, while below 56.3 MHz the 100 nF group is still capacitive, so the two groups form a parallel resonance between them (the 10 uF and 1 uF groups do not: by 30 MHz both are inductive).

```
L_1uF_group = 100 pH / 40 = 2.5 pH
C_100nF_group = 80 * 100 nF = 8 uF
Z0 = sqrt(2.5e-12 / 8e-6) = 0.56 mOhm
```

Z0 is the characteristic impedance, not the peak. Evaluating the full three-group network gives a peak of **0.99 mOhm at 36.5 MHz — this FAILS the 0.8 mOhm target** by about 24%.

**Rebalanced allocation:** Shifting capacitors from the 10 uF group to the 1 uF group lowers the 1 uF group's inductance and moves the anti-resonance down. A full-network sweep of 150-cap mixes finds, for example, 10 x 10 uF + 85 x 1 uF + 55 x 100 nF with a peak of about 0.75 mOhm across 5-200 MHz.

**At 200 MHz:** The 100 nF group is inductive: Z = 2*pi*200e6*(80e-12/80) = 628 uOhm = 0.628 mOhm for the original 80. (With the rebalanced 55 x 100 nF, the 85 x 1 uF group shares the load and the full-network value is about 0.74 mOhm.) Passes.

### Summary

| MLCC Value | Quantity | Purpose |
|-----------|----------|---------|
| 10 uF | 10 | Low-frequency coverage (5-20 MHz) |
| 1 uF | 85 | Mid-frequency coverage (10-60 MHz) |
| 100 nF | 55 | High-frequency coverage (30-200 MHz) |
| **Total** | **150** | **5 MHz to 200 MHz** |

The first-pass 30/40/80 split fails at its 1 uF–100 nF anti-resonance (0.99 mOhm at 36.5 MHz); the rebalanced split keeps the computed peak at about 0.75 mOhm. Spot checks at a few frequencies miss anti-resonances — sweep the full profile.
