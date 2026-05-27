# 5-Stage Pipelined RISC-V Processor with Hazard Unit

A fully functional 64-bit pipelined RISC-V processor designed and simulated in [Digital](https://github.com/hneemann/Digital), a digital logic simulator. The processor implements a classic 5-stage pipeline and includes a custom Hazard Unit that automatically resolves data and control hazards — eliminating the need for manual NOP instructions.

---

## Pipeline Stages

| Stage | Name | Description |
|-------|------|-------------|
| 0 | Instruction Fetch | Fetches instruction from ROM using the PC; PC increments by 4 each cycle |
| 1 | Decode / Register Read | Decodes the instruction and reads source registers from the register file |
| 2 | Execute | ALU performs the operation; branch unit evaluates branch conditions |
| 3 | Memory | Reads or writes data RAM for `lw`/`sw` instructions |
| 4 | Write Back | Writes the result back to the destination register |

Each stage is separated by a pipeline register that latches all signals needed by downstream stages.

---

## Hazard Unit

The Hazard Unit is the core addition to the baseline processor. It monitors the pipeline and automatically resolves three classes of hazards:

### 1. Data Hazard — Register Forwarding
When an instruction in EX needs a value that a later stage hasn't written back yet, the hazard unit forwards the value directly from the pipeline register.
- Compares source registers (RS1, RS2) against destination registers of instructions in EX/MEM (WR3) and MEM/WB (WR4)
- Outputs 2-bit forwarding selects FWDRS1 and FWDRS2 to muxes at the ALU inputs
- `00` = use register file, `01` = forward from MEM/WB, `10` = forward from EX/MEM

### 2. Load-Use Hazard — Pipeline Stall
A `lw` result is not available until after the MEM stage — one cycle too late for the immediately following instruction.
- Detects: MR3 = 1 (instruction in EX is a load) AND (WR3 == RR0 OR WR3 == RR1)
- Asserts STALL, which holds the PC and IF/DR register (EN = 0)
- Clears the DR/EX register to insert a NOP bubble into the EX stage

### 3. Control Hazard — Pipeline Flush
When a branch or jump is taken, instructions that entered the pipeline behind it must be discarded.
- PCbr is asserted by the branch unit when a branch is taken
- Hazard unit asserts FLUSH, which clears the IF/DR and DR/EX pipeline registers
- OR-ed with the global CLR signal so resets and flushes are handled uniformly

---

## Project Structure

---

## Test Programs

| Test | Description | Hazard Type |
|------|-------------|-------------|
| `00-add-3nop` | ADD with 3 manual NOPs | None (baseline) |
| `01-add-2nop` | ADD with 2 manual NOPs | None (baseline) |
| `02-jal` | Jump and link with NOPs | None (baseline) |
| `03-ld` | Load/store with NOPs | None (baseline) |
| `04-add-fwd` | ADD with no NOPs | Data hazard → forwarding |
| `05-ld-stl` | Load followed immediately by use | Load-use hazard → stall |
| `06-jal-fls` | Jump with no NOPs | Control hazard → flush |

---

## How to Run

1. Download and install [Digital](https://github.com/hneemann/Digital/releases)
2. Clone this repository:
```bash
   git clone https://github.com/ayacheikh/riscv-pipeline-processor
```
3. Open `project07.dig` in Digital
4. Select a program using the `PROG` input
5. Use the test buttons to run automated test cases

---

## Tools & Concepts
- Digital (logic simulator)
- RISC-V ISA (RV64I subset)
- Pipelining, hazard detection, register forwarding
- Computer Architecture

