# Quiz: Advanced Topics

Test your knowledge of DVFS, integrated voltage regulators, and chiplet power delivery.

---

**Q1.** The primary power savings mechanism of DVFS is:

A) Reducing leakage only  
B) Reducing dynamic power through V^2*f scaling  
C) Eliminating IR drop  
D) Increasing clock frequency  

**Q2.** A low-to-high level shifter is needed when:

A) A signal moves from a higher voltage domain to a lower one  
B) A signal moves from a lower voltage domain to a higher one  
C) Both domains are at the same voltage  
D) The signal is a clock  

**Q3.** Retention flip-flops preserve state during power gating by using:

A) Battery backup  
B) An always-on shadow latch  
C) Non-volatile memory in each flip-flop  
D) The clock signal  

**Q4.** The efficiency of an on-die LDO converting 1.0 V to 0.6 V is:

A) 40%  
B) 60%  
C) 80%  
D) 100%  

**Q5.** A switched-capacitor converter with a 2:1 ratio ideally converts 1.0 V to:

A) 2.0 V  
B) 0.5 V  
C) 0.25 V  
D) 1.5 V  

**Q6.** Intel's FIVR technology uses inductors that are:

A) Discrete components on the motherboard  
B) Embedded in the package substrate  
C) Fabricated on the die using standard BEOL  
D) Not required (uses SC topology)  

**Q7.** In a 2.5D chiplet architecture, power TSVs pass through:

A) The active die only  
B) The silicon interposer  
C) The PCB  
D) The heat sink  

**Q8.** The keep-out zone around a TSV exists because:

A) TSVs generate electromagnetic radiation  
B) The TSV process creates mechanical stress that affects nearby transistors  
C) TSVs are too hot for nearby transistors  
D) TSVs are electrically connected to all layers  

**Q9.** Adaptive voltage scaling (AVS) differs from basic DVFS by:

A) Not using a voltage regulator  
B) Adjusting voltage based on actual silicon speed rather than a fixed table  
C) Operating only at one frequency  
D) Requiring a separate die for monitoring  

**Q10.** The rush current during power gating turn-on is caused by:

A) Short circuit between VDD and VSS  
B) Charging of all capacitance in the powered-off domain  
C) Electromagnetic interference  
D) Clock tree imbalance  

**Q11.** Backside power delivery (BSPD) delivers power through:

A) The front-side bumps as usual  
B) Wire bonds from the package lead frame  
C) TSVs from the wafer backside to buried power rails  
D) Optical waveguides  

**Q12.** In a 3D stacked architecture, the die furthest from the package substrate has:

A) The lowest temperature and lowest IR drop  
B) The highest temperature and highest cumulative IR drop  
C) The same temperature and IR drop as all other dies  
D) The best power delivery  

**Q13.** A hybrid SC+LDO IVR architecture is attractive because:

A) It requires no silicon area  
B) The SC provides efficient coarse conversion and the LDO provides fine regulation, both without inductors  
C) It has 100% efficiency  
D) It only works at one voltage  

**Q14.** The primary challenge of IVRs at 3 nm process nodes is:

A) The transistors are too fast  
B) Reduced analog performance and increased leakage on a digital-optimized process  
C) The die is too small for an IVR  
D) DVFS is not needed at 3 nm  

**Q15.** The UCIe standard for chiplet interconnects addresses:

A) Only signal interfaces  
B) Standardized chiplet interfaces including power delivery specifications  
C) Only thermal management  
D) Only test and debug  

**Q16.** For a DVFS voltage step from 0.55 V to 0.85 V, the correct sequencing is:

A) Increase frequency first, then raise voltage  
B) Raise voltage first, then increase frequency  
C) Change both simultaneously  
D) The order does not matter  

---

## Answer Key

| Question | Answer | Explanation |
|----------|--------|-------------|
| Q1 | B | P_dynamic = alpha*C*V^2*f; reducing V and f reduces power as V^2*f |
| Q2 | B | Low-to-high shifts signal from low voltage domain to high voltage domain |
| Q3 | B | A shadow latch on the always-on supply preserves data during power-off |
| Q4 | B | LDO efficiency = Vout/Vin = 0.6/1.0 = 60% |
| Q5 | B | 2:1 step-down produces Vin/2 = 0.5 V |
| Q6 | B | FIVR uses thin-film inductors embedded in the package substrate |
| Q7 | B | Power TSVs pass through the silicon interposer |
| Q8 | B | TSV fabrication creates stress that shifts Vth of nearby transistors |
| Q9 | B | AVS uses on-die speed monitors to adapt voltage to actual process corner |
| Q10 | B | Header switches connect discharged capacitance to VDD, causing rush current |
| Q11 | C | BSPD uses nano-TSVs from the wafer backside to reach buried power rails |
| Q12 | B | Top die has the most intervening resistance and is thermally insulated by layers below |
| Q13 | B | SC handles the voltage step-down efficiently, LDO regulates; no inductors needed |
| Q14 | B | Digital processes have poor analog characteristics; leakage is high |
| Q15 | B | UCIe standardizes the complete chiplet interface including power |
| Q16 | B | Voltage must reach target before frequency increases to avoid timing violations |
