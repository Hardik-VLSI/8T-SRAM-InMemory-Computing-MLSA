8T SRAM-Based In-Memory Computing Cell with Multi-Logic Sense Amplifier (MLSA)

Design, simulation, and layout of an In-Memory Computing (IMC) architecture that performs Boolean logic operations (OR/NOR, AND/NAND, XOR/XNOR) directly within an 8-transistor SRAM array — implemented and verified in Cadence Virtuoso at 180nm CMOS.


Table of Contents


Overview
Problem Statement
Objectives
Background & Motivation
Design Methodology
Circuit Design
Layout Design
Results
Performance Analysis
Key Learnings
Challenges
Future Work
Tools Used
References



Overview

Conventional (von Neumann) computing architectures physically separate memory and processing units, causing significant latency and energy overhead from constant data movement between the two. This project implements an In-Memory Computing (IMC) cell that executes logic operations inside the memory array itself, eliminating this bottleneck for Boolean and arithmetic operations.

The design is built around two components:


An 8-transistor (8T) SRAM cell with electrically isolated read and write paths
A Multi-Logic Sense Amplifier (MLSA) capable of producing multiple logic outputs (OR/NOR, AND/NAND, XOR/XNOR) from a single sensing circuit, using a tunable reference voltage


Problem Statement

Data-intensive applications suffer from latency and power inefficiency caused by frequent data transfer between memory and compute units. This project addresses that bottleneck by designing an 8T SRAM-based IMC architecture that supports Boolean logic directly within memory, while managing the associated design challenges: preserving signal integrity during simultaneous multi-wordline access, optimizing the sensing scheme, and maintaining low-power operation.

Objectives


Design an 8T SRAM cell with reliable read/write functionality suited for low-power, in-memory logic
Develop a sensing mechanism (sense amplifier + reference tuning) that can interpret bitline discharge as logic outputs
Implement Boolean operations (AND, OR, XOR, NAND, NOR, XNOR) using bitline discharge behavior across multiple activated wordlines
Verify all designs (schematic + layout) in Cadence Virtuoso at the 180nm node, using Spectre for simulation
Characterize performance — power, delay, PDP — and benchmark against a conventional sense-amplifier baseline


Background & Motivation

Early IMC implementations used standard 6T SRAM cells, but their shared read/write path limits read stability and risks data corruption during simultaneous multi-cell access. 8T (and 8+T) SRAM topologies address this by adding a dedicated, isolated read port, enabling more robust in-memory logic execution — an approach demonstrated in prior work by Dong et al. and Rajput et al.

However, most prior designs required separate, dedicated sense amplifiers per logic function, increasing area, complexity, and power. This project builds on the Multi-Logic Sense Amplifier (MLSA) concept (Chang & Chen) — a single sensing circuit that produces multiple logic outputs (OR/NOR, AND/NAND, XOR/XNOR) by tuning the reference voltage, reducing hardware overhead while preserving functional flexibility.

Design Methodology

1. 8T SRAM Cell Architecture
An 8-transistor cell (6T storage core + 2T isolated read port) was chosen to decouple read and write paths, enabling multiple wordlines to be activated simultaneously without disturbing stored data — a prerequisite for bitline-based logic sensing.

Show Image

2. Bitline-Based Logic Operation
Word lines corresponding to different memory cells are enabled simultaneously. The resulting discharge behavior on the read bitline (RBL) — which varies depending on the logic values stored in each activated cell — is monitored and interpreted as a logic outcome.

3. Write-Back Operation
Computed logic results are routed back into the memory array via a controlled switching network, enabling a full compute-in-memory cycle without offloading data to external logic.

4. Multi-Logic Sense Amplifier (MLSA) Design
The MLSA uses a voltage-controlled latch with additional transistor-level logic to distinguish between OR, NOR, and XNOR states, based on precisely tuned reference voltages (Vref_High, Vref_Low) compared against the RBL voltage.

ABRBL VoltageOR (out1)NOR (~out1)XNOR (out2)00Vrbl > Vref01101VDD/2 < Vrbl < Vref10010VDD/2 < Vrbl < Vref10011Vrbl < VDD/2101

5. Simulation & Verification
All circuits were simulated using the Spectre simulator in Cadence Virtuoso at the 180nm CMOS node, including transient analysis across input combinations to extract timing, voltage response, and power behavior.

Circuit Design

BlockDescription8T SRAM CellStorage core (M1–M4) + isolated read port (M6–M8); sized to balance read stability and performanceMLSAVoltage-controlled latch + auxiliary logic for OR/NOR/AND/NAND/XOR/XNOR outputs, with a multiplexer for write-back routingIMC CellTwo 8T SRAM cells + one MLSA, forming a complete compute-in-memory unit, with a dedicated testbench for transient verification


Show Image
Show Image

Layout Design

Following schematic verification, the 8T SRAM cell, MLSA, and supporting logic were laid out in Cadence Virtuoso Layout Editor. All layouts were verified using Design Rule Check (DRC) and Layout Versus Schematic (LVS) to confirm physical correctness and functional equivalence with the schematic.

Show Image
Show Image
Show Image

Results

Read/Write Functionality
The 8T SRAM cell correctly performs read operations (RBL discharges based on stored data via the isolated read port) and write operations (data driven onto WBL and latched via the write wordline path).

Show Image

Bitline Discharge Characterization


Increasing the reference voltage accelerates RBL discharge; below the NMOS threshold voltage, discharge slows significantly, degrading sensing accuracy/speed.
A 12 ps sense-amplifier enable window was identified as optimal, giving clear voltage separation between intermediate (01/10) and matched (11) input states.


Show Image

Logic Operation Verification


Conventional latch-type sense amplifiers only natively support NOR/NAND; XOR/XNOR requires additional circuitry, increasing delay and power.
The MLSA performs OR, NOR, and XNOR from a single circuit, with XNOR achieving lower delay than the conventional sense-amp equivalent.


Show Image

Monte Carlo Analysis
Process-variation robustness was verified via Monte Carlo simulation (threshold voltage and carrier mobility varied randomly across 200 iterations), confirming stable performance across process corners.

Show Image

Performance Analysis

XNOR Operation

CasePower (µW)Delay (pS)PDP (fJ)0033.80170.905.7701/1052.19618.350.9511133.116.752.29

OR Operation

CasePower (µW)Delay (pS)PDP (fJ)0033.80107.653.6401/1052.196198.01110.3311133.196.212.80

Key Learnings


Isolating read/write paths in SRAM cells is essential for enabling reliable multi-wordline access without data corruption — a foundational requirement for any bitline-based IMC scheme
Reference-voltage tuning in sense amplifiers directly trades off sensing speed against reliability, and must be carefully characterized rather than assumed
Consolidating multiple logic functions into a single sense amplifier (MLSA) meaningfully reduces area and power compared to using dedicated sense amps per function


Challenges


Ensuring stable read-disturb margins while sizing transistors for the isolated read port
Tuning dual reference voltages (Vref_High, Vref_Low) precisely enough to reliably separate multiple logic states from a single sensing window
Balancing sense-amplifier enable timing against bitline discharge dynamics across varying stored-data combinations


Future Work


Extend the MLSA to support full arithmetic operations (addition, subtraction) beyond Boolean logic
Scale the design to a larger memory array to evaluate area/power trends at higher densities
Port the design to a more advanced CMOS node and re-characterize performance
Explore radiation/aging robustness for the sense amplifier under extended process-corner analysis


Tools Used


Cadence Virtuoso — schematic capture, layout design
Spectre Simulator — transient, DC, and Monte Carlo simulation
Technology Node: 180nm CMOS
Verification: DRC, LVS


References


Pateliya, R. (2019). Static Noise Margin of 6T and 8T SRAM Cell in 28-nm CMOS. ICACM.
Wang, J., et al. (2019). A 28-nm compute SRAM with bit-serial logic/arithmetic operations for programmable in-memory vector computing. IEEE JSSC, 55(1), 76–86.
Thakur, M., & Mehra, R. An Energy-Efficient Sense Amplifier Using 180nm for SRAM.
Chen, J., et al. (2021). Analysis and design of reconfigurable sense amplifier for compute SRAM with high-speed compute and normal read access. IEEE TCAS-II, 68(12), 3503–3507.
Kumar, S. S., & Nayak, J. (2023). Efficient Implementation of Fast Half Adder using 8T SRAM for In-Memory Computing Applications. MoSICom, 648–652.
Rajput, A. K., & Pattanaik, M. (2020). Implementation of Boolean and arithmetic functions with 8T SRAM cell for in-memory computation. INCET, 1–5.
Chang, Y. J., & Chen, S. H. (2023). Multi-Logic Sense Amplifier (MLSA) Design for In-Memory Computing. IEEE JETCAS, 13(1), 371–381.
