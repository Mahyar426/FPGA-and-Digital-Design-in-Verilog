# ⚡ FPGA & Digital Design in Verilog

A full-stack Verilog HDL portfolio spanning six progressive modules — from gate-level combinational logic to a **dual-clock, UART-interfaced matrix co-processor** synthesized for Xilinx FPGAs. Every design is verified with custom testbenches and documented with detailed reports.

---

## 🔍 What's Inside

Six hardware modules, each building on the last. By the end, the designs integrate DCM clock management, block RAM, FIFO buffering, UART communication, and multi-operation matrix arithmetic — all wired together in a single synthesizable top-level.

---

## 🗂️ Repository Structure

```
📦 FPGA-Digital-Design/
├── HW1/   Combinational Logic Foundations
│   ├── Q2/         4-bit hierarchical comparator (gate-level)
│   ├── Q3/         ALU with 4-bit ripple-carry adder & MUX fabric
│   ├── Q4/         Combinational lock circuit
│   └── Q_Optional/ ASCII case-conversion unit
│
├── HW2/   Datapath Components
│   ├── Q1/         8-bit bidirectional barrel rotate shifter
│   ├── Q2/         16-bit Hamming distance calculator
│   └── Q3/         4-op ALU + hex-to-7-segment display driver
│
├── HW3/   Clocked Sequential Systems
│   ├── Q1/         Real-time digital speedometer (pulse-width measurement)
│   ├── Q2/         Overlapping & non-overlapping pattern detector FSM
│   ├── Q3/         CRC-5 LFSR — MSB-first and LSB-first variants
│   └── Q4/         UART receiver with start/stop framing validation
│
├── HW4/   IP Cores & Memory Architectures
│   ├── Q1/         Tri-state bi-directional bus — two-module handshake over inout
│   ├── Q2/         Block RAM-accelerated 64-element vector multiply
│   ├── Q3/         Clocked 8×8 vector dot-product accumulator (registers)
│   ├── Q4/         Dual-port Block RAM dot-product engine (Xilinx IP core)
│   ├── Q5/         FIFO + DCM clock domain crossing — 250 MHz → 100 MHz averager
│   └── opt/        Edge-triggered speedometer (rising/falling edge detection)
│
├── HW5/   Advanced Sequential & Communication
│   ├── Q1/         Full UART transceiver — Hamming-coded 4-bit TX + error-checked RX
│   ├── Q2/         Multi-sensor industrial FSM (overheat / stuck / low-material)
│   ├── Q3/         6-state Moore FSM with 2-bit encoded input
│   └── Q4/         FIFO controller with Xilinx Block RAM IP integration
│
└── HW6/   Matrix Co-Processor System (Full SoC Integration)
    ├── top.v          System top-level — UART RX → memory → compute → UART TX
    ├── calc.v         32×32 matrix ALU (8 operations: A+B, A-B, B-A, A×B, Aᵀ, Bᵀ, tr(A), tr(B))
    ├── UART_rec.v     UART receiver feeding block RAM write port
    ├── UART_trans.v   UART transmitter draining result block RAM (1024 elements)
    ├── clk_gen.v      DCM wrapper — generates clk_calc and clk_uart domains
    └── ipcore_dir/    Three dual-port block RAMs + DCM (Xilinx Coregen)
```

---

## 🧠 Technical Highlights

### HW1 — Combinational Logic

| Module | What it does | Design technique |
|---|---|---|
| `fourbitcomp` | Hierarchical 4-bit magnitude comparator | Gate-level `bufif1` tri-state chaining |
| `myALU` | 4-bit ALU (ADD / SUB / pass / invert) | MUX-controlled ripple-carry adder |
| `Lock` | Combinational digital lock | Boolean minimization, direct gate instantiation |
| `ASCIIConverter` | Lowercase ↔ uppercase ASCII converter | Single XOR on ASCII bit[5] |

---

### HW2 — Datapath Components

| Module | What it does | Design technique |
|---|---|---|
| `barrelSR` | 8-bit bidirectional barrel rotate shifter | Full case-expression, all 16 `{ctrl, shift_mag}` combos |
| `hamming_distance` | 16-bit Hamming distance in one cycle | Parallel XOR reduction + tri-state enable |
| `alu` + `hexto7segment` | 4-op ALU piped into 7-segment display | Bitfield masking; complete 0–F segment LUT |

---

### HW3 — Clocked Sequential Systems

| Module | What it does | Design technique |
|---|---|---|
| `speedometer` | Real-time speed from pulse width | Clock-cycle counting; on-the-fly division |
| `stringdetectora` | Serial pattern detector (overlapping) | Shift-register FSM, sliding window compare |
| `crc5` | CRC-5 serial checksum — both bit-order variants | LFSR with XOR feedback taps (USB polynomial) |
| `reciever` | UART frame receiver | 10-bit shift register + `{start,stop}` framing check |

---

### HW4 — IP Cores & Memory Architectures

The step up to real FPGA primitives: tri-state buses, Xilinx block RAMs, FIFO generators, and the Digital Clock Manager (DCM).

| Module | What it does | Key technique |
|---|---|---|
| Q1 `Q1_1` / `Q1_2` | Bi-directional tri-state bus handshake | `inout` port, `IO_Select` phased write/read over shared wire |
| Q2/Q3 `q3module` | 64-element 8-bit vector multiply + accumulate | Register array; sequential multiply-accumulate loop |
| Q4 `q4_first_try` | Same vector engine backed by dual-port block RAM | Xilinx `q4core` (read-write port split across negedge/posedge) |
| Q5 `topq5` | 250 MHz write / 100 MHz read FIFO averager | Xilinx `my_fifo` + `mydcm` DCM; 4-sample sliding average |
| `opt` | Edge-detecting speedometer | XOR of current/last pulse for rising+falling edge count |

The **FIFO+DCM** design is particularly significant: it crosses two clock domains generated from a single input clock by the on-chip DCM, using the FIFO as the safe crossing primitive — the same pattern used in real high-speed FPGA datapaths.

---

### HW5 — Advanced Sequential & Communication

| Module | What it does | Key technique |
|---|---|---|
| `uart_sender` | UART TX with in-hardware Hamming encoding | Parity task generates 3 check bits inline; serial shift-register output |
| `uart_reciever` | UART RX with parity error detection | Checks Hamming bits on received frame |
| `q2_first_try` | Industrial machine FSM (5 states, 4 sensors) | Named `localparam` states; timer-gated transitions; `on_off` output |
| `q3_first_try` | 6-state Moore FSM, 2-bit input | Fully separated: combinational next-state + registered state + output |
| `fifofirst` | FIFO controller wrapping Xilinx Block RAM IP | Xilinx `core1` BMG; separate read/write pointer logic |

The **UART transmitter** encodes 4-bit data with a 3-bit Hamming parity code before serialization — forward error correction baked directly into the hardware frame, not software.

---

### HW6 — Matrix Co-Processor System *(Capstone)*

The most advanced design in the portfolio: a **synthesized FPGA SoC** that receives two 32×32 matrices over UART, computes one of eight operations, stores the result in block RAM, and transmits it back — all at hardware speed.

```
UART RX (A) ──→ Block RAM A ──┐
                               ├──→ calc (32×32 ALU) ──→ Block RAM C ──→ UART TX
UART RX (B) ──→ Block RAM B ──┘
```

**`calc.v` — 8 matrix operations, fully synthesized:**

| Op code | Operation |
|---|---|
| `000` | Element-wise A + B |
| `001` | Element-wise A − B |
| `010` | Element-wise B − A |
| `011` | Matrix multiply A × B (32×32, dot-product accumulation) |
| `100` | Transpose of A |
| `101` | Transpose of B |
| `110` | Trace of A (diagonal sum) |
| `111` | Trace of B |

**System-level engineering highlights:**

- **Dual clock domain**: DCM generates `clk_uart` and `clk_calc` from a single input; block RAMs bridge the domains with separate read/write ports clocked independently
- **1024-element result streaming**: UART transmitter walks the result block RAM address-by-address, serializing each 8-bit element as a framed UART byte
- **Result truncation / normalization**: `calc.v` locates the first significant bit in the 22-bit product accumulator and right-shifts to fit the 8-bit output — hardware-level fixed-point normalization
- **Trace output path**: diagonal sum wired through a separate output (`trace_out`) and mux-selected at the transmitter for `Op >= 6`
- **Fully wired top-level**: `top.v` connects all seven submodules with proper clock gating, flag-based transmit triggering, and input registration for metastability

---

## 🛠️ Tools & Environment

| Tool | Purpose |
|---|---|
| Xilinx ISE / Vivado | Synthesis, place & route, bitstream generation |
| Xilinx Coregen | Block RAM (BMG), FIFO Generator, DCM/Clock Wizard IP |
| ModelSim / ISim | Behavioral simulation & waveform verification |
| Verilog HDL | RTL description throughout |
| Custom testbenches | Every module has a dedicated `*_test.v` or `test_*.v` file |

---

## 💡 Skills Demonstrated

- **Gate-level design** — explicit primitive instantiation (`and`, `or`, `bufif1`, `not`)
- **Behavioral & dataflow RTL** — mixed-style across all modules
- **Hierarchical composition** — deep module hierarchies (top → UART → memory → calc)
- **Xilinx IP core integration** — Block RAM (dual-port), FIFO Generator, DCM / Clock Wizard
- **Clock domain crossing** — FIFO-based CDC with DCM-generated multi-clock outputs
- **FSM design** — Moore machines, industrial state diagrams, pattern detectors
- **LFSR / CRC** — real error-detection logic (CRC-5, USB polynomial, both bit orders)
- **UART — full stack** — receiver, transmitter, Hamming encoding, frame validation
- **Matrix arithmetic in hardware** — 32×32 multiply, transpose, trace, element-wise ops
- **Fixed-point normalization** — bit-position detection and shift for 8-bit output packing
- **Self-verification** — every design ships with a testbench

---

## 📄 Reports

Each module folder contains a detailed PDF covering architecture decisions, simulation waveforms, and edge case analysis.

---
