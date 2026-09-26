# Worked Problem 03: Power Bump Optimization

## Problem Statement

An SoC die (10 mm x 8 mm) uses copper pillar bumps at 130 um pitch. The VCORE rail requires 35 A of current. Each bump has:

- Resistance: 10 mOhm
- Inductance: 25 pH
- EM current limit: 120 mA (DC, at 110C)

The target impedance is 1.2 mOhm. Determine:

1. The minimum number of VDD bumps for EM compliance
2. The minimum number for IR drop (bump budget = 3 mV)
3. The minimum number for inductance (at 300 MHz)
4. The bump allocation as a percentage of total available bumps
5. The impact of reducing bump pitch to 80 um

---

## Worked Solution

### Step 1: Total available bumps at 130 um pitch

```
Bumps_x = floor(10000 / 130) + 1 = 76 + 1 = 77
Bumps_y = floor(8000 / 130) + 1 = 61 + 1 = 62
Total_bumps = 77 * 62 = 4774
```

### Step 2: EM constraint

```
N_vdd_em = I_total / I_em_limit = 35 / 0.120 = 291.7 -> 292 bumps
```

Add 20% margin: 292 * 1.2 = 350 VDD bumps.

Need equal VSS bumps: 350 VSS bumps.

### Step 3: IR drop constraint

```
R_bump_total = R_bump / N <= V_budget / I_total
N >= R_bump * I_total / V_budget
N >= 10e-3 * 35 / 3e-3
N >= 116.7 -> 117 bumps
```

### Step 4: Inductance constraint

At 300 MHz, the inductive impedance of N bumps in parallel must be below Ztarget:

```
2 * pi * 300e6 * (L_bump / N) <= Ztarget
N >= 2 * pi * 300e6 * L_bump / Ztarget
N >= 2 * pi * 300e6 * 25e-12 / 1.2e-3
N >= 47.1e-3 / 1.2e-3
N >= 39.3 -> 40 bumps
```

### Step 5: Binding constraint

| Constraint | N_vdd required |
|-----------|---------------|
| EM (with margin) | 350 |
| IR drop | 117 |
| Inductance | 40 |
| **Binding** | **350 (EM)** |

### Step 6: Bump allocation

Total power/ground bumps: 350 VDD + 350 VSS = 700

```
P/G fraction = 700 / 4774 = 14.7%
Signal bumps available = 4774 - 700 = 4074
```

This is a modest P/G allocation. Some designs allocate 40-60% for power/ground. The remaining 4074 bumps are available for signal I/O.

Verify that the 350 VDD bumps provide adequate IR drop:

```
R_bump_total = 10 mOhm / 350 = 0.0286 mOhm
V_drop = 35 * 0.0286e-3 = 1.0 mV (well within 3 mV budget)
```

And inductance at 300 MHz:

```
Z_L = 2*pi*300e6 * (25e-12/350) = 2*pi*300e6 * 71.4e-15 = 134.7 uOhm = 0.135 mOhm
```

Well below the 1.2 mOhm target.

### Step 7: Impact of reducing pitch to 80 um

At 80 um pitch:

```
Bumps_x = floor(10000 / 80) + 1 = 125 + 1 = 126
Bumps_y = floor(8000 / 80) + 1 = 100 + 1 = 101
Total_bumps = 126 * 101 = 12726
```

With 2.67x more total bumps, the same 350 VDD bumps now consume only:

```
P/G fraction = 700 / 12726 = 5.5%
```

Alternatively, if we allocate the same 14.7% to power:

```
N_vdd = 0.147 * 12726 / 2 = 933 VDD bumps
```

With 933 VDD bumps:

```
R_bump = 10 mOhm / 933 = 0.0107 mOhm (IR drop = 0.38 mV)
Z_L_300MHz = 2*pi*300e6 * (25e-12/933) = 50.5 uOhm
EM current per bump = 35/933 = 37.5 mA (3.2x margin)
```

However, microbumps at 80 um pitch may have higher per-bump resistance (~15-20 mOhm) and inductance (~20 pH) due to their smaller size. Even with 15 mOhm per bump:

```
R_total = 15/933 = 0.0161 mOhm (still excellent)
```

### Summary

| Parameter | 130 um pitch | 80 um pitch |
|-----------|-------------|-------------|
| Total bumps | 4774 | 12726 |
| VDD bumps (EM-limited) | 350 | 350 (same requirement) |
| P/G fraction | 14.7% | 5.5% (or allocate more) |
| Bump IR drop | 1.0 mV | 1.0 mV (same N) |
| Inductance at 300 MHz | 0.135 mOhm | 0.135 mOhm (same N) |
| Signal bumps available | 4074 | 12026 |

The finer pitch primarily benefits signal I/O density. However, if more VDD bumps are allocated, the PDN performance improves significantly (~2.7x lower resistance and inductance).
