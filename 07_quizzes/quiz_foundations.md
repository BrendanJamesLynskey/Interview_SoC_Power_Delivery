# Quiz: Foundations

Test your knowledge of PDN fundamentals, target impedance, and voltage regulation.

---

**Q1.** The target impedance formula is Ztarget = Vdd * ripple% / Imax. For Vdd = 0.9 V, 4% ripple, and Imax = 60 A, what is the target impedance?

A) 0.6 mOhm  
B) 1.5 mOhm  
C) 2.4 mOhm  
D) 6.0 mOhm  

**Q2.** Which PDN element is primarily responsible for maintaining low impedance in the 100 MHz to 1 GHz range?

A) VRM control loop  
B) PCB bulk capacitors  
C) Package MLCCs and on-die decoupling  
D) Board-level electrolytic capacitors  

**Q3.** What causes anti-resonance in a PDN?

A) Short circuit between VDD and VSS  
B) Parallel resonance between the inductance of one stage and the capacitance of another  
C) Excessive on-die decoupling  
D) VRM oscillation  

**Q4.** A multiphase buck converter with 6 phases at 1 MHz per phase has an effective output ripple frequency of:

A) 1 MHz  
B) 3 MHz  
C) 6 MHz  
D) 12 MHz  

**Q5.** What is the primary advantage of adaptive voltage positioning (AVP)?

A) It eliminates the need for decoupling capacitors  
B) It reduces the effective transient voltage window, allowing less output capacitance  
C) It increases the switching frequency  
D) It improves VRM efficiency at light load  

**Q6.** The series resonant frequency of a capacitor with C = 1 uF, ESL = 1 nH, and ESR = 2 mOhm is approximately:

A) 503 kHz  
B) 5.03 MHz  
C) 50.3 MHz  
D) 503 MHz  

**Q7.** Which of the following increases the VRM control loop bandwidth?

A) Increasing the output inductor value  
B) Increasing the output capacitor value  
C) Increasing the switching frequency  
D) Decreasing the reference voltage  

**Q8.** Power supply rejection ratio (PSRR) of an LDO is most important at:

A) DC only  
B) Frequencies below the LDO bandwidth  
C) The switching frequency of the upstream buck converter  
D) Frequencies above 1 GHz  

**Q9.** What happens to the target impedance as technology scales to smaller nodes?

A) It increases because transistors are faster  
B) It remains constant  
C) It decreases because supply voltage drops and current increases  
D) It depends only on the PCB design  

**Q10.** In a PDN hierarchy, which element provides charge at the highest frequencies (above 500 MHz)?

A) VRM output capacitors  
B) PCB ceramic MLCCs  
C) Package-mounted MLCCs  
D) On-die MOS/MOM decoupling capacitors  

**Q11.** The efficiency of a buck converter with Vin = 12 V and Vout = 0.85 V is primarily limited by:

A) Conduction losses in the output capacitor  
B) Switching losses due to very low duty cycle  
C) Dielectric losses in the inductor core  
D) Gate leakage of the pass transistor  

**Q12.** What is the typical di/dt for a 20 A current step with a 50 ns risetime?

A) 4 x 10^5 A/s  
B) 4 x 10^7 A/s  
C) 4 x 10^8 A/s  
D) 4 x 10^11 A/s  

**Q13.** The number of power domains in a modern mobile SoC is typically:

A) 1-3  
B) 5-10  
C) 10-30  
D) 100+  

**Q14.** Which type of capacitor has the lowest ESL?

A) Aluminum electrolytic  
B) Tantalum polymer  
C) PCB-mounted MLCC  
D) On-die MOS decap  

---

## Answer Key

| Question | Answer | Explanation |
|----------|--------|-------------|
| Q1 | A | Ztarget = 0.9 * 0.04 / 60 = 0.6 mOhm |
| Q2 | C | Package MLCCs cover ~50-300 MHz, on-die decaps cover above ~300 MHz |
| Q3 | B | Anti-resonance occurs when one stage is inductive and the adjacent is capacitive |
| Q4 | C | Effective ripple frequency = N * f_sw = 6 * 1 MHz = 6 MHz |
| Q5 | B | AVP uses the voltage window more efficiently, halving the transient excursion |
| Q6 | B | f_res = 1/(2*pi*sqrt(1e-9*1e-6)) = 5.03 MHz |
| Q7 | C | Higher switching frequency allows higher crossover frequency (bandwidth) |
| Q8 | C | PSRR at the switching frequency determines how much ripple reaches the load |
| Q9 | C | Lower Vdd and higher Imax both reduce Ztarget |
| Q10 | D | On-die decaps have the lowest ESL and provide charge at the highest frequencies |
| Q11 | B | At D = 7.1%, the high-side FET has very short on-time and high switching loss |
| Q12 | C | di/dt = 20A / 50ns = 4 x 10^8 A/s |
| Q13 | C | Modern mobile SoCs have 10-30 voltage domains for fine-grained power management |
| Q14 | D | On-die decaps have sub-pH ESL, far lower than any discrete capacitor |
