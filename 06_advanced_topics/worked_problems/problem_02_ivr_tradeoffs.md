# Worked Problem 02: IVR Tradeoffs

## Problem Statement

Compare three IVR options for a compute chiplet requiring 10 A at output voltages from 0.55 V to 0.85 V, powered from a 1.0 V input:

1. On-die LDO
2. On-die 2:1 switched-capacitor + LDO
3. On-package inductor-based buck converter

Evaluate efficiency, area, and transient response for each.

---

## Worked Solution

### Option 1: On-die LDO

**Efficiency:** eta = Vout / Vin

| Vout | eta |
|------|-----|
| 0.85 V | 85% |
| 0.70 V | 70% |
| 0.55 V | 55% |

**Power dissipation at Vout = 0.55 V:**

```
P_loss = (Vin - Vout) * Iout = (1.0 - 0.55) * 10 = 4.5 W
```

**Area:** Pass transistor for 10 A with 150 mV max dropout needs Rdson < 15 mOhm. For a PMOS with Rdson*W ~ 50 Ohm*um, the total width needed: W = 50 / 0.015 = 3333 um = 3.3 mm. At 0.5 um per finger: ~6600 fingers. Total pass transistor area: approximately 0.3 to 0.5 mm^2. Error amplifier and reference: 0.01 mm^2. Total: ~0.5 mm^2.

**Transient response:** Bandwidth > 10 MHz. For a 5 A step in 10 ns: V_droop ~ 5 * ESR_pass ~ 5 * 15e-3 = 75 mV (before loop responds). With on-die output decoupling of 100 nF: V_cap_droop = 5 * 10e-9 / 100e-9 = 0.5 V — not negligible: 100 nF cannot carry a 5 A step for 10 ns, let alone the ~50 ns loop response time. The fast loop only helps if the on-die decoupling is sized for the step (e.g. ~1 uF per 50 mV for 10 ns).

### Option 2: 2:1 SC + LDO

**SC stage:** Converts 1.0 V to 0.5 V (2:1 ratio). At Vout_SC = 0.5 V: eta_SC ~ 95% (accounting for switch resistance). The LDO then regulates from 0.5 V to the desired output.

Wait -- the LDO cannot step UP from 0.5 V to 0.85 V. The SC output must be higher than the maximum Vout. Let us reconsider: use a 2:1 SC only when Vout < 0.5 V, and bypass the SC (direct 1.0 V to LDO) when Vout > 0.5 V. Or use a 3:2 SC (output = 0.67 V) followed by LDO.

Revised: Use a 3:2 SC converter (output = 2/3 * 1.0 = 0.667 V).

**For Vout = 0.55 V:** SC output = 0.667 V, LDO from 0.667 to 0.55 V.
- eta_SC = Vout_SC_ideal / Vin * (Vin / Vout_SC_actual) ~ 0.667/1.0 * correction ~ 90%
- eta_LDO = 0.55 / 0.667 = 82.5%
- eta_total = 0.90 * 0.825 = 74.3%

**For Vout = 0.85 V:** SC is bypassed (or uses 1:1 ratio), LDO from 1.0 to 0.85 V.
- eta_total = 85%

| Vout | Mode | eta_SC | eta_LDO | eta_total |
|------|------|--------|---------|-----------|
| 0.85 V | Bypass SC | 100% | 85% | 85% |
| 0.70 V | 3:2 SC | 90% | 105%? | Not possible |

The 3:2 SC produces 0.667 V, which is below 0.70 V. An LDO cannot produce 0.70 V from 0.667 V. We need a different SC ratio.

Revised again: Use an SC with selectable ratios: 1:1 (bypass, output = 1.0 V) and 2:3 (output = 0.667 V).

| Vout | SC Ratio | V_SC_out | eta_SC | eta_LDO | eta_total |
|------|----------|----------|--------|---------|-----------|
| 0.85 V | 1:1 | 1.0 V | 100% | 85% | 85% |
| 0.70 V | 1:1 | 1.0 V | 100% | 70% | 70% |
| 0.55 V | 2:3 | 0.667 V | ~90% | 82.5% | 74.3% |

**Area:** SC flying capacitors: For 10 A at 100 MHz switching: C_fly = I / (f * delta_V) = 10 / (100e6 * 0.05) = 2 uF. At 15 fF/um^2 (MOS cap): area = 2e-6 / 15e-15 = 1.33e8 um^2 = 133 mm^2 — larger than most dies. Switches + LDO: ~0.5 mm^2. Total: ~134 mm^2, so at 10 A this SC stage is only practical with much denser capacitors (e.g. deep-trench) or a higher switching frequency.

**Transient response:** Same as LDO alone (the LDO is the final regulation stage).

### Option 3: On-package buck converter

**Efficiency:** Typical 85-90% across the full voltage range.

| Vout | eta |
|------|-----|
| 0.85 V | 88% |
| 0.70 V | 86% |
| 0.55 V | 83% |

**Power dissipation at Vout = 0.55 V:**

```
P_loss = P_out * (1/eta - 1) = 5.5 * (1/0.83 - 1) = 5.5 * 0.205 = 1.13 W
```

**Area:** On-die switches: ~0.3 mm^2. Package inductors (2 phases, 5 nH each): ~1 mm^2 on package. Controller: ~0.1 mm^2 on die. Total die area: ~0.4 mm^2. Package area: ~1 mm^2.

**Transient response:** Bandwidth limited by switching frequency (~200 MHz/5 = 40 MHz). Load step response time: ~25 ns. Slower than LDO but still fast.

### Summary Comparison

| Parameter | LDO | SC + LDO | Buck |
|-----------|-----|----------|------|
| eta at 0.85 V | 85% | 85% | 88% |
| eta at 0.55 V | 55% | 74% | 83% |
| P_loss at 0.55 V | 4.5 W | 1.9 W | 1.13 W |
| Die area | 0.5 mm^2 | ~134 mm^2 (MOS flying caps) | 0.4 mm^2 |
| Package area | 0 | 0 | 1 mm^2 |
| Transient BW | >10 MHz | >10 MHz | ~40 MHz |
| Complexity | Low | Medium | High |

The buck converter is the most efficient but requires package inductors. The LDO is simplest but has poor efficiency at low Vout. The SC+LDO hybrid provides intermediate efficiency without inductors, but its flying capacitors need a much denser capacitor technology than MOS decap to fit on the die.
