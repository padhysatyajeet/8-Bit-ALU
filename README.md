
# 8 - Bit ALU
### eSim Marathon
  - [Abstract](#abstract)
  - [Reference Circuit Details](#reference-circuit-details)
  - [Technology & environment](#technology--environment)
  - [Simulation in esim](#simulation-in-esim)
  - [Final Schematic](#final-schematic)
  - [Circuit Waveform](#circuit-waveform)
  - [Contributor](#contributor)
  - [Acknowledgement](#acknowledgement)
  - [References](#references)

## Abstract
This project implements an 8-bit Arithmetic Logic Unit (ALU) designed and verified at the MOSFET (transistor) level in esim. The ALU performs eight operations selected by a 3-bit control word (S2 S1 S0): OR, XNOR, XOR, AND, INC, ADD, DEC, and SUB (see table below). All basic logic gates (NOT, OR, XOR, XNOR, AND) and 2:1 multiplexers were realized as discrete MOSFET circuits using 180 nm technology with Vdd = 5 V; those gates were used as building blocks to create half-adders / half-subtractors and the multi-bit datapath. Multi-bit operations are implemented by replicating the bit-slice circuitry across 8 bits and by appropriate carry handling for arithmetic results. The design includes a dedicated carry flag (MSB) logic so that operations that produce results wider than 1 bit present their MSB as a carry flag; for purely bitwise logical ops the carry flag is forced to 0. Functional verification (opcode sweeps and representative vectors) was performed in esim; schematics and waveforms are included in the report. (Report formatting follows the provided example.)

## Reference Circuit Details

### Operation map (select lines)

S2 S1 S0 → Operation

000 → OR

001 → XNOR

010 → XOR

011 → AND

100 → Increment (A + 1)

101 → Addition (A + B)

110 → Decrement (A − 1)

111 → Subtraction (A − B)

### Fundamental building blocks
Inverter (NOT) — MOSFET implementation.

2-input gates: OR, AND — transistor networks implemented in MOSFET style.

XOR, XNOR — implemented using MOSFET transistor-level topologies (constructed from the basic gates where appropriate).

2:1 MUX (master primitive) — MOSFET-level pass/transmission gate or complementary MUX implementation; this is the fundamental multiplexer building block used to construct larger MUXes.

### Adders / Subtractors
Half-adder building block: implemented using XOR (sum) and AND (carry) primitives. You used these half-adder elements as the basic data path for bitwise addition where appropriate.

Half-subtractor building block: implemented using XOR (difference), AND and NOT (borrow) primitives; used to create bitwise subtraction elements.

Multi-bit arithmetic: the 8-bit arithmetic datapath is built by replicating the bit-slice adder/subtractor logic across all 8 bits. Carry/borrow propagation between bit-slices is used for full multi-bit addition/subtraction (i.e., a ripple style chaining of carry/borrow signals produced by the per-bit adder/subtractor blocks).

Increment / Decrement: implemented by using the adder/subtractor with the second operand tied to logic 1 (Vdd) so that A + 1 and A − 1 are produced without a separate dedicated incrementer/decrementer block.

### Multiplexer architecture and MSB (carry-flag) selection
LSB..bit outputs: For each of the 8 bits the 8:1 data multiplexer selects which operation’s bit result is presented to that bit of the ALU output according to S2 S1 S0.

8:1 MUX construction: implemented using seven 2:1 MUX primitives (i.e., a tree of 2:1 multiplexers) — all 2:1 MUXes were implemented at MOSFET level.

MSB / carry flag logic: because the last four operations (INC, ADD, DEC, SUB — select codes 100..111) can produce a 2-bit result (LSB + MSB/carry), a separate path selects the MSB (carry) for the ALU:

A 4:1 MUX selects the MSB bits produced by the last four arithmetic operations (inputs = the MSBs of INC, ADD, DEC, SUB). This 4:1 MUX is implemented using three 2:1 MUX primitives.

The 4:1 MUX output is fed to the in1 input of a final 2:1 MUX whose select is S2. The in2 of that final 2:1 MUX is tied to GND (logic 0). This arrangement yields:

when S2 = 0 (first four logical ops), the carry flag is forced to 0 (in2 = 0),

when S2 = 1 (last four arithmetic ops), the carry flag equals the selected MSB from the 4:1 MUX (in1).

Why this arrangement: it cleanly separates logical 1-bit outputs (carry = 0) from arithmetic ops that may produce an extra MSB; it also minimizes extra logic by reusing the 2:1 MUX primitive.

### Output format
Primary output: 8-bit result bus (bit 7 = most significant bit of the chosen operation’s LSB output when weighted as normal).

Carry flag / extra MSB: separate 1-bit carry flag that is the MSB of arithmetic results (selected and gated as described above). For purely logical operations (OR, XNOR, XOR, AND) the carry flag = 0.

### Implementation notes & verification
All gates and multiplexers were designed at transistor level (no behavioral primitives). The 2:1 MUX design was re-used to compose 4:1 and 8:1 multiplexers (3×2:1 → 4:1; 7×2:1 → 8:1 tree).

Arithmetic building blocks were constructed from your MOS-level XOR/AND/NOT primitives (you mentioned implementing half-adder and half-subtractor as XOR+AND and XOR+AND+NOT respectively). Those blocks were replicated across 8 bits and connected with carry/borrow chaining for multi-bit add/sub operations.

Increment/Decrement were realized by driving the second operand to logic 1 (Vdd) into the adder/subtractor block.

Simulation checklist (recommended): opcode sweep (all 8 opcodes) on representative vectors (0x00, 0x01, 0x7F, 0x80, 0xFF), verify carry flag behavior for boundary conditions (e.g., 0xFF + 1), and capture timing waveforms for a typical add and a typical logic operation. Include schematic screenshots and waveforms in the report (figure captions: “Fig. X — MOSFET-level 2:1 MUX”, “Fig. Y — 8-bit ALU top-level schematic”, “Fig. Z — Waveforms for opcode 101 (ADD) showing carry” etc.).

## Technology & environment
Process: 180 nm CMOS (MOSFET-level transistors).

Supply: Vdd = 5 V (logic high), GND = 0 V (logic low).

Simulator: esim — all gates and MUXes built and tested at transistor level.

# Simulation in esim

## CMOS Inverter
<img width="682" height="652" alt="image" src="https://github.com/user-attachments/assets/992d08f2-7594-4608-aef3-6211fb12d4c3" />


## OR Gate
<img width="1390" height="721" alt="image" src="https://github.com/user-attachments/assets/0a0fd59c-7601-4a2e-957c-303dae8af16e" />


## AND Gate
<img width="1461" height="803" alt="image" src="https://github.com/user-attachments/assets/f0cee83f-a63e-4b04-b894-42ab6dbc633a" />


## XOR Gate
<img width="1695" height="791" alt="image" src="https://github.com/user-attachments/assets/ddb5e1cc-7f1a-47ed-98d8-447662a4af64" />


## XNOR Gate
<img width="1472" height="794" alt="image" src="https://github.com/user-attachments/assets/a6e6ed7c-b295-4eb0-809d-96cb8cad8e7d" />


## 2:1 MUX
<img width="1717" height="733" alt="image" src="https://github.com/user-attachments/assets/f220a9b5-6315-4b02-9be9-efd5c515d832" />


## 4:1 MUX using 2:1 MUXes
<img width="1227" height="812" alt="image" src="https://github.com/user-attachments/assets/d9c3e1dc-fa5b-4cd9-abfb-e81a62f9e86b" />


## 8:1 MUX using 2:1 MUXes
<img width="1415" height="813" alt="image" src="https://github.com/user-attachments/assets/154e2bf9-c550-4187-b72d-bdbe34e309b3" />


## Adder using XOR & AND Subcircuits
<img width="1671" height="738" alt="image" src="https://github.com/user-attachments/assets/c71c06dc-2265-4657-b8c2-bb341766c8e8" />


## Subtracter using XOR, AND & NOT Subcircuits
<img width="1728" height="689" alt="image" src="https://github.com/user-attachments/assets/e7c3af2d-4319-4911-9184-ebffa9a810ca" />


## Final Schematic
The Final Circuit is compoed of 8:1 MUX, 4:1 MUX, Adders, Subtractors & NOT Gate Subcircuits.
<img width="1169" height="809" alt="Screenshot 2025-10-27 190709" src="https://github.com/user-attachments/assets/206baa30-21ee-4fda-9315-f947d28a904a" />


## Circuit Waveform
### Select Lines
#### Sel0
<img width="891" height="640" alt="Screenshot 2025-10-27 191125" src="https://github.com/user-attachments/assets/0bdfea5b-c9de-4e52-9c6e-d5a8376fd6bc" />

#### Sel1
<img width="892" height="637" alt="Screenshot 2025-10-27 191146" src="https://github.com/user-attachments/assets/2b9b6f32-8ebc-44b6-91c3-1ef9e12290c5" />

#### Sel2
<img width="887" height="645" alt="Screenshot 2025-10-27 191206" src="https://github.com/user-attachments/assets/fa6e628d-0bef-406a-8b6b-d1678fcd3ba5" />

### Inputs
#### A (= 0V )(Logic 0)
<img width="884" height="646" alt="Screenshot 2025-10-27 191232" src="https://github.com/user-attachments/assets/30116c79-53cb-4296-b0c2-536b803b005c" />

#### B (= 5V )(Logic 1)
<img width="881" height="633" alt="Screenshot 2025-10-27 191241" src="https://github.com/user-attachments/assets/480200ec-8cef-4a34-b7eb-26d3814d5553" />

### Output
#### V(out)
<img width="892" height="642" alt="Screenshot 2025-10-27 191300" src="https://github.com/user-attachments/assets/e8a5f40d-c32a-4bb2-83cc-0d795e7b2188" />

#### Carry Flag
<img width="888" height="635" alt="Screenshot 2025-10-27 191249" src="https://github.com/user-attachments/assets/61099410-a99c-4cef-92cd-4f1de3f3065b" />


## Contributor
**Satyajeet Padhy**  
B.Tech, Dept. of Electronics and Communication Engineering  
National Institute of Technology, Rourkela  
📧 *padhysatyajeet@gmail.com*  

## Acknowledgement
Special thanks to the **FOSSEE eSim Team, IIT Bombay**, for providing open-source EDA tools and guidance under the **FOSSEE Circuit Simulation Project** initiative.

## References
[1] Manit Kantawala, “Design and implementation of 8 bit and 16 bit ALU using Verilog language” in nternational Journal of Engineering Applied Sciences and Technology, 2018 Vol. 3, Issue 2, ISSN No. 2455-2143, Pages 30-34 Published Online June 2018 in IJEAST (http://www.ijeast.com) 

[2] C.Arunabala, Ch. Jyothirmayi, D N S V Sreeja.T, Suma Burra, Hrithika Reddy Udumula, I.R.Anusha Devi, “Design of a 4 bit Arithmetic and Logical unit with Low Power and High Speed” in International Journal of Innovative Technology and Exploring Engineering (IJITEE) ISSN: 2278-3075 (Online), Volume-10 Issue-5, March 2021

