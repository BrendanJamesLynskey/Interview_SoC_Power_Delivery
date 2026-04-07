# Worked Problem 02: Transient Simulation

## Problem Statement

An SoC experiences a 35 A load step in 50 ns when a CPU core is ungated. The PDN has:

- VRM bandwidth: 150 kHz, output inductance per phase: 150 nH (4 phases)
- PCB output capacitance: 3000 uF (polymer + MLCC combined), ESR_total = 0.08 mOhm
- Package spreading inductance: 30 pH
- On-die decoupling: 300 nF, ESR = 0.5 mOhm
- Supply voltage: 0.85 V, target ripple: 5% (42.5 mV)

Calculate the expected voltage droop at the die.

---

## Worked Solution

### Step 1: First droop (resistive, on-die)

The initial droop is caused by the current step flowing through the on-die decoupling ESR and grid resistance:

```
V1 = I_step * ESR_die = 35 * 0.5e-3 = 17.5 mV
```

This occurs within the first few nanoseconds.

### Step 2: Second droop (capacitive, on-die + package)

After the ESR-limited initial response, the on-die capacitors supply the transient current. The voltage drops as the capacitors discharge:

```
dV/dt = I_step / C_die = 35 / 300e-9 = 1.167e8 V/s = 116.7 mV/ns
```

This is very fast. However, charge from the package capacitors begins arriving through the package spreading inductance. The time for the package charge to reach the die is limited by the L-C time constant:

```
t_LC = pi * sqrt(L_spread * C_die) = pi * sqrt(30e-12 * 300e-9) = pi * sqrt(9e-18) = pi * 3e-9 = 9.42 ns
```

During this time, the on-die caps supply the current alone. The capacitive droop during this period:

```
V2 = I_step * t_LC / C_die = 35 * 9.42e-9 / 300e-9 = 1.10 V
```

Wait -- this is larger than Vdd, which is clearly wrong. The issue is that 300 nF is far too small to supply 35 A for even 10 ns without massive droop. Let us reconsider.

Actually, the second droop calculation should account for the charge sharing between the on-die and package capacitance through the spreading inductance. A more accurate model:

The maximum voltage droop (second droop) when current transitions from on-die cap to package cap is limited by:

```
V2 = I_step * sqrt(L_spread / C_die)
```

This is the characteristic impedance of the L-C network:

```
V2 = 35 * sqrt(30e-12 / 300e-9) = 35 * sqrt(1e-4) = 35 * 0.01 = 0.35 V
```

This is still very large -- 350 mV on a 0.85 V rail, which is 41% of Vdd and clearly unacceptable.

This indicates a design problem: the on-die decoupling (300 nF) is far too small for a 35 A transient with 30 pH package inductance. The sqrt(L/C) impedance is 10 mOhm, which gives 350 mV for a 35 A step.

Let us recalculate with a more realistic on-die decoupling. If the design has 300 nF intentional + 400 nF intrinsic = 700 nF total:

```
V2 = 35 * sqrt(30e-12 / 700e-9) = 35 * sqrt(42.9e-6) = 35 * 6.55e-3 = 229 mV
```

Still very large. This shows that for 35 A transients, the on-die capacitance and package inductance are critical bottlenecks.

In reality, the risetime of the current step (50 ns) is much longer than the L-C resonance period (about 30 ns for 30 pH and 700 nF). When the risetime is longer than the resonance period, the effective droop is reduced because the package capacitors have time to respond during the current ramp. A more accurate estimate:

The inductive impedance of the package at the frequency corresponding to the risetime:

```
f_step = 1 / (pi * t_rise) = 1 / (pi * 50e-9) = 6.37 MHz
Z_L = 2 * pi * 6.37e6 * 30e-12 = 1.20 mOhm
```

At this frequency, the on-die capacitive impedance:

```
Z_C = 1 / (2 * pi * 6.37e6 * 700e-9) = 35.7 mOhm
```

The on-die capacitors are not effective at 6.37 MHz (too high impedance). The package capacitors (3000 uF PCB + package MLCCs) are effective:

```
Z_PCB = 1 / (2 * pi * 6.37e6 * 3000e-6) = 8.3 uOhm = 0.0083 mOhm
```

But they are behind 30 pH of package inductance (1.20 mOhm). The total impedance at 6.37 MHz:

```
Z_total ~ Z_L + Z_PCB = 1.20 + 0.008 = 1.21 mOhm (approximately just the inductance)
```

The voltage droop from the package inductance:

```
V_droop_pkg = L_spread * di/dt = 30e-12 * (35/50e-9) = 30e-12 * 7e8 = 21 mV
```

### Step 3: Third droop (VRM response)

The VRM responds after approximately:

```
t_vrm = 1 / (2*pi*150e3) = 1.06 us
```

During this 1.06 us, the PCB output capacitors supply the current. The capacitive droop:

```
V3 = I_step * t_vrm / C_PCB = 35 * 1.06e-6 / 3000e-6 = 12.4 mV
```

### Step 4: Total estimated droop

```
V_total = V1 + V_L_spread + V3
V_total = 17.5 + 21 + 12.4 = 50.9 mV
```

This exceeds the 42.5 mV budget by about 20%. The design needs improvement.

### Step 5: Recommendations to reduce droop

1. Increase on-die decoupling to reduce V1 (more decap cells) and improve high-frequency impedance
2. Reduce package spreading inductance (more power bumps, optimized package planes)
3. Increase VRM bandwidth to reduce t_vrm and thus V3
4. Implement rush current limiting on the clock ungating event to reduce the effective di/dt

### Summary

| Droop Component | Mechanism | Estimate |
|----------------|-----------|----------|
| First (ESR) | On-die grid + decap ESR | 17.5 mV |
| Second (Ldi/dt) | Package inductance | 21 mV |
| Third (capacitive) | PCB cap discharge during VRM response | 12.4 mV |
| **Total** | | **50.9 mV (FAIL)** |
| Budget | 5% of 0.85 V | 42.5 mV |
