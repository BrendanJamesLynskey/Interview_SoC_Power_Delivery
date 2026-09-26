# Worked Problem 01: PCB PDN Stackup

## Problem Statement

Design a PCB stackup for an SoC board with the following requirements:
- Three power rails: VCORE (0.85V, 50A), VGPU (0.80V, 20A), VIO (1.8V, 3A)
- High-speed signal routing: DDR5, PCIe Gen5
- Board thickness: 2.0 mm (+/- 10%)
- Material: high-Tg FR4 (Dk=4.2, Df=0.02)
- Minimum 4 signal routing layers

Propose a 12-layer stackup with appropriate plane allocation.

---

## Worked Solution

### Step 1: Determine plane requirements

Each power rail needs a dedicated power plane adjacent to a ground plane:
- VCORE plane + GND plane (closest pair, thinnest dielectric for max plane cap)
- VGPU plane + GND plane
- VIO can share a plane with VGPU (split) or use a separate plane

Signal routing needs at least 4 layers, each with an adjacent ground reference.

### Step 2: Propose the 12-layer stackup

| Layer | Function | Copper (oz) | Dielectric Below (mil) |
|-------|----------|------------|----------------------|
| L1 | Signal (top components) | 1 | 3.5 (prepreg) |
| L2 | GND | 1 | 3.0 (core) |
| L3 | Signal | 0.5 | 3.5 (prepreg) |
| L4 | VCORE | 2 | 3.0 (core) |
| L5 | GND | 1 | 5.0 (prepreg) |
| L6 | Signal | 0.5 | 3.0 (core) |
| L7 | Signal | 0.5 | 3.0 (core) |
| L8 | GND | 1 | 5.0 (prepreg) |
| L9 | VGPU / VIO (split) | 2 | 3.0 (core) |
| L10 | Signal | 0.5 | 3.5 (prepreg) |
| L11 | GND | 1 | 3.0 (core) |
| L12 | Signal (bottom components) | 1 | -- |

### Step 3: Verify total thickness

Sum all dielectric layers (approximating prepreg and core):
- Copper layers: 12 layers, average ~0.7 oz = ~24 um each, total ~288 um = 0.29 mm
- Dielectric: 3.5+3+3.5+3+5+3+3+5+3+3.5+3 = 39 mil = 0.99 mm

Total: ~0.99 + 0.29 = 1.28 mm. This is thin. Adjust dielectric thicknesses to reach 2.0 mm:

Revised: increase core thicknesses to 5-8 mil:

| Dielectric | Thickness (mil) |
|-----------|----------------|
| L1-L2 prepreg | 4.0 |
| L2-L3 core | 5.0 |
| L3-L4 prepreg | 4.0 |
| L4-L5 core | 3.0 (thin for VCORE-GND pair) |
| L5-L6 prepreg | 8.0 |
| L6-L7 core | 8.0 |
| L7-L8 prepreg | 8.0 |
| L8-L9 core | 3.0 (thin for VGPU-GND pair) |
| L9-L10 prepreg | 4.0 |
| L10-L11 core | 5.0 |
| L11-L12 prepreg | 4.0 |

Total dielectric: 4+5+4+3+8+8+8+3+4+5+4 = 56 mil = 1.42 mm
Copper: ~0.35 mm (with 2oz layers being thicker)
Total: ~1.77 mm. Close to 2.0 mm. Fine-tune core thicknesses to hit 2.0 mm.

### Step 4: Calculate plane capacitance for VCORE

VCORE plane (L4) paired with GND (L5), separated by 3.0 mil = 76 um.

Assuming the plane overlap area is 30 mm x 30 mm (package area + surrounding region):

```
C = epsilon_0 * epsilon_r * A / d
C = 8.854e-12 * 4.2 * 900e-6 / 76e-6
C = 8.854e-12 * 4.2 * 11.84
C = 441 pF
```

This ~440 pF is a small but useful contribution to decoupling at GHz frequencies.

### Step 5: Verify signal integrity

- L1 referenced to L2 (GND): controlled impedance stripline/microstrip
- L3 referenced to L2 (GND) and L4 (VCORE): asymmetric stripline
- L6 referenced to L5 (GND) and L7 (Signal -- not ideal, but L8 GND is nearby)
- L10 referenced to L9 (VGPU) and L11 (GND): asymmetric stripline

Issue: L6 and L7 signal layers reference each other, which is not ideal. Signal layers should always reference a ground plane. Revised approach: swap L7 to a GND plane and add the 4th signal layer elsewhere.

### Step 6: Revised stackup

| Layer | Function | Notes |
|-------|----------|-------|
| L1 | Signal | Top, reference L2 |
| L2 | GND | |
| L3 | Signal | Reference L2, L4 |
| L4 | VCORE (2oz) | |
| L5 | GND | 3 mil to L4 (key pair) |
| L6 | Signal | Reference L5, L7 |
| L7 | GND | |
| L8 | Signal | Reference L7, L9 |
| L9 | GND | 3 mil to L10 (key pair) |
| L10 | VGPU/VIO (2oz) | |
| L11 | Signal | Reference L10, L12 |
| L12 | GND + Signal | Bottom |

This provides 5 signal layers (L1, L3, L6, L8, L11), 4 GND planes, and 2 power planes with thin dielectric to their adjacent GND planes.

### Summary

The optimized 12-layer stackup provides:
- Dedicated VCORE and VGPU power planes with 2oz copper
- 3-mil dielectric between power planes and adjacent ground planes for maximum plane capacitance and minimum inductance
- Every signal layer has an adjacent ground plane reference
- 5 signal routing layers for DDR5 and PCIe escape routing
