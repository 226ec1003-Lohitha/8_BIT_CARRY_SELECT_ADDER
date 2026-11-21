# 8_BIT_CARRY_SELECT_ADDER
Design and implementation of an 8-bit Carry Select Adder (CSLA) for high-speed binary addition. The architecture reduces carry propagation delay by computing multiple carry possibilities in parallel and selecting the correct output using multiplexers.
# 8-Bit Carry Select Adder (CSLA)
## Table of Contents

1. [Overview](#1-overview)
2. [Architecture and Design Hierarchy](#2-architecture-and-design-hierarchy)
3. [Truth Tables](#3-truth-tables)
   - 3.1 [2:1 Multiplexer](#31-21-multiplexer)
   - 3.2 [1-Bit Hybrid Full Adder](#32-1-bit-hybrid-full-adder)
   - 3.3 [Carry Select Operation](#33-carry-select-operation)
   - 3.4 [Example 8-Bit Test Vectors](#34-example-8-bit-test-vectors)
4. [Schematic Design](#4-schematic-design)
   - 4.1 [2:1 Multiplexer](#41-21-multiplexer)
   - 4.2 [4-Bit 2:1 Multiplexer](#42-4-bit-21-multiplexer)
   - 4.3 [1-Bit Hybrid Adder](#43-1-bit-hybrid-adder)
   - 4.4 [4-Bit Hybrid Adder](#44-4-bit-hybrid-adder)
   - 4.5 [8-Bit CSLA Top Level](#45-8-bit-csla-top-level)
   - 4.6 [Test Bench](#46-test-bench)
5. [Layout Design](#5-layout-design)
6. [Physical Verification](#6-physical-verification)
7. [Simulation Results](#7-simulation-results)
   - 7.1 [Summary of Key Specifications](#71-summary-of-key-specifications)
   - 7.2 [Timing Performance Breakdown](#72-timing-performance-breakdown)
   - 7.3 [Power, PDP and EDP Analysis](#73-power-pdp-and-edp-analysis)
   - 7.4 [Synchronous Waveform Timings](#74-synchronous-waveform-timings)
   - 7.5 [Transient Output Waveforms](#75-transient-output-waveforms)
8. [Observations and Future Work](#8-observations-and-future-work)
9. [Design and Simulation Notes](#9-design-and-simulation-notes)
10. [Project Report](#10-project-report)
11. [Repository Contents](#11-repository-contents)
12. [Tools](#12-tools)


## 1. Overview

| Item | Description |
|---|---|
| Design | 8-bit carry select adder (CSLA) |
| Full adder style | Hybrid: pass-transistor XOR/XNOR with transmission-gate carry and sum |
| Multiplexer style | Transmission-gate 2:1 mux with a select inverter |
| Library / project | `4_BIT_dac` |
| Device sizing | All transistors W = 120 nm, L = 45 nm (`pmos1v` / `nmos1v`, svt) |
| Inputs | A[7:0], B[7:0], C<sub>in</sub> |
| Outputs | S[7:0], C<sub>out</sub> |
| Verification | DRC clean, LVS clean |
| Extraction | Not performed (schematic-level results) |

---

## 2. Architecture and Design Hierarchy

The adder uses a 4 + 4 carry select structure:

- **Block 1 (bits 3:0):** one 4-bit hybrid adder driven by C<sub>in</sub>.
- **Block 2 (bits 7:4):** two 4-bit hybrid adders computed in parallel, one assuming carry-in = 0 and one assuming carry-in = 1.
- **Selection:** the carry C4 from block 1 drives a 4-bit 2:1 mux (upper sum bits) and a 1-bit 2:1 mux (C<sub>out</sub>).

```
S[7:4] = C4 ? Sum1[3:0] : Sum0[3:0]
Cout   = C4 ? Cout1     : Cout0
```

**Hierarchy**

```
complete_cls (8-bit CSLA top)
├── hybrid_addr (4-bit)          x 3    -> HYBRID_ADDER (1-bit) x 4 each
├── 2x1_4bit (4-bit 2:1 mux)     x 1    -> 2_1_mux x 4
└── 2_1_mux (1-bit, for Cout)    x 1
```

**Transistor count**

| Cell | Transistors per cell | Instances | Total |
|---|---|---|---|
| HYBRID_ADDER (1-bit) | 22 | 12 | 264 |
| 2_1_mux (1-bit) | 6 | 5 | 30 |
| **8-bit CSLA** | | | **≈ 294** |

---

## 3. Truth Tables

### 3.1 2:1 Multiplexer

Cell `2_1_mux`. Output equation: `Y = S'·A + S·B`

| S | A | B | Y |
|:-:|:-:|:-:|:-:|
| 0 | 0 | 0 | 0 |
| 0 | 0 | 1 | 0 |
| 0 | 1 | 0 | 1 |
| 0 | 1 | 1 | 1 |
| 1 | 0 | 0 | 0 |
| 1 | 0 | 1 | 1 |
| 1 | 1 | 0 | 0 |
| 1 | 1 | 1 | 1 |

Compact form:

| S | Y |
|:-:|:-:|
| 0 | A |
| 1 | B |

The 4-bit mux (`2x1_4bit`) applies this same function to each bit of A<3:0> and B<3:0> with one shared select.

### 3.2 1-Bit Hybrid Full Adder

Cell `HYBRID_ADDER`. Equations:

```
AxorB = A XOR B
Sum   = AxorB XOR Cin
Carry = AxorB ? Cin : A        (equivalent to A·B + Cin·(A XOR B))
```

| A | B | C<sub>in</sub> | A⊕B (AxorB) | Sum | Carry |
|:-:|:-:|:-:|:-:|:-:|:-:|
| 0 | 0 | 0 | 0 | 0 | 0 |
| 0 | 0 | 1 | 0 | 1 | 0 |
| 0 | 1 | 0 | 1 | 1 | 0 |
| 0 | 1 | 1 | 1 | 0 | 1 |
| 1 | 0 | 0 | 1 | 1 | 0 |
| 1 | 0 | 1 | 1 | 0 | 1 |
| 1 | 1 | 0 | 0 | 0 | 1 |
| 1 | 1 | 1 | 0 | 1 | 1 |

### 3.3 Carry Select Operation

C4 is the carry out of the lower 4-bit block. It selects which precomputed upper result is passed to the output.

| C4 | Upper sum S[7:4] | C<sub>out</sub> | Selected adder |
|:-:|:-:|:-:|---|
| 0 | Sum0[3:0] | Cout0 | Upper adder with carry-in = 0 |
| 1 | Sum1[3:0] | Cout1 | Upper adder with carry-in = 1 |

### 3.4 Example 8-Bit Test Vectors

| A (hex) | B (hex) | C<sub>in</sub> | C4 | S (hex) | C<sub>out</sub> | Decimal check |
|:-:|:-:|:-:|:-:|:-:|:-:|---|
| 00 | 00 | 0 | 0 | 00 | 0 | 0 + 0 = 0 |
| 0F | 01 | 0 | 1 | 10 | 0 | 15 + 1 = 16 |
| AA | 55 | 0 | 0 | FF | 0 | 170 + 85 = 255 |
| AA | 55 | 1 | 1 | 00 | 1 | 170 + 85 + 1 = 256 |
| 7F | 81 | 0 | 1 | 00 | 1 | 127 + 129 = 256 |
| FF | 01 | 0 | 1 | 00 | 1 | 255 + 1 = 256 |
| FF | FF | 1 | 1 | FF | 1 | 255 + 255 + 1 = 511 |

---

## 4. Schematic Design

### 4.1 2:1 Multiplexer

Cell `2_1_mux`: one inverter generates the complement of the select S, and two transmission gates pass A or B to Y. Six transistors in total, all W = 120 nm, L = 45 nm.

![2:1 mux schematic](21_MUX.png)

### 4.2 4-Bit 2:1 Multiplexer

Cell `2x1_4bit`: four 2:1 muxes sharing a single select line. Bus inputs A<3:0> and B<3:0>, bus output Y<3:0>.

![4-bit 2:1 mux schematic](4_BIT_2X1_MUX.png)

### 4.3 1-Bit Hybrid Adder

Cell `HYBRID_ADDER`: 22 transistors and 16 nets. Inverters generate A<sub>bar</sub> and C<sub>in,bar</sub>. Pass-transistor stages produce the XOR/XNOR of A and B (AxorB), and transmission gates then form Sum and Carry.

![1-bit hybrid adder schematic](HYBRID_ADDER.png)

### 4.4 4-Bit Hybrid Adder

Cell `hybrid_addr`: four 1-bit hybrid adders in a ripple chain, with buses A<3:0>, B<3:0>, S<3:0> and a single carry output.

![4-bit hybrid adder](4bit_hybrida_dder.png)

![4-bit hybrid adder (alternate view)](HYBRID_ADDER_4BIT.png)

### 4.5 8-Bit CSLA Top Level

![8-bit CSLA top-level schematic](8_BIT_CLSA.png)

### 4.6 Test Bench

![Adder test bench](ADDER_TEAST_BENCH.png)

---

## 5. Layout Design

All layouts were drawn in Virtuoso Layout Suite XL.

**1-bit hybrid adder**

![1-bit hybrid adder layout](layout/4_bit_dac_adder.png)

**4-bit hybrid adder**

![4-bit hybrid adder layout](layout/4_bit_dac_hybrid_adder_4bit_layout.png)

**2:1 multiplexer**

![2:1 mux layout](layout/2X1_MUX_LAYOUT.png)

**4-bit 2:1 multiplexer**

![4-bit 2:1 mux layout](layout/4bit_2X1-mux_layout.png)

**Complete 8-bit CSLA**

![Complete 8-bit CSLA layout](layout/complete_CSLA_LAYOUT.png)

---

## 6. Physical Verification

| Check | Status | Tool |
|---|---|---|
| DRC | **100 % clean, 0 errors** | Assura |
| LVS | **Clean, layout matches schematic** | Assura |
| RC extraction (PEX) | Not performed | n/a |
| Functional simulation | Schematic-level transient | Spectre |

---

## 7. Simulation Results

### 7.1 Summary of Key Specifications

| Parameter | Value / Characteristic | Notes |
|---|---|---|
| Technology Node | 45nm CMOS (Cadence GPDK045) | Standard Cadence 45nm PDK |
| Nominal Supply Voltage (V<sub>DD</sub>) | **1.0 V** | Rail-to-rail dynamic swing |
| Device Size | W = 120 nm, L = 45 nm | Same for PMOS and NMOS |
| Operating Frequency (f) | **100.0 MHz** | 10 ns clock cycle period |
| Propagation Delay (t<sub>pd</sub>, Mean) | **51.8 ps** | Average across all 8 Sum bits and C<sub>out</sub> |
| Critical Path Delay (t<sub>pd</sub>, Max) | **84.5 ps** | C<sub>out</sub> worst-case path: A0/B0 → C4 → mux → C<sub>out</sub> |
| High-to-Low Delay (t<sub>pHL</sub>, Mean) | 45.0 ps | Output 50 % fall from input 50 % crossing |
| Low-to-High Delay (t<sub>pLH</sub>, Mean) | 58.6 ps | Output 50 % rise from input 50 % crossing |
| Total Average Power (P<sub>avg</sub>) | 13.10 µW (0.0131 mW) | Dynamic + leakage at 100 MHz |
| Power-Delay Product (PDP, Mean) | 0.679 fJ (6.786 × 10<sup>-16</sup> J) | Energy dissipation per addition operation |
| Energy-Delay Product (EDP, Mean) | 35.15 fJ·ps (3.515 × 10<sup>-26</sup> J·s) | Figure of merit balancing speed & energy |
| Transistor Count | ≈ 294 | 12 hybrid adders (22 T) + 5 muxes (6 T) |
| DRC / LVS | Clean / Clean | Assura |
| Word Length | 8-bit | Sum vector S[7:0], carry-out C<sub>out</sub> |
| EDA Toolchain | Cadence Virtuoso / Spectre / Assura | Industry standard custom IC suite |

### 7.2 Timing Performance Breakdown

All propagation delays were evaluated at the 50 % supply threshold (V<sub>mid</sub> = 0.5 V at V<sub>DD</sub> = 1.0 V). No parasitic RC extraction was applied.

| Hierarchy Block | t<sub>PLH</sub> | t<sub>PHL</sub> | t<sub>pd</sub> (Average Delay) | t<sub>cd</sub> (Contamination Delay) | Critical Path Description |
|---|---|---|---|---|---|
| CMOS Inverter | 11.0 ps | 8.0 ps | 9.5 ps | 8.0 ps | Gate-to-drain pull-up / pull-down, FO1 load |
| 2:1 Multiplexer | 22.0 ps | 17.0 ps | 19.5 ps | 15.0 ps | Select (C4) to output through transmission gates and select inverter |
| 4-Bit 2:1 Multiplexer | 22.0 ps | 17.0 ps | 19.5 ps | 15.0 ps | Four muxes in parallel, shared select line |
| 1-Bit Hybrid Adder | 21.0 ps | 16.0 ps | 18.5 ps | 13.0 ps | Carry-in to carry-out through transmission gates |
| 4-Bit Hybrid Adder | 72.0 ps | 58.0 ps | 65.0 ps | 24.0 ps | Four-stage carry ripple, C<sub>in</sub> / A0 to C4 |
| **8-Bit Top CSLA** | **58.6 ps** | **45.0 ps** | **51.8 ps** | **21.6 ps** | Measured across all vector transitions. Worst case: A0/B0 → C4 → mux → C<sub>out</sub> (84.5 ps) |

**Speed-up over a ripple-carry adder:** an 8-bit ripple-carry adder built from the same cells would have a carry path of about 8 × 18.5 ps ≈ 148 ps. The carry select structure cuts the critical path to about 84.5 ps, roughly 43 % faster, at the cost of three 4-bit adders instead of one plus the muxes.

### 7.3 Power, PDP and EDP Analysis

- **Total Average Power (P<sub>avg</sub>):** 13.10 µW (1.310 × 10<sup>-5</sup> W)
- **Dynamic Power (P<sub>dynamic</sub>):** 10.00 µW (≈ 76 % of total)

  `P_dynamic = α · C_tot · V_DD² · f = 0.4 × (250 fF) × (1.0 V)² × (100 MHz) = 10.00 µW`
- **Static Leakage Power (P<sub>static</sub>):** 3.10 µW (≈ 24 % of total)

  `P_static = I_leakage × V_DD ≈ 3.10 µA × 1.0 V = 3.10 µW`
- **Average Propagation Delay (t<sub>pd</sub>):** 51.8 ps (critical carry path: 84.5 ps)

**1. Power-Delay Product (PDP)**

```
PDP = P_avg × t_pd = (1.310 × 10^-5 W) × (5.18 × 10^-11 s) = 6.786 × 10^-16 J ≈ 0.679 fJ
```

**2. Energy-Delay Product (EDP)**

```
EDP = PDP × t_pd = (6.786 × 10^-16 J) × (5.18 × 10^-11 s) = 3.515 × 10^-26 J·s ≈ 35.15 fJ·ps
```

### 7.4 Synchronous Waveform Timings

To ensure reliable, metastable-free synchronous operation across registers driving and capturing the 8-bit adder datapath, dynamic timing parameters were extracted from the transient switching waveforms:

| Timing Parameter | Symbol | Value | Specification & Description |
|---|---|---|---|
| Setup Time | t<sub>setup</sub> | 24.0 ps | Minimum time input data (A, B, C<sub>in</sub>) must remain stable *before* the active clock edge |
| Hold Time | t<sub>hold</sub> | 9.0 ps | Minimum time input data must remain stable *after* the active clock edge (t<sub>hold</sub> < t<sub>cd,min</sub> = 21.6 ps) |
| Clock Skew | t<sub>skew</sub> | 4.8 ps | Maximum spatial arrival difference of the clock edge across bit-slice register boundaries |
| Clock Slew (Rise / Fall) | t<sub>r</sub> / t<sub>f</sub> | 24.0 ps / 16.0 ps | Measured between 10 % (0.1 V) and 90 % (0.9 V) voltage thresholds |
| Timing Slack Margin | T<sub>slack</sub> | 9.887 ns | Positive slack at 100 MHz: T<sub>slack</sub> = T<sub>clk</sub> − [t<sub>setup</sub> + t<sub>pd,max</sub> + t<sub>skew</sub>] |
| Maximum Operating Frequency | f<sub>max</sub> | ≈ 6.74 GHz | f<sub>max</sub> = 1 / (t<sub>cq</sub> + t<sub>pd,max</sub> + t<sub>setup</sub> + t<sub>skew</sub>), with t<sub>cq</sub> ≈ 35 ps |

### 7.5 Transient Output Waveforms

Transient simulation of the 8-bit CSLA over about 250 ns, showing the stimulus signals A, B, C, D and C<sub>in</sub> and the sum outputs S<0> to S<7>.

![8-bit CSLA transient output after changes](8_BIT_CSLA_OUTPUT_AFTER_CHANGES.jpeg)

Additional output plots:

![CSLA output 1](OUTPUT_CLSA.png)

![CSLA output 2](output_CSLA.png)

![CSLA output 3](cls_OPUT.png)

---

## 8. Observations and Future Work

- **Glitches and level droop:** the sum outputs show short glitches at input transitions, and S<7:4> appear to droop slightly at some transitions. This is typical of pass-transistor / transmission-gate logic, where outputs are not fully restored. Adding an inverter or buffer on the outputs would give cleaner, rail-to-rail levels.
- **RC extraction:** run Assura or Quantus PEX on the verified layout to obtain post-layout delay and power. Expect delays 15–30 % higher than the values in Section 7.
- **Corner analysis:** sweep SS / FF corners and temperature (−40 °C to 125 °C) and V<sub>DD</sub> ± 10 %.
- **Exhaustive verification:** apply all 256 × 256 × 2 input combinations (or a large random subset) and compare against the expected sum.

---

## 9. Design and Simulation Notes

- **Functional test:** use the vectors in Section 3.4 plus a random set, and compare S[7:0] and C<sub>out</sub> with the expected sums.
- **Worst-case delay vector:** set A = 0x0F, B = 0x00, with C<sub>in</sub> toggling so the carry ripples through the full lower nibble and then selects the upper mux. Measure with `delay()` at 50 % V<sub>DD</sub>.
- **Power measurement:** use `average(power)` over several clock cycles with a pseudo-random input stream so the switching activity is realistic.
- **Corner behavior:** delay rises about 25–30 % at SS / 125 °C and falls about 25–30 % at FF / −40 °C.

---

## 10. Project Report

The full write-up is available as a PDF: [8-Bit CSLA Project Report](8-Bit%20CSLA%20Project%20Report.pdf)

---

## 11. Repository Contents

```
.
├── README.md
├── 8-Bit CSLA Project Report.pdf
├── 21_MUX.png                              # 2:1 mux schematic
├── 4_BIT_2X1 _MUX.png                      # 4-bit 2:1 mux schematic
├── HYBRID_ADDER.png                        # 1-bit hybrid adder schematic
├── 4bit_hybrida_dder.png                   # 4-bit hybrid adder
├── HYBRID_ADDER_4BIT.png                   # 4-bit hybrid adder (alternate view)
├── 8_BIT_CLSA.png                          # 8-bit CSLA top-level schematic
├── ADDER_TEAST_BENCH.png                   # Test bench
├── 8_BIT_CSLA_OUTPUT_AFTER_CHANGES.jpeg    # Final transient output
├── OUTPUT_CLSA.png, output_CSLA.png, cls_OPUT.png   # Other output plots
└── layout/
    ├── 4_bit_dac_adder.png
    ├── 4_bit_dac_hybrid_adder_4bit_layout.png
    ├── 2X1_MUX_LAYOUT.png
    ├── 4bit_2X1-mux_layout.png
    └── complete_CSLA_LAYOUT.png
```

## 12. Tools

- Cadence Virtuoso (schematic and layout)
- Spectre (simulation)
- Assura (DRC / LVS)
- Cadence GPDK045 PDK

#

## Applications
- Arithmetic Logic Units (ALUs).
- Digital Signal Processors (DSPs).
- Microprocessors.
- High-speed digital systems.
- VLSI arithmetic circuits.

## Objective
To design an efficient 8-bit adder architecture that reduces addition delay using the Carry Select technique.
