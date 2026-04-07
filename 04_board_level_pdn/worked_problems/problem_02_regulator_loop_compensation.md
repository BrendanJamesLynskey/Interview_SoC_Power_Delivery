# Worked Problem 02: Regulator Loop Compensation

## Problem Statement

A single-phase current-mode buck converter has the following parameters:

- Input voltage: Vin = 5 V
- Output voltage: Vout = 0.85 V
- Switching frequency: fsw = 1 MHz
- Output inductor: L = 220 nH, DCR = 1.5 mOhm
- Output capacitor: Cout = 500 uF (ceramic + polymer), ESR = 0.5 mOhm
- Load current: 10 A
- Current sense gain: Ri = 10 mOhm (DCR sensing)
- Error amplifier transconductance: Gm_ea = 1 mA/V
- Reference voltage: Vref = 0.6 V
- Voltage divider ratio: Vout/Vfb = 0.85/0.6 = 1.417 (feedback divider gain = 0.706)

Design a Type II compensation network to achieve a crossover frequency of 100 kHz with at least 55 degrees of phase margin.

---

## Worked Solution

### Step 1: Identify the power stage transfer function

For a current-mode buck converter, the control-to-output transfer function (from compensation output to Vout) has a single pole at:

```
fp = 1 / (2 * pi * Rload * Cout)
```

where Rload = Vout / Iload = 0.85 / 10 = 0.085 ohm.

```
fp = 1 / (2 * pi * 0.085 * 500e-6) = 1 / (2.67e-4) = 3.74 kHz
```

There is also an ESR zero at:

```
fz_esr = 1 / (2 * pi * ESR * Cout) = 1 / (2 * pi * 0.5e-3 * 500e-6) = 637 kHz
```

The DC gain of the power stage (at the PWM comparator output) is approximately:

```
Gps_dc = Vout / (Ri * Iload) * Rload = ... 
```

More precisely, the modulator gain is:

```
Gmod = Vin / (Se * Ts)
```

where Se is the compensating slope. For simplicity, at current-mode operation:

```
Gps(s) = Gps_dc / (1 + s/(2*pi*fp)) * (1 + s/(2*pi*fz_esr))
```

The DC gain from the duty cycle command to Vout is approximately Rload/Ri:

```
Gps_dc = Rload / Ri = 0.085 / 0.010 = 8.5
```

(This is a simplified model; the actual gain includes the PWM modulator gain and varies with operating point.)

### Step 2: Design the Type II compensator

The Type II compensator has:
- A pole at origin (integrator) for DC accuracy
- One zero (fz_comp) to provide phase boost at crossover
- One high-frequency pole (fp_comp) to roll off noise

Place the compensator zero at or below the crossover frequency to provide phase boost:

```
fz_comp = fc / 3 = 100 kHz / 3 = 33 kHz
```

Place the high-frequency pole at half the switching frequency to attenuate noise:

```
fp_comp = fsw / 2 = 500 kHz
```

### Step 3: Calculate component values

For a Type II compensator using a transconductance amplifier (OTA):

```
C1 = Gm_ea / (2 * pi * fc * Gps_dc * Gfb)
```

where Gfb is the feedback divider gain = 0.706.

```
C1 = 1e-3 / (2 * pi * 100e3 * 8.5 * 0.706)
C1 = 1e-3 / (3.77e6)
C1 = 265 pF
```

For the compensator zero:

```
R2 = 1 / (2 * pi * fz_comp * C1) ... 
```

Actually, for an OTA-based Type II:
- C1 sets the integrator pole (pole at origin)
- R2 in series with C2 provides the zero
- C1 sets the high-frequency pole with R2

```
R2 = 1 / (2 * pi * fz_comp * C2)
```

And the high-frequency pole:

```
fp_comp = 1 / (2 * pi * R2 * C1)
```

From the high-frequency pole equation:

```
R2 = 1 / (2 * pi * fp_comp * C1) = 1 / (2 * pi * 500e3 * 265e-12) = 1.20 kOhm
```

From the zero:

```
C2 = 1 / (2 * pi * fz_comp * R2) = 1 / (2 * pi * 33e3 * 1200) = 4.02 nF
```

Select standard values: R2 = 1.2 kOhm, C1 = 270 pF, C2 = 3.9 nF.

### Step 4: Verify phase margin

At the crossover frequency (100 kHz):

Power stage phase: -90 degrees (from the single pole at 3.74 kHz, well below fc) + some recovery from the ESR zero at 637 kHz (small at 100 kHz).

Phase from power stage pole:
```
phase_ps = -arctan(fc/fp) = -arctan(100/3.74) = -arctan(26.7) = -87.9 degrees
```

Phase from ESR zero:
```
phase_esr = +arctan(fc/fz_esr) = +arctan(100/637) = +arctan(0.157) = +8.9 degrees
```

Total power stage phase at fc: -87.9 + 8.9 = -79.0 degrees.

Compensator phase at fc:
- Integrator: -90 degrees
- Zero at 33 kHz: +arctan(fc/fz) = +arctan(100/33) = +arctan(3.03) = +71.7 degrees
- HF pole at 500 kHz: -arctan(fc/fp) = -arctan(100/500) = -arctan(0.2) = -11.3 degrees

Total compensator phase: -90 + 71.7 - 11.3 = -29.6 degrees.

Feedback divider phase: 0 degrees (resistive).

Total loop phase at fc: -79.0 + (-29.6) = -108.6 degrees.

Phase margin = 180 - 108.6 = 71.4 degrees.

This exceeds the 55-degree target with comfortable margin.

### Summary

| Component | Value |
|-----------|-------|
| R2 | 1.2 kOhm |
| C1 | 270 pF |
| C2 | 3.9 nF |
| Crossover frequency | 100 kHz |
| Phase margin | ~71 degrees |
| Compensator zero | 34 kHz |
| High-frequency pole | 490 kHz |
