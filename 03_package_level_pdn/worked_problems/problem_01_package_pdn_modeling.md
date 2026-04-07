# Worked Problem 01: Package PDN Modeling

## Problem Statement

A flip-chip BGA package has the following characteristics:

- Die size: 12 mm x 12 mm
- Package substrate: 8 layers, 35 mm x 35 mm
- Power/ground plane pair: layer 3 (VDD) and layer 4 (VSS), 40 um dielectric separation
- Dielectric constant: epsilon_r = 3.8
- Substrate copper resistivity: 2.0 uOhm-cm, copper thickness 18 um per layer
- C4 bumps: 150 um pitch, 400 VDD bumps, 400 VSS bumps, 15 mOhm each
- BGA balls: 1.0 mm pitch, 200 VDD balls, 200 VSS balls, 3 mOhm each
- Through-substrate vias: 500 power vias, 5 mOhm each

Calculate: (a) total DC resistance from BGA to die, (b) plane capacitance, (c) spreading inductance estimate, and (d) the frequency at which the plane capacitance resonates with the spreading inductance.

---

## Worked Solution

### Step 1: DC resistance -- BGA balls

With 200 VDD balls in parallel:

```
R_BGA = R_ball / N = 3 mOhm / 200 = 0.015 mOhm
```

Similarly for VSS: 0.015 mOhm.

### Step 2: DC resistance -- substrate vias

With 500 power vias in parallel:

```
R_vias = R_via / N = 5 mOhm / 500 = 0.01 mOhm
```

### Step 3: DC resistance -- plane spreading

The plane resistance depends on the current distribution pattern. For a rough estimate, we model the plane pair as a resistive sheet. The sheet resistance of the VDD plane:

```
Rsh = rho / T = 2.0e-6 Ohm-cm / (18e-4 cm) = 1.11 mOhm/sq
```

The current must spread from the die area (12 mm x 12 mm) to the BGA area (35 mm x 35 mm). This is a complex 2D current distribution problem, but a rough estimate uses the number of squares from die edge to package edge:

Average spreading distance: (35 - 12) / 2 = 11.5 mm on each side. The effective number of squares is approximately:

```
N_squares ~ ln(A_package / A_die) / (2 * pi) ~ ln(35^2 / 12^2) / (2*pi) ~ ln(8.51) / 6.28 ~ 2.14 / 6.28 ~ 0.34 squares
```

(Using the concentric ring approximation for spreading from a smaller area to a larger area.)

```
R_plane = Rsh * N_squares = 1.11 * 0.34 = 0.38 mOhm
```

This is for one plane (VDD). The VSS plane has the same resistance. Total plane resistance:

```
R_planes = 2 * 0.38 = 0.76 mOhm
```

### Step 4: DC resistance -- C4 bumps

With 400 VDD bumps in parallel:

```
R_C4 = 15 mOhm / 400 = 0.0375 mOhm
```

### Step 5: Total DC resistance

```
R_total = R_BGA + R_vias + R_planes + R_C4
R_total = 0.015 + 0.01 + 0.76 + 0.0375
R_total = 0.82 mOhm
```

The plane spreading resistance dominates the total DC resistance.

### Step 6: Plane capacitance

```
C_plane = epsilon_0 * epsilon_r * A / d

A = 35 mm * 35 mm = 1225 mm^2 = 1225e-6 m^2
d = 40 um = 40e-6 m

C_plane = 8.854e-12 * 3.8 * 1225e-6 / 40e-6
C_plane = 8.854e-12 * 3.8 * 30.625
C_plane = 8.854e-12 * 116.375
C_plane = 1030 pF ~ 1.03 nF
```

### Step 7: Spreading inductance estimate

The inductance per square of the plane pair:

```
L_sheet = mu_0 * d = 4*pi*1e-7 * 40e-6 = 50.3 pH/sq
```

Using the same number of effective squares as for resistance (0.34):

```
L_spread ~ L_sheet * N_squares = 50.3 * 0.34 = 17.1 pH
```

This is a rough estimate. Electromagnetic simulation would give a more accurate value, typically in the range of 10 to 50 pH for this type of package.

### Step 8: Plane resonant frequency

The plane capacitance and spreading inductance form a resonant circuit:

```
f_res = 1 / (2 * pi * sqrt(L_spread * C_plane))
f_res = 1 / (2 * pi * sqrt(17.1e-12 * 1.03e-9))
f_res = 1 / (2 * pi * sqrt(17.6e-21))
f_res = 1 / (2 * pi * 4.2e-11)
f_res = 1 / (2.64e-10)
f_res = 3.79 GHz
```

### Summary

| Parameter | Value |
|-----------|-------|
| Total DC resistance (BGA to die) | 0.82 mOhm |
| Dominant contributor | Plane spreading (0.76 mOhm) |
| Plane capacitance | 1.03 nF |
| Spreading inductance | ~17 pH |
| Plane resonance frequency | ~3.8 GHz |

The 0.82 mOhm total DC resistance is below a typical 1 mOhm target, so the package contributes acceptably to the DC IR drop budget. The plane resonance at 3.8 GHz is well above the frequency range where discrete decoupling is needed, so plane resonance is not a concern for this package geometry.
