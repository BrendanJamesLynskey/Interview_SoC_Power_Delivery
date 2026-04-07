# Worked Problem 02: Decap Optimization

## Problem Statement

An SoC core voltage rail has the following specifications:

- Supply voltage: VDD = 0.80 V
- Target impedance: Ztarget = 1.0 mOhm
- Package decoupling effective up to: 150 MHz
- On-die decoupling must cover: 150 MHz to 2 GHz
- Available decap cells:
  - MOS decap (thin oxide): 15 fF/um^2, leakage = 30 A/cm^2 at 0.8 V
  - MOS decap (thick oxide): 4 fF/um^2, leakage = 0.01 A/cm^2 at 0.8 V
  - MOM decap: 3 fF/um^2, leakage = 0 (negligible)
- Die area: 80 mm^2
- Maximum area for intentional decaps: 10% of die area = 8 mm^2
- Maximum allowed decap leakage: 500 mW

Determine the optimal mix of decap types to maximize total on-die decoupling while meeting both area and leakage constraints.

---

## Worked Solution

### Step 1: Calculate the minimum required on-die capacitance

At the lowest frequency where on-die decaps must be effective (150 MHz), the capacitive impedance must be below Ztarget:

```
C_min = 1 / (2 * pi * f * Ztarget)
C_min = 1 / (2 * pi * 150e6 * 1.0e-3)
C_min = 1 / (942477)
C_min = 1.061 uF = 1061 nF
```

This is the minimum on-die capacitance needed to meet the impedance target at 150 MHz. At higher frequencies, the impedance of this capacitance is lower (better), so 150 MHz is the binding constraint.

### Step 2: Calculate area required for each decap type alone

If using only thin-oxide MOS decaps:

```
Area_thin = C_min / density_thin = 1061e-9 / 15e-15 = 70.7e6 um^2 = 70.7 mm^2
```

This exceeds the 8 mm^2 budget -- thin-oxide MOS alone cannot meet the requirement.

Wait -- the intrinsic gate capacitance of the logic transistors should also be included. Let us assume the design has an estimated intrinsic decoupling of 400 nF (typical for an 80 mm^2 die). Then:

```
C_intentional_needed = 1061 - 400 = 661 nF
```

If using only thin-oxide MOS decaps:

```
Area_thin = 661e-9 / 15e-15 = 44.1 mm^2  (still exceeds 8 mm^2)
```

This means we cannot meet the full 1.0 mOhm target at 150 MHz with the area budget. However, the package decoupling is still somewhat effective at 150 MHz (it is just starting to become inductive). In practice, the transition is gradual. Let us recalculate assuming the on-die decaps must achieve Ztarget at 300 MHz (with some overlap from package caps at lower frequencies):

```
C_min_300MHz = 1 / (2 * pi * 300e6 * 1.0e-3) = 531 nF
C_intentional_needed = 531 - 400 = 131 nF
```

### Step 3: Evaluate area and leakage for each decap type to provide 131 nF

**Thin-oxide MOS only:**

```
Area = 131e-9 / 15e-15 = 8.73 mm^2  (slightly exceeds 8 mm^2 limit)
Leakage_current = 30 A/cm^2 * 8.73e-2 cm^2 = 2.62 A
Leakage_power = 2.62 * 0.80 = 2.10 W  (far exceeds 500 mW limit)
```

Fails both constraints.

**Thick-oxide MOS only:**

```
Area = 131e-9 / 4e-15 = 32.75 mm^2  (far exceeds 8 mm^2 limit)
```

Fails area constraint.

**MOM only:**

```
Area = 131e-9 / 3e-15 = 43.7 mm^2  (far exceeds 8 mm^2 limit)
```

Fails area constraint.

### Step 4: Formulate the optimization problem

Let:
- A1 = area of thin-oxide MOS decaps (mm^2)
- A2 = area of thick-oxide MOS decaps (mm^2)
- A3 = area of MOM decaps (mm^2)

Constraints:

```
A1 + A2 + A3 <= 8 mm^2                           (area)
30 * A1 * 1e-2 * 0.80 + 0.01 * A2 * 1e-2 * 0.80 <= 0.500 W  (leakage)
```

Simplifying the leakage constraint (converting mm^2 to cm^2 by multiplying by 1e-2):

```
0.24 * A1 + 0.00008 * A2 <= 0.500
```

The thick-oxide leakage contribution is negligible. Effectively:

```
A1 <= 0.500 / 0.24 = 2.083 mm^2
```

Objective: maximize total capacitance:

```
C_total = 15 * A1 + 4 * A2 + 3 * A3  (in fF/um^2 * mm^2 = fF * 1e6 = nF when scaled)
C_total = 15e-15 * A1 * 1e6 + 4e-15 * A2 * 1e6 + 3e-15 * A3 * 1e6
C_total (nF) = 15 * A1 + 4 * A2 + 3 * A3  (with A in mm^2, C in nF)
```

### Step 5: Solve the optimization

Since thin-oxide MOS has the highest density, we use as much as the leakage allows:

```
A1 = 2.08 mm^2 (leakage-limited)
C1 = 15 * 2.08 = 31.2 nF
Leakage_power = 0.24 * 2.08 = 0.499 W (at limit)
```

Remaining area: 8.0 - 2.08 = 5.92 mm^2

Between thick-oxide MOS (4 fF/um^2) and MOM (3 fF/um^2), thick-oxide is denser. Use thick-oxide for the remainder (its leakage is negligible):

```
A2 = 5.92 mm^2
C2 = 4 * 5.92 = 23.7 nF
A3 = 0 mm^2
```

Total intentional decoupling:

```
C_intentional = 31.2 + 23.7 = 54.9 nF
```

### Step 6: Assess the result

Total on-die decoupling:

```
C_total_ondie = C_intrinsic + C_intentional = 400 + 54.9 = 454.9 nF
```

Impedance at 300 MHz:

```
Z_300MHz = 1 / (2 * pi * 300e6 * 454.9e-9) = 1.17 mOhm
```

This is 17% above the 1.0 mOhm target. The design is slightly short. Options to close the gap:

1. Accept the 1.17 mOhm and verify that the anti-resonance between package and die decoupling does not exceed 1.17 mOhm (may be acceptable with proper damping).
2. Reduce the package-to-die transition frequency by improving package decoupling (more MLCCs) to remain effective to higher frequencies.
3. Increase the allowed decap area or leakage budget.

### Summary

| Decap Type | Area (mm^2) | Capacitance (nF) | Leakage (mW) |
|-----------|-------------|-------------------|---------------|
| Thin-oxide MOS | 2.08 | 31.2 | 499 |
| Thick-oxide MOS | 5.92 | 23.7 | 0.5 |
| MOM | 0 | 0 | 0 |
| Intrinsic (free) | -- | 400 | -- |
| **Total** | **8.0** | **454.9** | **~500** |
| Required | -- | 531 at 300 MHz | -- |
| Gap | -- | -76 nF (short) | -- |

This problem illustrates the real-world tension between decoupling capacitance density, leakage power, and area constraints at advanced process nodes.
