# Worked Problem 03: VRM Selection

## Problem Statement

You are selecting a voltage regulator for the core supply rail of an SoC with these requirements:

- Input voltage: 5 V (from system PMIC)
- Output voltage: 0.85 V (DVFS range: 0.65 V to 1.0 V)
- Maximum load current: 40 A
- Load transient: 30 A step in 100 ns
- Maximum output voltage droop during transient: 40 mV
- Maximum steady-state ripple: 10 mV peak-to-peak
- Switching frequency: 2 MHz per phase
- Target efficiency: greater than 85% at full load

Determine (a) the number of phases needed, (b) the inductor value per phase, (c) the minimum output capacitance, and (d) the estimated efficiency.

---

## Worked Solution

### Step 1: Determine the number of phases

A practical per-phase current limit for discrete multiphase regulators is 10 to 15 A per phase with modern power stages. Choosing 10 A per phase for thermal margin:

```
N_phases = Imax / I_per_phase = 40 / 10 = 4 phases
```

With 4 phases, each phase carries 10 A at full load. The effective switching frequency at the output is 4 * 2 MHz = 8 MHz due to interleaving.

### Step 2: Select the inductor value per phase

The inductor value determines the current ripple in each phase. A common design guideline is to set the peak-to-peak inductor current ripple to 30-40% of the per-phase DC current. Choosing 30%:

```
delta_I = 0.30 * I_per_phase = 0.30 * 10 = 3 A peak-to-peak
```

The duty cycle at the nominal output voltage:

```
D = Vout / Vin = 0.85 / 5.0 = 0.17
```

The inductor value is determined from the voltage across the inductor during the on-time:

```
V_L_on = Vin - Vout = 5.0 - 0.85 = 4.15 V
delta_I = (V_L_on * D) / (L * f_sw)
L = (V_L_on * D) / (delta_I * f_sw)
L = (4.15 * 0.17) / (3 * 2e6)
L = 0.7055 / 6e6
L = 117.6 nH
```

Choose a standard inductor value: **120 nH** per phase. Verify the actual ripple:

```
delta_I_actual = (4.15 * 0.17) / (120e-9 * 2e6) = 0.7055 / 0.24 = 2.94 A
```

This is 29.4% of the 10 A per-phase current, which is acceptable.

### Step 3: Calculate minimum output capacitance

**For steady-state ripple:** With 4-phase interleaving at D = 0.17, the effective output ripple is significantly reduced compared to a single phase. For a 4-phase converter, the output ripple is approximately:

```
V_ripple = delta_I_phase / (8 * N * f_sw * C_out)  [simplified for interleaved operation]
```

However, a more accurate approach considers that at D = 0.17, the ripple cancellation is partial. The effective ripple current at the output for 4 phases at this duty cycle is approximately:

```
I_ripple_out = delta_I_phase * (1 - N*D) / (1 - D) [for D < 1/N]
```

Since N*D = 4 * 0.17 = 0.68 < 1:

```
I_ripple_out = 2.94 * (1 - 0.68) / (1 - 0.17) = 2.94 * 0.32 / 0.83 = 1.133 A
```

For 10 mV ripple:

```
C_out_ripple = I_ripple_out / (8 * N * f_sw * V_ripple)
C_out_ripple = 1.133 / (8 * 4 * 2e6 * 10e-3)
C_out_ripple = 1.133 / 640000
C_out_ripple = 1.77 uF
```

This is a very small capacitance requirement due to interleaving.

**For load transient droop (dominant requirement):**

During a 30 A load step, the inductor current in each phase ramps up at:

```
slew_rate = (Vin - Vout) / (L / N) = (5.0 - 0.85) / (120e-9 / 4) = 4.15 / 30e-9 = 138.3 A/us
```

Wait -- this is the total slew rate across all 4 phases. The time for the inductor current to ramp up by 30 A:

```
t_ramp = 30 A / 138.3 A/us = 0.217 us = 217 ns
```

During this ramp time, the output capacitor must supply the deficit current. The charge deficit is:

```
Q_deficit = 0.5 * I_step * t_ramp = 0.5 * 30 * 217e-9 = 3.255 uC
```

The capacitance needed to limit the droop to 40 mV:

```
C_out_transient = Q_deficit / V_droop = 3.255e-6 / 40e-3 = 81.4 uF
```

We must also account for the ESR contribution to droop. If we use MLCCs with ESR = 2 mOhm each and need the total ESR contribution to be less than, say, 20 mV (leaving 20 mV for the capacitive droop):

```
ESR_total = 20 mV / 30 A = 0.667 mOhm
```

The number of MLCCs needed for ESR alone: N_cap = 2 mOhm / 0.667 mOhm = 3. This is a very small number.

For the capacitive droop of 20 mV:

```
C_out = Q_deficit / 20e-3 = 3.255e-6 / 20e-3 = 162.8 uF
```

### Step 4: Select output capacitor configuration

Choose a combination of:

- **Bulk capacitors:** 2 x 100 uF polymer capacitors (ESR ~ 10 mOhm each). In parallel: 200 uF, ESR = 5 mOhm.
- **Ceramic capacitors:** 20 x 10 uF MLCCs (ESR ~ 2 mOhm, ESL ~ 0.5 nH each). In parallel: 200 uF additional, ESR = 0.1 mOhm, ESL = 25 pH.

Total capacitance: 400 uF (provides substantial margin over the 163 uF minimum).

### Step 5: Estimate efficiency at full load

**Conduction losses** (assuming Rdson = 3 mOhm per high-side FET, 1.5 mOhm per low-side FET):

```
P_cond_HS = N * D * (I_per_phase)^2 * Rdson_HS = 4 * 0.17 * 100 * 0.003 = 0.204 W
P_cond_LS = N * (1-D) * (I_per_phase)^2 * Rdson_LS = 4 * 0.83 * 100 * 0.0015 = 0.498 W
P_inductor = N * (I_per_phase)^2 * DCR = 4 * 100 * 0.002 = 0.8 W  [DCR = 2 mOhm]
```

Total conduction loss: 0.204 + 0.498 + 0.8 = 1.502 W

**Switching losses** (approximate, assuming 2 ns transitions and gate charge Qg = 20 nC):

```
P_sw = N * 0.5 * Vin * I_per_phase * (t_r + t_f) * f_sw
P_sw = 4 * 0.5 * 5.0 * 10 * 4e-9 * 2e6 = 0.8 W
P_gate = N * Qg * Vdrive * f_sw = 4 * 20e-9 * 5.0 * 2e6 = 0.8 W
```

Total switching loss: 0.8 + 0.8 = 1.6 W

**Total loss:** 1.502 + 1.6 = 3.102 W

**Output power:**

```
P_out = Vout * Iout = 0.85 * 40 = 34 W
```

**Efficiency:**

```
eta = P_out / (P_out + P_loss) = 34 / (34 + 3.102) = 34 / 37.102 = 91.6%
```

This exceeds the 85% target.

### Summary

| Parameter | Selected Value |
|-----------|---------------|
| Number of phases | 4 |
| Inductor per phase | 120 nH |
| Output capacitance | 400 uF (2x100 uF polymer + 20x10 uF MLCC) |
| Estimated efficiency | 91.6% at full load |
| Transient droop | < 40 mV for 30 A step |
| Steady-state ripple | < 10 mV pp |
