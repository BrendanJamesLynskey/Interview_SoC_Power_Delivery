# Worked Problem 03: PDN-Induced Jitter

## Problem Statement

A PLL generating a 2 GHz clock has the following characteristics:

- PLL bandwidth: 5 MHz
- VCO supply sensitivity: Kvdd = 8 %/V (at 0.85V, this is 8%*2GHz/V = 160 MHz/V)
- PLL PSRR at 5 MHz: 10 dB (3.16x attenuation)
- PLL PSRR at 50 MHz: 0 dB (no attenuation)
- PLL PSRR at 500 kHz: 30 dB (31.6x attenuation)

The PDN noise spectrum at the PLL supply shows:

| Frequency | Noise amplitude (mV peak) |
|-----------|--------------------------|
| 500 kHz | 15 |
| 5 MHz | 8 |
| 50 MHz | 5 |
| 200 MHz | 3 |

Calculate the supply-induced jitter contribution from each frequency component and the total RMS jitter.

---

## Worked Solution

### Step 1: Calculate noise at VCO after PSRR filtering

The PSRR attenuates the supply noise before it reaches the VCO:

| Frequency | Supply noise (mV) | PSRR (dB) | Attenuation | Noise at VCO (mV) |
|-----------|-------------------|-----------|-------------|-------------------|
| 500 kHz | 15 | 30 | 31.6x | 0.475 |
| 5 MHz | 8 | 10 | 3.16x | 2.53 |
| 50 MHz | 5 | 0 | 1x | 5.0 |
| 200 MHz | 3 | 0 | 1x | 3.0 |

(Note: PSRR above the PLL bandwidth is assumed to be 0 dB for simplicity. In reality, the VCO itself may have some inherent supply rejection at very high frequencies due to limited bandwidth, but we use 0 dB as worst case.)

### Step 2: Calculate frequency deviation from each component

The VCO frequency deviation is:

```
delta_f = Kvdd_Hz * V_noise_at_VCO
```

where Kvdd_Hz = 160 MHz/V = 160 kHz/mV.

| Frequency | Noise at VCO (mV) | delta_f (kHz) |
|-----------|--------------------|---------------|
| 500 kHz | 0.475 | 76.0 |
| 5 MHz | 2.53 | 404.8 |
| 50 MHz | 5.0 | 800.0 |
| 200 MHz | 3.0 | 480.0 |

### Step 3: Convert frequency deviation to jitter

For a sinusoidal noise at frequency f_noise, the peak phase deviation is:

```
phi_peak = delta_f / f_noise (in radians, if delta_f and f_noise are in same units)
```

And the peak period jitter is:

```
J_peak = phi_peak / (2*pi*f_clock) = delta_f / (f_noise * 2*pi*f_clock)
```

Wait -- more precisely, if the VCO frequency modulates sinusoidally at frequency f_noise with peak deviation delta_f:

```
phi(t) = (delta_f / f_noise) * sin(2*pi*f_noise*t)
```

The peak phase deviation in radians:

```
phi_peak = delta_f / f_noise
```

The corresponding peak timing jitter (peak deviation of the clock edge):

```
J_peak = phi_peak / (2*pi*f_clock)
```

The RMS jitter from this component:

```
J_rms = J_peak / sqrt(2) = delta_f / (f_noise * 2*pi*f_clock * sqrt(2))
```

| f_noise | delta_f (Hz) | phi_peak (rad) | J_peak (ps) | J_rms (ps) |
|---------|-------------|----------------|-------------|------------|
| 500 kHz | 76.0e3 | 0.152 | 12.1 | 8.55 |
| 5 MHz | 404.8e3 | 0.0810 | 6.44 | 4.55 |
| 50 MHz | 800.0e3 | 0.0160 | 1.27 | 0.90 |
| 200 MHz | 480.0e3 | 0.00240 | 0.191 | 0.135 |

### Step 4: Compute total RMS jitter

Assuming the noise components are independent (uncorrelated):

```
J_total_rms = sqrt(J1^2 + J2^2 + J3^2 + J4^2)
J_total_rms = sqrt(8.55^2 + 4.55^2 + 0.90^2 + 0.135^2)
J_total_rms = sqrt(73.1 + 20.7 + 0.81 + 0.018)
J_total_rms = sqrt(94.6)
J_total_rms = 9.73 ps
```

### Step 5: Analyze the dominant contributors

| Component | J_rms (ps) | % of total variance |
|-----------|-----------|-------------------|
| 500 kHz | 8.55 | 77.3% |
| 5 MHz | 4.55 | 21.9% |
| 50 MHz | 0.90 | 0.86% |
| 200 MHz | 0.135 | 0.02% |

The 500 kHz component dominates because despite the PLL's 30 dB PSRR attenuation, the original noise amplitude is large (15 mV), and the low frequency means the phase deviation per Hz of frequency modulation is large.

### Key Insights

1. Low-frequency supply noise (near or below the PLL bandwidth) can contribute significant jitter even with PSRR attenuation because the phase integration effect is strongest at low frequencies.
2. The 5 MHz component (at the PLL bandwidth) is the second-largest contributor because the PSRR is only 10 dB at this frequency.
3. High-frequency noise (50 MHz, 200 MHz) contributes negligibly to jitter because the short modulation period means very little phase is accumulated per cycle.
4. To reduce the total jitter from 9.73 ps, the highest-priority action is to reduce the 500 kHz supply noise from 15 mV to a lower level (this is likely VRM switching ripple or bulk cap resonance).

### Summary

| Parameter | Value |
|-----------|-------|
| Total supply-induced jitter | 9.73 ps RMS |
| Dominant contributor | 500 kHz noise (8.55 ps, 77%) |
| Jitter budget for supply (typical) | 5-8 ps RMS |
| Status | FAIL (needs PDN improvement at 500 kHz) |
