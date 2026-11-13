# 8_BIT_CARRY_SELECT_ADDER
Design and implementation of an 8-bit Carry Select Adder (CSLA) for high-speed binary addition. The architecture reduces carry propagation delay by computing multiple carry possibilities in parallel and selecting the correct output using multiplexers.
# 8-Bit Carry Select Adder (CSLA)

## Project Overview
This project focuses on the design and implementation of an 8-Bit Carry Select Adder (CSLA) on Cadence virtuoso, a high-speed arithmetic circuit used for binary addition in digital VLSI systems.

The Carry Select Adder improves addition speed by reducing carry propagation delay. It divides the adder into smaller blocks and computes the sum and carry outputs for both possible carry-in values (0 and 1) in parallel. A multiplexer then selects the correct result based on the actual incoming carry.

## Key Features
- 8-bit binary addition.
- Reduced carry propagation delay.
- Parallel computation of carry-in = 0 and carry-in = 1.
- Multiplexer-based carry selection.
- Improved speed compared to a conventional Ripple Carry Adder.
- Suitable for high-speed arithmetic applications.

## Working Principle
The 8-bit adder is divided into smaller sections(4_bit hybrid adders). The first section generates the actual carry, while the subsequent sections calculate two possible results in parallel. Multiplexers select the appropriate sum and carry depending on the previous block's carry output.

## Applications
- Arithmetic Logic Units (ALUs).
- Digital Signal Processors (DSPs).
- Microprocessors.
- High-speed digital systems.
- VLSI arithmetic circuits.

## Objective
To design an efficient 8-bit adder architecture that reduces addition delay using the Carry Select technique.
