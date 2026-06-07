<div align="center">

![header](https://readme-typing-svg.demolab.com?font=Fira+Code&size=26&pause=1000&color=F5A623&center=true&vCenter=true&width=800&lines=⚡+FPGA+%26+Digital+Design+in+Verilog;Gate-level+logic+to+32×32+matrix+co-processor.;Six+modules.+Real+silicon.+All+verified.)

```
██╗   ██╗███████╗██████╗ ██╗██╗      ██████╗  ██████╗
██║   ██║██╔════╝██╔══██╗██║██║     ██╔═══██╗██╔════╝
██║   ██║█████╗  ██████╔╝██║██║     ██║   ██║██║  ███╗
╚██╗ ██╔╝██╔══╝  ██╔══██╗██║██║     ██║   ██║██║   ██║
 ╚████╔╝ ███████╗██║  ██║██║███████╗╚██████╔╝╚██████╔╝
  ╚═══╝  ╚══════╝╚═╝  ╚═╝╚═╝╚══════╝ ╚═════╝  ╚═════╝
         hardware that does the math
```

[![Verilog](https://img.shields.io/badge/HDL-Verilog-F5A623?style=flat-square)](https://en.wikipedia.org/wiki/Verilog)
[![Xilinx](https://img.shields.io/badge/Toolchain-Xilinx_ISE_%2F_Vivado-E53935?style=flat-square)](https://www.xilinx.com/)
[![ModelSim](https://img.shields.io/badge/Simulation-ModelSim_%2F_ISim-5C6BC0?style=flat-square)](https://www.intel.com/content/www/us/en/software/programmable/quartus-prime/modelsim.html)
[![Domain](https://img.shields.io/badge/Domain-Digital_Design_%7C_RTL_%7C_FPGA-00897B?style=flat-square)](#)

</div>

---

A full-stack Verilog HDL portfolio spanning six progressive modules — from gate-level combinational logic to a **dual-clock, UART-interfaced 32×32 matrix co-processor** synthesized for Xilinx FPGAs. Every design is verified with custom testbenches, every module builds on the last.

---

## 🗂️ Repository Structure

```
📦 FPGA-Digital-Design/
├── HW1/   Combinational Logic Foundations
├── HW2/   Datapath Components
├── HW3/   Clocked Sequential Systems
├── HW4/   IP Cores & Memory Architectures
├── HW5/   Advanced Sequential & Communication
└── HW6/   Matrix Co-Processor System ← capstone
```

---

## 🧠 What's Inside — Module by Module

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
| Q1 `Q1_1` / `Q1_2` | Bi-directional tri-state bus handshake | `inout` port, `IO_Select` phased write/read |
| Q2/Q3 `q3module` | 64-element 8-bit vector multiply + accumulate | Register array; sequential MAC loop |
| Q4 `q4_first_try` | Same engine backed by dual-port block RAM | Xilinx `q4core` (read-write port split) |
| Q5 `topq5` | 250 MHz write / 100 MHz read FIFO averager | Xilinx `my_fifo` + `mydcm` DCM; 4-sample sliding average |

The **FIFO+DCM** design crosses two clock domains generated from a single input clock — the same pattern used in real high-speed FPGA datapaths.

---

### HW5 — Advanced Sequential & Communication

| Module | What it does | Key technique |
|---|---|---|
| `uart_sender` | UART TX with in-hardware Hamming encoding | 3 check bits generated inline; serial shift-register output |
| `uart_reciever` | UART RX with parity error detection | Checks Hamming bits on received frame |
| `q2_first_try` | Industrial machine FSM (5 states, 4 sensors) | Named `localparam` states; timer-gated transitions |
| `q3_first_try` | 6-state Moore FSM, 2-bit input | Separated combinational next-state + registered state + output |
| `fifofirst` | FIFO controller wrapping Xilinx Block RAM IP | Separate read/write pointer logic |

---

### HW6 — Matrix Co-Processor System *(Capstone)*

The most advanced design: a **synthesized FPGA SoC** that receives two 32×32 matrices over UART, computes one of eight operations, stores the result in block RAM, and streams it back — all at hardware speed.

```
UART RX (A) ──→ Block RAM A ──┐
                               ├──→ calc (32×32 ALU) ──→ Block RAM C ──→ UART TX
UART RX (B) ──→ Block RAM B ──┘
```

**`calc.v` — 8 matrix operations, fully synthesized:**

| Op code | Operation | Op code | Operation |
|---|---|---|---|
| `000` | Element-wise A + B | `100` | Transpose of A |
| `001` | Element-wise A − B | `101` | Transpose of B |
| `010` | Element-wise B − A | `110` | Trace of A |
| `011` | Matrix multiply A × B | `111` | Trace of B |

**System-level engineering highlights:**
- **Dual clock domain** — DCM generates `clk_uart` and `clk_calc` from a single input; block RAMs bridge domains with separate read/write ports
- **1024-element result streaming** — UART transmitter walks the result block RAM address-by-address
- **Fixed-point normalization** — `calc.v` locates the first significant bit in the 22-bit accumulator and right-shifts to fit 8-bit output
- **Fully wired top-level** — `top.v` connects all seven submodules with flag-based transmit triggering and metastability-safe input registration

---

## 🛠️ Tools & Environment

| Tool | Purpose |
|---|---|
| Xilinx ISE / Vivado | Synthesis, place & route, bitstream generation |
| Xilinx Coregen | Block RAM (BMG), FIFO Generator, DCM / Clock Wizard IP |
| ModelSim / ISim | Behavioral simulation & waveform verification |
| Verilog HDL | RTL description throughout |
| Custom testbenches | Every module has a dedicated `*_test.v` file |

---

## 💡 Skills Demonstrated

- **Gate-level design** — explicit primitive instantiation (`and`, `or`, `bufif1`)
- **Behavioral & dataflow RTL** — mixed-style across all modules
- **Xilinx IP core integration** — Block RAM, FIFO Generator, DCM / Clock Wizard
- **Clock domain crossing** — FIFO-based CDC with DCM-generated multi-clock outputs
- **FSM design** — Moore machines, industrial state diagrams, pattern detectors
- **UART — full stack** — receiver, transmitter, Hamming encoding, frame validation
- **Matrix arithmetic in hardware** — 32×32 multiply, transpose, trace, element-wise ops
- **Fixed-point normalization** — bit-position detection and shift for 8-bit output packing
- **Self-verification** — every design ships with a testbench

---

*Each module folder contains a detailed report covering architecture decisions, simulation waveforms, and edge-case analysis.*
