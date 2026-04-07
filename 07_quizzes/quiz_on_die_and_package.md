# Quiz: On-Die and Package

Test your knowledge of on-die power grid design, decoupling, electromigration, and package PDN.

---

**Q1.** In a mesh power grid topology, metal stripes run on:

A) One metal layer only  
B) Two orthogonal metal layers connected by vias  
C) Only the M1 layer  
D) Signal routing layers  

**Q2.** What is the primary advantage of MOM decaps over MOS decaps?

A) Higher capacitance density  
B) No gate oxide reliability (TDDB) concern  
C) Lower ESR  
D) Smaller area  

**Q3.** Black's equation for electromigration is MTTF = A * J^(-n) * exp(Ea/(kT)). What is the typical value of n for copper?

A) 0.5  
B) 1 to 2  
C) 5  
D) 10  

**Q4.** The Blech effect provides EM relief for:

A) All metal segments regardless of length  
B) Only very long metal segments  
C) Metal segments shorter than the Blech length  
D) Only vias  

**Q5.** A C4 bump at 150 um pitch with 15 mOhm resistance. If 300 VDD bumps carry 40 A total, the average current per bump is:

A) 13.3 mA  
B) 133 mA  
C) 1.33 A  
D) 40 mA  

**Q6.** What fraction of die area is typically dedicated to intentional decoupling capacitors in a high-performance SoC?

A) Less than 1%  
B) 5 to 15%  
C) 30 to 50%  
D) More than 50%  

**Q7.** Package spreading inductance typically falls in what range for a flip-chip BGA?

A) 1-10 nH  
B) 100-500 pH  
C) 10-100 pH  
D) 1-10 fH  

**Q8.** Which tool is used for on-die static and dynamic IR drop analysis?

A) Cadence Allegro  
B) ANSYS RedHawk  
C) Synopsys Formality  
D) Cadence Genus  

**Q9.** The effective resistance (Reff) of an on-die power grid is defined as:

A) The resistance of the thickest metal layer  
B) The sum of all metal resistances  
C) The average voltage drop divided by the total current  
D) The resistance of a single via  

**Q10.** DC bias derating of an MLCC refers to:

A) Increased capacitance with applied DC voltage  
B) Decreased capacitance with applied DC voltage  
C) Increased ESR with temperature  
D) Decreased inductance with frequency  

**Q11.** In a package substrate, which type of via has the lowest resistance?

A) Plated through-hole via (hollow)  
B) Filled copper via (solid)  
C) Laser-drilled microvia (blind)  
D) All vias have the same resistance  

**Q12.** Dynamic IR drop is typically how many times larger than static IR drop?

A) The same  
B) 2 to 3 times  
C) 10 times  
D) 100 times  

**Q13.** The intrinsic gate capacitance of a 5-billion-transistor SoC at 0.1 fF per transistor is approximately:

A) 5 nF  
B) 50 nF  
C) 500 nF  
D) 5 uF  

**Q14.** What determines the upper frequency limit of a package-mounted MLCC's effectiveness?

A) Its capacitance value  
B) Its ESL (body + mounting inductance)  
C) Its voltage rating  
D) Its physical size only  

**Q15.** Current crowding in C4 bumps is most severe at:

A) The center of the bump  
B) The interface between UBM and solder  
C) The bottom of the bump  
D) All locations equally  

**Q16.** How many metal layers does a typical 3 nm SoC have?

A) 4-6  
B) 6-9  
C) 12-16  
D) 20+  

---

## Answer Key

| Question | Answer | Explanation |
|----------|--------|-------------|
| Q1 | B | Mesh topology uses horizontal stripes on one layer and vertical on another |
| Q2 | B | MOM decaps use thick inter-metal dielectric, no thin-oxide reliability risk |
| Q3 | B | n is typically 1 (void growth) to 2 (void nucleation) for copper |
| Q4 | C | Short segments below the Blech length have stress gradient relief |
| Q5 | B | 40 A / 300 bumps = 133 mA per bump |
| Q6 | B | 5 to 15% of die area is typical for intentional decaps |
| Q7 | C | FC-BGA packages have 10-100 pH spreading inductance |
| Q8 | B | ANSYS RedHawk (and Cadence Voltus) are the primary tools |
| Q9 | C | Reff = V_drop_avg / I_total |
| Q10 | B | High-K dielectrics lose capacitance under DC electric field |
| Q11 | B | Filled copper vias have more conducting cross-section |
| Q12 | B | Dynamic IR drop includes Ldi/dt in addition to resistive drop |
| Q13 | C | 5e9 * 0.1e-15 = 500e-9 = 500 nF |
| Q14 | B | ESL determines the frequency where the cap becomes inductive |
| Q15 | B | Current transitions from thin UBM to bulk solder at the interface |
| Q16 | C | 3 nm processes typically have 12-16 metal layers |
