# Worked Problem 02: Power Budget Analysis

## Problem Statement

A quad-core application processor SoC has the following power specifications:

| Block | Voltage Rail | Average Current | Peak Current | Activity Factor |
|-------|-------------|-----------------|--------------|-----------------|
| CPU cluster (4 cores) | VCORE 0.85 V | 12 A | 30 A | 0.7 |
| GPU | VGPU 0.80 V | 8 A | 18 A | 0.5 |
| Memory controller | VCORE 0.85 V | 3 A | 5 A | 0.8 |
| I/O subsystem | VIO 1.8 V | 1.5 A | 2.5 A | 0.6 |
| PLL + analog | VPLL 0.85 V (LDO) | 0.1 A | 0.2 A | 1.0 |

1. Calculate the total average and peak power for each rail.
2. Determine the worst-case PDN power loss assuming 15 mOhm total DC resistance on the VCORE rail and 20 mOhm on VGPU.
3. Calculate the target impedance for each rail assuming 5% ripple budget.

---

## Worked Solution

### Step 1: Calculate power per block

Average power for each block = Voltage * Average Current:

| Block | Voltage (V) | Avg I (A) | Peak I (A) | Avg Power (W) | Peak Power (W) |
|-------|------------|-----------|-----------|---------------|----------------|
| CPU cluster | 0.85 | 12 | 30 | 10.20 | 25.50 |
| GPU | 0.80 | 8 | 18 | 6.40 | 14.40 |
| Memory controller | 0.85 | 3 | 5 | 2.55 | 4.25 |
| I/O subsystem | 1.8 | 1.5 | 2.5 | 2.70 | 4.50 |
| PLL + analog | 0.85 | 0.1 | 0.2 | 0.085 | 0.17 |

### Step 2: Aggregate power per rail

**VCORE rail (0.85 V):** CPU cluster + memory controller + PLL (PLL fed through LDO but draws from VCORE)

```
Avg current: 12 + 3 + 0.1 = 15.1 A
Peak current: 30 + 5 + 0.2 = 35.2 A  (if all peak simultaneously -- conservative)
Avg power: 10.20 + 2.55 + 0.085 = 12.835 W
Peak power: 25.50 + 4.25 + 0.17 = 29.92 W
```

**VGPU rail (0.80 V):**

```
Avg current: 8 A
Peak current: 18 A
Avg power: 6.40 W
Peak power: 14.40 W
```

**VIO rail (1.8 V):**

```
Avg current: 1.5 A
Peak current: 2.5 A
Avg power: 2.70 W
Peak power: 4.50 W
```

**Total SoC power:**

```
Avg total: 12.835 + 6.40 + 2.70 = 21.935 W
Peak total: 29.92 + 14.40 + 4.50 = 48.82 W
```

### Step 3: PDN resistive power loss (I-squared-R)

**VCORE rail** (R_dc = 15 mOhm):

```
P_loss_avg = I_avg^2 * R_dc = (15.1)^2 * 0.015 = 228.01 * 0.015 = 3.42 W
P_loss_peak = I_peak^2 * R_dc = (35.2)^2 * 0.015 = 1239.04 * 0.015 = 18.59 W
```

Note: 3.42 W average PDN loss on VCORE represents 3.42 / 12.835 = 26.6% of the delivered power. This is extremely high and indicates the 15 mOhm path resistance is too large. A redesign to reduce DC resistance (more bumps, wider metal, thicker copper planes) would be needed.

**VGPU rail** (R_dc = 20 mOhm):

```
P_loss_avg = (8)^2 * 0.020 = 64 * 0.020 = 1.28 W
P_loss_peak = (18)^2 * 0.020 = 324 * 0.020 = 6.48 W
```

The VGPU PDN loss is 1.28 / 6.40 = 20.0% of delivered power, also quite high.

### Step 4: Static IR drop check

**VCORE rail:**

```
IR_drop_avg = I_avg * R_dc = 15.1 * 0.015 = 226.5 mV
IR_drop_peak = I_peak * R_dc = 35.2 * 0.015 = 528 mV
```

A 226.5 mV average IR drop on a 0.85 V rail is 26.6% of Vdd -- far too high. The minimum voltage at the load would be 0.85 - 0.2265 = 0.6235 V under average conditions, which would likely cause timing failures. This confirms the path resistance must be reduced significantly.

Target: The 5% ripple budget allows only 0.85 * 0.05 = 42.5 mV of total noise. The DC IR drop alone should be well below this. A practical DC resistance target for VCORE would be on the order of:

```
R_dc_target = 42.5 mV / 35.2 A = 1.21 mOhm
```

### Step 5: Target impedance calculation per rail

**VCORE rail:**

```
Ztarget_VCORE = Vdd * ripple% / Imax
Ztarget_VCORE = 0.85 * 0.05 / 35.2
Ztarget_VCORE = 0.0425 / 35.2
Ztarget_VCORE = 1.21 mOhm
```

**VGPU rail:**

```
Ztarget_VGPU = 0.80 * 0.05 / 18
Ztarget_VGPU = 0.04 / 18
Ztarget_VGPU = 2.22 mOhm
```

**VIO rail:**

```
Ztarget_VIO = 1.8 * 0.05 / 2.5
Ztarget_VIO = 0.09 / 2.5
Ztarget_VIO = 36 mOhm
```

### Step 6: Summary

| Rail | Avg Power (W) | Peak Power (W) | Ztarget (mOhm) | PDN Loss @ Avg (W) |
|------|--------------|----------------|-----------------|---------------------|
| VCORE 0.85 V | 12.84 | 29.92 | 1.21 | 3.42 (too high) |
| VGPU 0.80 V | 6.40 | 14.40 | 2.22 | 1.28 (too high) |
| VIO 1.8 V | 2.70 | 4.50 | 36.0 | Negligible |
| **Total** | **21.94** | **48.82** | -- | -- |

### Key Observations

1. The VCORE and VGPU rails have sub-2 mOhm target impedances, requiring aggressive PDN design with many decoupling capacitors and low-resistance delivery paths.
2. The assumed 15 mOhm and 20 mOhm path resistances are far too high -- they must be reduced by 10x or more. This drives the need for many power bumps, wide on-die metal stripes, and thick PCB copper planes.
3. The VIO rail at 1.8 V has a much more relaxed 36 mOhm target because of the higher voltage and lower current.
4. Total SoC peak power of nearly 49 W requires careful thermal management in addition to electrical PDN design.
