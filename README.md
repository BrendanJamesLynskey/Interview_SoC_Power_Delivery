![SoC Power Delivery](https://img.shields.io/badge/topic-SoC%20power%20delivery-blue)

# Interview Preparation: SoC Power Delivery

A comprehensive study guide for SoC power delivery network (PDN) design, covering everything from board-level voltage regulation through package and on-die power distribution. This repository is structured for engineers preparing for interviews in roles involving power integrity, PDN design, and SoC physical design.

---

## Table of Contents

### 01 -- Foundations
Core concepts underlying all power delivery network design.

- [Power Delivery Fundamentals](01_foundations/power_delivery_fundamentals.md) -- PDN hierarchy, power delivery challenges, and design goals
- [PDN Impedance and Target Impedance](01_foundations/pdn_impedance_and_target.md) -- Target impedance formula, frequency domains, and impedance budgeting
- [Voltage Regulation Overview](01_foundations/voltage_regulation_overview.md) -- VRM topologies, regulation accuracy, and transient response
- [Worked Problems](01_foundations/worked_problems/)
  - [Problem 01: Target Impedance Calculation](01_foundations/worked_problems/problem_01_target_impedance_calculation.md)
  - [Problem 02: Power Budget Analysis](01_foundations/worked_problems/problem_02_power_budget_analysis.md)
  - [Problem 03: VRM Selection](01_foundations/worked_problems/problem_03_vrm_selection.md)

### 02 -- On-Die Power Delivery
Power distribution within the silicon die itself.

- [On-Die Power Grid Design](02_on_die_power_delivery/on_die_power_grid_design.md) -- Mesh and stripe topologies, grid sizing, and voltage uniformity
- [Decoupling Capacitance](02_on_die_power_delivery/decoupling_capacitance.md) -- Intrinsic gate cap, MOM/MOS decaps, and thin-oxide reliability
- [IR Drop and Electromigration](02_on_die_power_delivery/ir_drop_and_em.md) -- Static/dynamic IR drop analysis and EM design rules
- [Worked Problems](02_on_die_power_delivery/worked_problems/)
  - [Problem 01: Power Grid Analysis](02_on_die_power_delivery/worked_problems/problem_01_power_grid_analysis.md)
  - [Problem 02: Decap Optimization](02_on_die_power_delivery/worked_problems/problem_02_decap_optimization.md)
  - [Problem 03: EM Reliability Check](02_on_die_power_delivery/worked_problems/problem_03_em_reliability_check.md)

### 03 -- Package-Level PDN
Power delivery through the IC package.

- [Package PDN Design](03_package_level_pdn/package_pdn_design.md) -- Package plane design, resonance, and modeling techniques
- [Package Decoupling](03_package_level_pdn/package_decoupling.md) -- MLCC selection, ESR/ESL considerations, and placement strategy
- [Bump and Via Allocation](03_package_level_pdn/bump_and_via_allocation.md) -- C4/microbump design, power bump ratios, and via resistance
- [Worked Problems](03_package_level_pdn/worked_problems/)
  - [Problem 01: Package PDN Modeling](03_package_level_pdn/worked_problems/problem_01_package_pdn_modeling.md)
  - [Problem 02: MLCC Placement Strategy](03_package_level_pdn/worked_problems/problem_02_mlcc_placement_strategy.md)
  - [Problem 03: Power Bump Optimization](03_package_level_pdn/worked_problems/problem_03_power_bump_optimization.md)

### 04 -- Board-Level PDN
PCB power plane design and voltage regulation.

- [PCB Power Planes](04_board_level_pdn/pcb_power_planes.md) -- Plane design, stackup, copper weight, and via stitching
- [Voltage Regulators](04_board_level_pdn/voltage_regulators.md) -- Buck converters, LDOs, multiphase regulators, and loop compensation
- [Bulk and Ceramic Decoupling](04_board_level_pdn/bulk_and_ceramic_decoupling.md) -- Tantalum, polymer, electrolytic, and MLCC selection
- [Worked Problems](04_board_level_pdn/worked_problems/)
  - [Problem 01: PCB PDN Stackup](04_board_level_pdn/worked_problems/problem_01_pcb_pdn_stackup.md)
  - [Problem 02: Regulator Loop Compensation](04_board_level_pdn/worked_problems/problem_02_regulator_loop_compensation.md)
  - [Problem 03: Decoupling Network Design](04_board_level_pdn/worked_problems/problem_03_decoupling_network_design.md)

### 05 -- Analysis and Simulation
Techniques for verifying PDN performance.

- [Frequency Domain Analysis](05_analysis_and_simulation/frequency_domain_analysis.md) -- Impedance profiles, anti-resonance, and cavity resonance
- [Time Domain Simulation](05_analysis_and_simulation/time_domain_simulation.md) -- Step load response, droop, overshoot, and settling time
- [Power Noise and Jitter](05_analysis_and_simulation/power_noise_and_jitter.md) -- Supply-induced jitter, PDN noise coupling to PLLs
- [Worked Problems](05_analysis_and_simulation/worked_problems/)
  - [Problem 01: Impedance Profile Analysis](05_analysis_and_simulation/worked_problems/problem_01_impedance_profile_analysis.md)
  - [Problem 02: Transient Simulation](05_analysis_and_simulation/worked_problems/problem_02_transient_simulation.md)
  - [Problem 03: PDN-Induced Jitter](05_analysis_and_simulation/worked_problems/problem_03_pdn_induced_jitter.md)

### 06 -- Advanced Topics
Emerging and advanced power delivery techniques.

- [Adaptive Voltage Scaling](06_advanced_topics/adaptive_voltage_scaling.md) -- DVFS, voltage domains, level shifters, and retention
- [Integrated Voltage Regulators](06_advanced_topics/integrated_voltage_regulators.md) -- On-die LDOs, switched-cap converters, and inductor-based IVRs
- [Chiplet Power Delivery](06_advanced_topics/chiplet_power_delivery.md) -- Per-chiplet regulation, interposer PDN, and power TSVs
- [Worked Problems](06_advanced_topics/worked_problems/)
  - [Problem 01: DVFS Design](06_advanced_topics/worked_problems/problem_01_dvfs_design.md)
  - [Problem 02: IVR Tradeoffs](06_advanced_topics/worked_problems/problem_02_ivr_tradeoffs.md)
  - [Problem 03: Chiplet PDN Partitioning](06_advanced_topics/worked_problems/problem_03_chiplet_pdn_partitioning.md)

### 07 -- Quizzes
Multiple choice quizzes to test your knowledge.

- [Quiz: Foundations](07_quizzes/quiz_foundations.md)
- [Quiz: On-Die and Package](07_quizzes/quiz_on_die_and_package.md)
- [Quiz: Board and Simulation](07_quizzes/quiz_board_and_simulation.md)
- [Quiz: Advanced Topics](07_quizzes/quiz_advanced.md)

---

## How to Use

1. **Sequential study**: Work through sections 01 through 06 in order for a structured learning path.
2. **Targeted review**: Jump directly to a specific topic area if you need to brush up on a particular domain.
3. **Practice problems**: Each section includes worked problems that walk through realistic interview-style calculations step by step.
4. **Self-assessment**: Use the quizzes in section 07 to test your retention and identify weak areas.
5. **Interview simulation**: Pick random Q&A entries from different sections and practice explaining the answers aloud.

---

## Contributing

Contributions are welcome. Please open an issue or submit a pull request if you would like to add questions, fix errors, or expand coverage of a topic.

---

## Related Repositories

- [Interview: Signal Integrity](https://github.com/BrendanJamesLynskey/Interview_Signal_Integrity)
- [Interview: VLSI Design](https://github.com/BrendanJamesLynskey/Interview_VLSI_Design)

---

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

Last updated: 2026-04-07
