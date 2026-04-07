# Worked Problem 03: EM Reliability Check

## Problem Statement

A power grid stripe on Metal 9 has the following characteristics:

- Width: 4 um
- Thickness: 0.9 um
- Length: 3 mm (spans the width of a processor core)
- Material: copper, resistivity rho = 2.2 uOhm-cm
- Temperature: 105C (378 K)
- EM limit for M9 at 105C, 10-year lifetime: J_max = 1.5 mA/um^2

The stripe carries current from a power bump to the farthest edge of the core. The current load is 25 mA (average DC) from the cells connected along its length, uniformly distributed.

1. Calculate the current density at the bump end (worst case) and verify EM compliance.
2. Calculate the IR drop along the stripe.
3. Determine the Blech length for this stripe at the operating current density.
4. If the EM check fails, determine the minimum stripe width to pass.

---

## Worked Solution

### Step 1: Determine the current distribution along the stripe

Since the 25 mA load is uniformly distributed along the 3 mm length, the current in the stripe is maximum at the bump end (carrying the full 25 mA) and decreases linearly to zero at the far end.

Current at position x from the bump end (x = 0 at bump, x = 3 mm at far end):

```
I(x) = 25 mA * (1 - x/3000 um)
```

Maximum current = 25 mA at x = 0.

### Step 2: Calculate the current density at the bump end

Cross-sectional area:

```
A_cross = W * T = 4 um * 0.9 um = 3.6 um^2
```

Current density at the bump end:

```
J_max_actual = I_max / A_cross = 25 mA / 3.6 um^2 = 6.94 mA/um^2
```

### Step 3: Compare to EM limit

```
J_max_actual = 6.94 mA/um^2
J_max_allowed = 1.5 mA/um^2
EM margin = J_max_allowed / J_max_actual = 1.5 / 6.94 = 0.216
```

The EM margin is 0.216, which is far below 1.0. This is an EM violation -- the stripe is carrying 4.6 times its EM current density limit.

### Step 4: Calculate the IR drop along the stripe

The resistance per unit length of the stripe:

```
R_per_um = rho / (W * T) = 2.2e-8 ohm-cm / (4e-4 cm * 0.9e-4 cm)
R_per_um = 2.2e-8 / (3.6e-8) = 0.611 ohm/cm = 6.11 mOhm/mm = 6.11e-3 mOhm/um
```

Wait, let us be more careful with units:

```
rho = 2.2 uOhm-cm = 2.2e-6 Ohm-cm = 2.2e-8 Ohm-m

R_per_unit_length = rho / A_cross
A_cross = 4 um * 0.9 um = 3.6 um^2 = 3.6e-12 m^2

R_per_meter = 2.2e-8 / 3.6e-12 = 6111 Ohm/m = 6.111 Ohm/mm
```

That seems high. Let us double-check using sheet resistance:

```
Rsh = rho / T = 2.2e-6 Ohm-cm / (0.9e-4 cm) = 24.4 mOhm/sq

R_stripe = Rsh * L / W = 24.4e-3 * (3000 um / 4 um) = 24.4e-3 * 750 = 18.3 Ohm
```

This is the total resistance if the full current flowed through the entire length. But with uniformly distributed load, the current decreases linearly, so the IR drop is:

```
V_drop = integral from 0 to L of I(x) * R_per_dx * dx
       = integral from 0 to L of [I_total * (1 - x/L)] * [Rsh / W] * dx
       = (I_total * Rsh / W) * integral from 0 to L of (1 - x/L) dx
       = (I_total * Rsh / W) * [L - L/2]
       = (I_total * Rsh / W) * L/2
       = (I_total * Rsh * L) / (2 * W)
```

```
V_drop = (25e-3 * 24.4e-3 * 3000) / (2 * 4)
       = (25e-3 * 73.2) / 8
       = 1.83 / 8
       = 0.229 V = 229 mV
```

This is an enormous IR drop (229 mV on a 0.8 V rail = 28.6% of VDD). However, this assumes a single stripe carries the entire 25 mA, which would not happen in practice -- there would be many parallel stripes. This problem illustrates the analysis for a single stripe. In a real grid with, say, 50 parallel stripes, each would carry 0.5 mA and the effective IR drop would be 50x less.

For the single-stripe scenario, the 229 mV drop is clearly unacceptable.

### Step 5: Determine the Blech length

The Blech product for copper is approximately (J * L)_crit = 4000 A/cm (a typical value):

```
L_Blech = (J * L)_crit / J_actual

J_actual at bump end = 6.94 mA/um^2 = 6.94e-3 A / (1e-8 cm^2) = 6.94e5 A/cm^2

L_Blech = 4000 A/cm / 6.94e5 A/cm^2 = 5.76e-3 cm = 57.6 um
```

Since the stripe length (3000 um) is much longer than the Blech length (57.6 um), the short-length effect does not provide relief, and full EM rules apply.

### Step 6: Determine minimum width to pass EM

To meet the EM limit, the current density must be below J_max_allowed:

```
J = I_max / (W * T) <= J_max_allowed
W >= I_max / (J_max_allowed * T)
W >= 25 mA / (1.5 mA/um^2 * 0.9 um)
W >= 25 / 1.35
W >= 18.5 um
```

The stripe must be at least 18.5 um wide to pass EM at 25 mA. This is 4.6x the original 4 um width, consistent with the EM margin of 0.216.

Alternatively, the current could be distributed across multiple parallel stripes:

```
N_stripes = ceil(I_total / (J_max * W * T))
N_stripes = ceil(25 / (1.5 * 4 * 0.9))
N_stripes = ceil(25 / 5.4)
N_stripes = ceil(4.63) = 5 parallel stripes
```

Five parallel 4 um stripes would share the 25 mA, with each carrying 5 mA, giving J = 5/3.6 = 1.39 mA/um^2 < 1.5 mA/um^2.

### Summary

| Check | Value | Limit | Status |
|-------|-------|-------|--------|
| Current density (bump end) | 6.94 mA/um^2 | 1.5 mA/um^2 | FAIL |
| EM margin | 0.216 | >= 1.0 | FAIL |
| IR drop (single stripe) | 229 mV | ~40 mV budget | FAIL |
| Blech length | 57.6 um | Stripe = 3000 um | No relief |

| Fix Option | Detail |
|-----------|--------|
| Widen stripe | >= 18.5 um (from 4 um) |
| Add parallel stripes | >= 5 stripes of 4 um each |
| Combination | 3 stripes of 7 um each |
