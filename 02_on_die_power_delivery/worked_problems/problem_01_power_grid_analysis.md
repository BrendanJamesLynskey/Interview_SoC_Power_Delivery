# Worked Problem 01: Power Grid Analysis

## Problem Statement

A processor core occupies a 4 mm x 4 mm area on a die. The power grid uses two upper metal layers:

- Metal 10 (horizontal): sheet resistance Rsh = 5 mOhm/sq, pitch = 20 um (50% allocated to VDD, 50% to VSS)
- Metal 11 (vertical): sheet resistance Rsh = 3 mOhm/sq, pitch = 20 um (50% allocated to VDD, 50% to VSS)

Power bumps are on a 200 um pitch grid across the die (VDD and VSS alternating). Each bump has 15 mOhm resistance.

The core draws 30 A uniformly distributed across its area.

1. Calculate the number of VDD power bumps.
2. Calculate the average current per VDD bump.
3. Estimate the bump contribution to IR drop.
4. Estimate the worst-case IR drop from grid resistance at a point equidistant from four bumps.

---

## Worked Solution

### Step 1: Count the VDD power bumps

Bump pitch is 200 um. Over a 4 mm x 4 mm area:

```
Bumps per row = 4000 um / 200 um + 1 = 21
Bumps per column = 21
Total bumps = 21 * 21 = 441
```

Assuming a checkerboard pattern where half the bumps are VDD and half are VSS:

```
VDD bumps = 441 / 2 ~ 220 (rounding; actual count depends on corner assignment)
```

### Step 2: Average current per VDD bump

```
I_per_bump = I_total / N_vdd_bumps = 30 A / 220 = 136 mA per bump
```

### Step 3: Bump contribution to IR drop

The bumps are in parallel, so the effective bump resistance is:

```
R_bump_eff = R_bump / N = 15 mOhm / 220 = 0.068 mOhm
```

Average bump IR drop:

```
V_bump = I_total * R_bump_eff = 30 * 0.068e-3 = 2.05 mV
```

However, the current is not perfectly uniform -- bumps in high-current areas carry more. The worst-case bump might carry 2x the average:

```
V_bump_worst = 2 * 136 mA * 15 mOhm = 4.1 mV
```

### Step 4: Grid resistance IR drop estimation

Consider a point at the center of a 200 um x 200 um cell defined by four neighboring VDD bumps (on a checkerboard, the VDD bump spacing is 200*sqrt(2) um ~ 283 um diagonally, but let us simplify to a 400 um x 400 um VDD-only grid -- every other bump is VDD, so VDD bumps are on a 400 um pitch if in a checkerboard).

Actually, with a checkerboard pattern, each VDD bump is surrounded by VSS bumps at 200 um spacing, and the next VDD bump is at 200*sqrt(2) = 283 um diagonally or 400 um in the same row/column.

Consider the worst case: a point 200 um from the nearest VDD bump (the center of a 400 um x 400 um square of VDD bumps).

Current flowing through the grid from the nearest VDD bump to this point must traverse approximately 200 um of grid.

The M10 VDD stripe width at 20 um pitch with 50% VDD allocation: 10 um wide stripes, one every 20 um. Over a 400 um span, there are 400/20 = 20 VDD stripes.

Each M10 stripe carrying current over 200 um:

```
R_stripe_M10 = Rsh * L / W = 5e-3 * 200 / 10 = 100 mOhm per stripe
```

With 20 parallel stripes:

```
R_M10_parallel = 100 mOhm / 20 = 5 mOhm
```

Similarly for M11 (vertical):

```
R_stripe_M11 = 3e-3 * 200 / 10 = 60 mOhm per stripe
R_M11_parallel = 60 mOhm / 20 = 3 mOhm
```

The M10 and M11 grids work together (parallel paths in a 2D mesh), so the effective grid resistance is approximately:

```
R_grid ~ R_M10 || R_M11 = (5 * 3) / (5 + 3) = 1.875 mOhm
```

(This is a rough estimate; the actual mesh resistance depends on the current distribution pattern.)

The current flowing to the worst-case point: if the point has a local current density equal to the average:

```
J_area = 30 A / (4mm * 4mm) = 1.875 A/mm^2 = 1.875 uA/um^2
```

Let us compute the current drawn by the 400 um x 400 um cell:

```
I_cell = 30 A * (400 * 400) / (4000 * 4000) = 30 * 0.16e6 / 16e6 = 30 * 0.01 = 0.3 A
```

Each cell draws 0.3 A, supplied by 4 surrounding VDD bumps. Each bump supplies roughly 0.3/4 = 0.075 A to this cell. The current at the center of the cell has traversed approximately 200 um of grid.

The IR drop from grid resistance at the worst-case point:

```
V_grid = I_cell/4 * R_grid = 0.075 * 1.875e-3 = 0.14 mV
```

Wait -- this seems too low. The issue is that the 1.875 mOhm is the parallel resistance of all 20 stripes. But the current from one bump flows through a distributed network. Let us reconsider.

A more accurate simplified model: the current from one bump feeds a 200 um x 200 um quadrant. The total current in this quadrant is I_cell/4 = 75 mA. This current flows through the grid from the bump at the corner to the center, which is about 200 um away.

Using a 2D grid approximation, the resistance from a corner of a square to its center for a uniform mesh is approximately:

```
R_corner_to_center ~ (Rsh_eff / pi) * ln(d/r0)
```

where Rsh_eff is the effective sheet resistance of the combined power grid, d is the distance (~200 um), and r0 is the bump pad radius (~50 um).

The effective sheet resistance of the power grid (both M10 and M11 combined):

```
Grid coverage on M10: 50% for VDD = 10um stripes at 20um pitch
Effective Rsh_M10 = Rsh_metal / coverage = 5 mOhm/sq / 0.5 = 10 mOhm/sq

Grid coverage on M11: 50% for VDD
Effective Rsh_M11 = 3 mOhm/sq / 0.5 = 6 mOhm/sq

Combined (parallel): Rsh_combined = (10 * 6) / (10 + 6) = 3.75 mOhm/sq
```

```
R = (3.75e-3 / pi) * ln(200/50) = 1.194e-3 * 1.386 = 1.66 mOhm
```

IR drop from the nearest bump to the center:

```
V_grid = I_quadrant * R = 75 mA * 1.66 mOhm = 0.124 mV
```

### Step 5: Total IR drop estimate

```
V_total = V_bump + V_grid = 4.1 mV + 0.124 mV ~ 4.2 mV
```

For a 0.85 V supply, this is 4.2/850 = 0.49% of VDD -- well within the typical 3-5% budget.

### Summary

| Component | IR Drop (mV) |
|-----------|-------------|
| Bump resistance (worst case) | 4.1 |
| Grid resistance (worst case) | 0.12 |
| **Total estimated** | **4.2** |
| Fraction of 0.85 V VDD | 0.49% |

This analysis shows that for this grid configuration with dense bumps, the IR drop is dominated by the bump resistance. In designs with fewer bumps or higher current density, the grid resistance contribution would be larger. Note that this estimate ignores the via stack resistance and M1 rail resistance, which can add 5-20 mV in practice.
