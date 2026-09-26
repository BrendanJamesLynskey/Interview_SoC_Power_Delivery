# Quiz: Board and Simulation

Test your knowledge of PCB power delivery, simulation methods, and power noise analysis.

---

**Q1.** Via stitching in a PCB primarily serves to:

A) Increase signal routing density  
B) Connect ground planes on different layers with low impedance  
C) Reduce the board thickness  
D) Improve copper adhesion  

**Q2.** The switching frequency of a multiphase VRM is typically:

A) 10 kHz to 50 kHz  
B) 100 kHz to 2 MHz per phase  
C) 10 MHz to 100 MHz  
D) 1 GHz to 5 GHz  

**Q3.** When a signal trace crosses a power plane split, the primary concern is:

A) Increased trace resistance  
B) Disrupted return current path  
C) Reduced signal amplitude  
D) Improved impedance matching  

**Q4.** In frequency-domain PDN analysis, the impedance profile should be:

A) Above the target impedance at all frequencies  
B) Below the target impedance at all frequencies  
C) Equal to the target impedance at the resonant frequency  
D) Independent of frequency  

**Q5.** The VRM control loop bandwidth for a 1 MHz switching frequency buck converter is typically:

A) 1 MHz  
B) 100 kHz to 200 kHz  
C) 10 kHz  
D) 50 MHz  

**Q6.** Supply-induced jitter in a PLL is most sensitive to noise at frequencies:

A) Well below the PLL bandwidth  
B) At or above the PLL bandwidth  
C) Only at DC  
D) Only at the clock frequency  

**Q7.** For a 50 A load step with 10 ns risetime, the effective frequency content extends to approximately:

A) 1 MHz  
B) 31.8 MHz  
C) 318 MHz  
D) 3.18 GHz  

**Q8.** The three droop components during a load transient are (in order):

A) Capacitive, inductive, resistive  
B) Resistive (ESR), inductive (Ldi/dt), capacitive (VRM response)  
C) VRM response, capacitive, resistive  
D) Inductive, capacitive, resistive  

**Q9.** Vectorless dynamic IR drop analysis differs from vectored analysis in that it:

A) Is always more accurate  
B) Uses statistical activity estimation instead of simulation-derived switching data  
C) Does not account for on-die decoupling  
D) Only works at DC  

**Q10.** The PSRR of an LDO is typically highest at:

A) Very high frequencies (above 1 GHz)  
B) Low frequencies within the LDO loop bandwidth  
C) The switching frequency of the upstream converter  
D) The resonant frequency of the output capacitor  

**Q11.** Cavity resonance in PCB power planes occurs at frequencies where:

A) The plane dimensions are comparable to the electromagnetic wavelength  
B) The copper thickness resonates with the dielectric constant  
C) The decoupling capacitors are at their ESR minimum  
D) The VRM output voltage oscillates  

**Q12.** Transfer impedance Z21 measures:

A) The impedance of a single capacitor  
B) Voltage noise at one location caused by current injected at another location  
C) The total PDN resistance  
D) The reflection coefficient at a port  

**Q13.** The typical mounting inductance added by a PCB via connection to an MLCC is:

A) 0.01 to 0.05 pH  
B) 0.3 to 1.5 nH  
C) 10 to 50 nH  
D) 1 to 5 uH  

**Q14.** Which measurement technique provides the best accuracy for sub-milliohm PDN impedance?

A) One-port VNA reflection  
B) Two-port VNA shunt-through  
C) Time-domain reflectometry  
D) Multimeter resistance measurement  

**Q15.** A 2 GHz PLL has a 5 MHz loop bandwidth and a VCO supply sensitivity of 10%/V (fractional frequency change per volt of supply). A 5 mV sinusoidal supply ripple at 10 MHz produces approximately how much peak jitter?

A) 0.8 ps  
B) 8 ps  
C) 80 ps  
D) 0.08 ps  

**Q16.** The copper sheet resistance of a 2 oz (70 um) PCB layer is approximately:

A) 0.025 mOhm/sq  
B) 0.245 mOhm/sq  
C) 2.45 mOhm/sq  
D) 24.5 mOhm/sq  

---

## Answer Key

| Question | Answer | Explanation |
|----------|--------|-------------|
| Q1 | B | Stitching vias provide low-impedance connections between ground planes |
| Q2 | B | Per-phase switching is typically 100 kHz to 2 MHz |
| Q3 | B | The return current must detour around the split, increasing loop area |
| Q4 | B | The impedance must stay below the target at all frequencies |
| Q5 | B | Bandwidth is typically 1/5 to 1/10 of fsw |
| Q6 | B | The PLL acts as a high-pass filter; noise near/above bandwidth passes through |
| Q7 | B | f_knee = 1/(pi*t_rise) = 1/(pi*10ns) = 31.8 MHz |
| Q8 | B | ESR droop first, then Ldi/dt, then capacitive discharge during VRM response |
| Q9 | B | Vectorless uses statistical toggle rates rather than actual simulation vectors |
| Q10 | B | Within the loop bandwidth, the feedback loop actively rejects input noise |
| Q11 | A | Standing wave resonance when plane size ~ wavelength |
| Q12 | B | Transfer impedance is the cross-coupling between two PDN ports |
| Q13 | B | Via inductance of 0.3-1.5 nH is typical for standard PCB mounting |
| Q14 | B | Two-port shunt-through provides the best sensitivity at low impedance |
| Q15 | B | 10 MHz is above the 5 MHz loop bandwidth, so the loop does not correct it. delta_f = 0.1/V * 5 mV * 2 GHz = 1 MHz peak; phase deviation = delta_f / f_noise = 0.1 rad; J = 0.1 / (2*pi*2 GHz) = 8.0 ps peak |
| Q16 | B | Rsh = rho/T = 1.72e-6/(70e-4) = 0.245 mOhm/sq |
