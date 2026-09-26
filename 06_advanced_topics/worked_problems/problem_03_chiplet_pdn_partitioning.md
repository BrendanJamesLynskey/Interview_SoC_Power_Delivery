# Worked Problem 03: Chiplet PDN Partitioning

## Problem Statement

A 2.5D chiplet system consists of 2 compute chiplets and 1 I/O chiplet on a silicon interposer:

- Compute chiplet A: 8 mm x 10 mm, 30 A at 0.80 V
- Compute chiplet B: 8 mm x 10 mm, 30 A at 0.80 V
- I/O chiplet: 6 mm x 12 mm, 10 A at 1.05 V
- Silicon interposer: 25 mm x 20 mm, 100 um thick
- Interposer TSV: 10 um diameter, copper-filled, 40 pH inductance each
- Interposer microbumps: 45 um pitch, 40 mOhm each, 25 pH each

Determine the number of power TSVs and microbumps needed for each chiplet, and calculate the interposer PDN resistance.

---

## Worked Solution

### Step 1: Calculate TSV requirements per chiplet

**TSV resistance:**

```
R_TSV = rho * h / (pi * r^2) = 1.72e-8 * 100e-6 / (pi * (5e-6)^2) = 21.9 mOhm
```

**Compute chiplet A (30 A at 0.80 V):**

EM constraint: assume TSV EM limit is 50 mA per TSV:

```
N_TSV_em = 30 / 0.050 = 600 VDD TSVs
```

IR drop constraint (TSV budget = 2 mV):

```
N_TSV_ir = R_TSV * I / V_budget = 21.9e-3 * 30 / 2e-3 = 329 VDD TSVs
```

Inductance constraint (at 500 MHz, Z_target = 1.0 mOhm):

```
N_TSV_L = 2*pi*500e6 * 40e-12 / 1.0e-3 = 126 VDD TSVs
```

Binding: EM at 600 VDD TSVs. Plus 600 VSS TSVs = 1200 total power TSVs for chiplet A.

**Chiplet B:** Same as A: 1200 power TSVs.

**I/O chiplet (10 A at 1.05 V):**

EM: 10 / 0.050 = 200 VDD TSVs.
IR: 21.9e-3 * 10 / 2e-3 = 110 VDD TSVs.
Inductance: 2*pi*500e6*40e-12/1.5e-3 = 84 VDD TSVs (relaxed target at higher voltage).

Binding: EM at 200. Total: 400 power TSVs for I/O chiplet.

### Step 2: TSV area overhead

Total power TSVs: 1200 + 1200 + 400 = 2800

With keep-out radius of 10 um around each TSV (20 um exclusion diameter):

```
Area per TSV = pi * (10e-6)^2 = 314 um^2
Total exclusion = 2800 * 314 = 0.88e6 um^2 = 0.88 mm^2
```

Interposer area: 25*20 = 500 mm^2. TSV overhead: 0.88/500 = 0.18%.

### Step 3: Calculate microbump requirements

**Compute chiplet A (8 mm x 10 mm, 45 um pitch):**

Total bumps available: (8000/45) * (10000/45) = 178 * 222 = 39,516 bumps

VDD microbumps needed (EM, assume 80 mA limit per microbump):

```
N_ubump_em = 30 / 0.080 = 375 VDD microbumps
```

IR constraint (microbump budget = 1.5 mV):

```
N_ubump_ir = 40e-3 * 30 / 1.5e-3 = 800 VDD microbumps
```

Binding: IR at 800 VDD + 800 VSS = 1600 power microbumps.

Power microbump fraction: 1600 / 39516 = 4.0%

**I/O chiplet (6 mm x 12 mm):**

Total: (6000/45) * (12000/45) = 133 * 267 = 35,511

N_ubump_ir = 40e-3 * 10 / 1.5e-3 = 267 VDD, 267 VSS = 534 total.

### Step 4: Interposer PDN total resistance per chiplet

For compute chiplet A:

| Element | R_single | N_parallel | R_total |
|---------|----------|-----------|---------|
| Package-side microbumps | 40 mOhm | 800 VDD | 0.05 mOhm |
| Interposer RDL (est.) | -- | -- | 0.3 mOhm |
| TSVs | 21.9 mOhm | 600 VDD | 0.037 mOhm |
| Chiplet-side microbumps | 40 mOhm | 800 VDD | 0.05 mOhm |
| **Total interposer path** | | | **0.44 mOhm** |

The interposer RDL resistance is estimated as 0.3 mOhm based on the RDL sheet resistance (~20 mOhm/sq for 2 um thick Cu) and the average current path length (~2 mm) with distributed current flow.

### Step 5: IR drop through interposer

```
V_IR_interposer = I * R_total = 30 * 0.44e-3 = 13.2 mV
```

This is 13.2 / 800 = 1.65% of Vdd. Acceptable but significant -- must be included in the total IR drop budget.

For the I/O chiplet, with the same 0.3 mOhm RDL estimate: 40/267 + 0.3 + 21.9/200 + 40/267 = 0.15 + 0.3 + 0.11 + 0.15 = 0.71 mOhm, so V_IR = 10 * 0.71e-3 = 7.1 mV (0.7% of 1.05 V).

### Summary

| Chiplet | Power TSVs | Power Microbumps | Interposer R | IR Drop |
|---------|-----------|-----------------|--------------|---------|
| Compute A | 1200 | 1600 | 0.44 mOhm | 13.2 mV |
| Compute B | 1200 | 1600 | 0.44 mOhm | 13.2 mV |
| I/O | 400 | 534 | 0.71 mOhm | 7.1 mV |
| **Total** | **2800** | **3734** | -- | -- |
