# Worked Problem 01: DVFS Design

## Problem Statement

A quad-core processor supports 5 DVFS operating points. Design the voltage-frequency table given:

- Process: 5 nm, Vth_nominal = 0.30 V
- Maximum frequency: 3.5 GHz at Vdd = 0.85 V
- Minimum frequency: 0.5 GHz at Vdd = 0.50 V
- Dynamic power at max: 15 W per core
- Leakage power at max voltage/temperature: 3 W per core

Calculate the power savings at each operating point and the DVFS transition time for the largest voltage step.

---

## Worked Solution

### Step 1: Define the V-F operating points

Assuming frequency scales approximately as (Vdd - Vth)^1.3 / Vdd (alpha-power model):

Normalized frequency: f_norm = ((Vdd - 0.30) / (0.85 - 0.30))^1.3 * (0.85 / Vdd)^0.3

| OP | Vdd (V) | f_norm | Frequency (GHz) |
|----|---------|--------|-----------------|
| 5 (max) | 0.85 | 1.000 | 3.50 |
| 4 | 0.75 | 0.716 | 2.51 |
| 3 | 0.65 | 0.460 | 1.61 |
| 2 | 0.55 | 0.233 | 0.82 |
| 1 (min) | 0.50 | 0.143 | 0.50 |

### Step 2: Calculate dynamic power at each point

P_dyn = P_dyn_max * (Vdd/0.85)^2 * (f/3.5)

| OP | Vdd (V) | f (GHz) | P_dyn (W) |
|----|---------|---------|-----------|
| 5 | 0.85 | 3.50 | 15.00 |
| 4 | 0.75 | 2.51 | 8.38 |
| 3 | 0.65 | 1.61 | 4.16 |
| 2 | 0.55 | 0.82 | 1.62 |
| 1 | 0.50 | 0.50 | 0.82 |

### Step 3: Estimate leakage power at each point

Leakage scales approximately as: P_leak ~ exp(-alpha * Vth / (n*Vt)) * Vdd

Where Vt = kT/q ~ 26 mV at room temperature. Simplified: leakage roughly halves for every 50 mV reduction in Vdd (due to DIBL and subthreshold effects).

| OP | Vdd (V) | P_leak (W) approx |
|----|---------|-------------------|
| 5 | 0.85 | 3.00 |
| 4 | 0.75 | 1.50 |
| 3 | 0.65 | 0.75 |
| 2 | 0.55 | 0.38 |
| 1 | 0.50 | 0.27 |

### Step 4: Total power and savings

| OP | Vdd | f | P_total (W/core) | Savings vs OP5 |
|----|-----|---|-------------------|----------------|
| 5 | 0.85 | 3.50 | 18.00 | 0% |
| 4 | 0.75 | 2.51 | 9.88 | 45% |
| 3 | 0.65 | 1.61 | 4.91 | 73% |
| 2 | 0.55 | 0.82 | 2.00 | 89% |
| 1 | 0.50 | 0.50 | 1.09 | 94% |

### Step 5: DVFS transition time

For the largest voltage step (OP1 to OP5): delta_V = 0.85 - 0.50 = 0.35 V

Assuming the VRM can slew at 10 mV/us (typical for a multiphase buck converter with SVID control):

```
t_voltage = delta_V / slew_rate = 350 mV / 10 mV/us = 35 us
```

Add settling time (~5 us) and frequency adjustment time (~2 us):

```
t_total_transition = 35 + 5 + 2 = 42 us
```

With an on-die IVR (slew rate ~100 mV/us):

```
t_voltage_IVR = 350 / 100 = 3.5 us
t_total_IVR = 3.5 + 1 + 1 = 5.5 us
```

### Summary

DVFS provides up to 94% power reduction per core at the lowest operating point. The IVR enables 7.6x faster voltage transitions compared to an off-chip VRM, which is critical for workloads that frequently switch between operating points.
