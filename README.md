# 8-bit Single-Cycle RISC-V Computer Processor

CO224 - Building a single cycle 8-bit RISC-V Computer Processor for basic ARM assembly code running

## Table of Contents
- [Introduction](#introduction)
- [RISC-V Architecture Overview](#risc-v-architecture-overview)
- [Single Cycle CPU Design](#single-cycle-cpu-design)
- [Instruction Formats](#instruction-formats)
- [Instruction Set Overview](#instruction-set-overview)
- [Datapath Components](#datapath-components)
- [Control Unit](#control-unit)
- [Instruction Examples](#instruction-examples)

## Introduction

This project implements a basic 8-bit single-cycle RISC-V processor. RISC-V (Reduced Instruction Set Computer - Five) is an open-source instruction set architecture (ISA) based on established RISC principles. A single-cycle processor executes each instruction in one clock cycle, making it simple to understand and implement.

## RISC-V Architecture Overview

RISC-V is a load-store architecture where:
- Operations are performed on registers
- Only load and store instructions access memory
- All instructions have a fixed length (32 bits in standard RISC-V)
- The architecture includes 32 general-purpose registers (x0-x31)
- Register x0 is hardwired to zero

### Key Features:
- **Simple and modular design**: Easy to implement and extend
- **Open-source**: No licensing fees or restrictions
- **Scalable**: From embedded systems to supercomputers
- **Clean separation**: Between user and privileged instructions

## Single Cycle CPU Design

A single-cycle processor completes each instruction in exactly one clock cycle. The main characteristics include:

### Advantages:
- Simple hardware design
- Easy to understand and debug
- Predictable timing

### Disadvantages:
- Clock cycle must be long enough for the slowest instruction
- Inefficient for instructions that could complete faster
- Lower maximum clock frequency

### Execution Steps (in one cycle):
1. **Instruction Fetch (IF)**: Fetch instruction from memory using Program Counter (PC)
2. **Instruction Decode (ID)**: Decode instruction and read registers
3. **Execute (EX)**: Perform ALU operation or calculate address
4. **Memory Access (MEM)**: Read from or write to data memory (if needed)
5. **Write Back (WB)**: Write result back to register file

## Instruction Formats

RISC-V uses six basic instruction formats. Each format is 32 bits wide:

### 1. R-Type (Register-Register Operations)
```
|   funct7   |  rs2  |  rs1  | funct3 |   rd   | opcode |
|  [31-25]   | [24-20]| [19-15]| [14-12]| [11-7] | [6-0]  |
|   7 bits   | 5 bits| 5 bits| 3 bits | 5 bits | 7 bits |
```
- **Used for**: Arithmetic and logical operations between registers
- **Examples**: ADD, SUB, AND, OR, XOR, SLT

### 2. I-Type (Immediate Operations)
```
|      imm[11:0]      |  rs1  | funct3 |   rd   | opcode |
|      [31-20]        | [19-15]| [14-12]| [11-7] | [6-0]  |
|      12 bits        | 5 bits| 3 bits | 5 bits | 7 bits |
```
- **Used for**: Operations with immediate values, loads
- **Examples**: ADDI, ANDI, ORI, LW, LB

### 3. S-Type (Store Operations)
```
| imm[11:5] |  rs2  |  rs1  | funct3 | imm[4:0] | opcode |
|  [31-25]  | [24-20]| [19-15]| [14-12]|  [11-7]  | [6-0]  |
|  7 bits   | 5 bits| 5 bits| 3 bits |  5 bits  | 7 bits |
```
- **Used for**: Store operations to memory
- **Examples**: SW, SB, SH

### 4. B-Type (Branch Operations)
```
|imm[12|10:5]|  rs2  |  rs1  | funct3 |imm[4:1|11]| opcode |
|   [31-25]  | [24-20]| [19-15]| [14-12]|  [11-7]   | [6-0]  |
|   7 bits   | 5 bits| 5 bits| 3 bits |  5 bits   | 7 bits |
```
- **Used for**: Conditional branch operations
- **Examples**: BEQ, BNE, BLT, BGE

### 5. U-Type (Upper Immediate)
```
|           imm[31:12]            |   rd   | opcode |
|            [31-12]              | [11-7] | [6-0]  |
|            20 bits              | 5 bits | 7 bits |
```
- **Used for**: Loading upper immediate values
- **Examples**: LUI, AUIPC

### 6. J-Type (Jump Operations)
```
|    imm[20|10:1|11|19:12]        |   rd   | opcode |
|            [31-12]              | [11-7] | [6-0]  |
|            20 bits              | 5 bits | 7 bits |
```
- **Used for**: Unconditional jump operations
- **Examples**: JAL

## Instruction Set Overview

### Arithmetic Instructions (R-Type & I-Type)
| Instruction | Format | Operation | Description |
|------------|--------|-----------|-------------|
| ADD rd, rs1, rs2 | R-Type | rd = rs1 + rs2 | Add registers |
| SUB rd, rs1, rs2 | R-Type | rd = rs1 - rs2 | Subtract registers |
| ADDI rd, rs1, imm | I-Type | rd = rs1 + imm | Add immediate |

### Logical Instructions (R-Type & I-Type)
| Instruction | Format | Operation | Description |
|------------|--------|-----------|-------------|
| AND rd, rs1, rs2 | R-Type | rd = rs1 & rs2 | Bitwise AND |
| OR rd, rs1, rs2 | R-Type | rd = rs1 \| rs2 | Bitwise OR |
| XOR rd, rs1, rs2 | R-Type | rd = rs1 ^ rs2 | Bitwise XOR |
| ANDI rd, rs1, imm | I-Type | rd = rs1 & imm | AND immediate |
| ORI rd, rs1, imm | I-Type | rd = rs1 \| imm | OR immediate |
| XORI rd, rs1, imm | I-Type | rd = rs1 ^ imm | XOR immediate |

### Shift Instructions (R-Type & I-Type)
| Instruction | Format | Operation | Description |
|------------|--------|-----------|-------------|
| SLL rd, rs1, rs2 | R-Type | rd = rs1 << rs2 | Shift left logical |
| SRL rd, rs1, rs2 | R-Type | rd = rs1 >> rs2 | Shift right logical |
| SRA rd, rs1, rs2 | R-Type | rd = rs1 >>> rs2 | Shift right arithmetic |
| SLLI rd, rs1, imm | I-Type | rd = rs1 << imm | Shift left logical immediate |
| SRLI rd, rs1, imm | I-Type | rd = rs1 >> imm | Shift right logical immediate |
| SRAI rd, rs1, imm | I-Type | rd = rs1 >>> imm | Shift right arithmetic immediate |

### Comparison Instructions (R-Type & I-Type)
| Instruction | Format | Operation | Description |
|------------|--------|-----------|-------------|
| SLT rd, rs1, rs2 | R-Type | rd = (rs1 < rs2) ? 1 : 0 | Set less than (signed) |
| SLTU rd, rs1, rs2 | R-Type | rd = (rs1 < rs2) ? 1 : 0 | Set less than (unsigned) |
| SLTI rd, rs1, imm | I-Type | rd = (rs1 < imm) ? 1 : 0 | Set less than immediate |
| SLTIU rd, rs1, imm | I-Type | rd = (rs1 < imm) ? 1 : 0 | Set less than immediate (unsigned) |

### Load Instructions (I-Type)
| Instruction | Format | Operation | Description |
|------------|--------|-----------|-------------|
| LW rd, offset(rs1) | I-Type | rd = M[rs1 + offset] | Load word (32 bits) |
| LH rd, offset(rs1) | I-Type | rd = M[rs1 + offset] | Load halfword (16 bits) |
| LB rd, offset(rs1) | I-Type | rd = M[rs1 + offset] | Load byte (8 bits) |
| LHU rd, offset(rs1) | I-Type | rd = M[rs1 + offset] | Load halfword unsigned |
| LBU rd, offset(rs1) | I-Type | rd = M[rs1 + offset] | Load byte unsigned |

### Store Instructions (S-Type)
| Instruction | Format | Operation | Description |
|------------|--------|-----------|-------------|
| SW rs2, offset(rs1) | S-Type | M[rs1 + offset] = rs2 | Store word (32 bits) |
| SH rs2, offset(rs1) | S-Type | M[rs1 + offset] = rs2 | Store halfword (16 bits) |
| SB rs2, offset(rs1) | S-Type | M[rs1 + offset] = rs2 | Store byte (8 bits) |

### Branch Instructions (B-Type)
| Instruction | Format | Operation | Description |
|------------|--------|-----------|-------------|
| BEQ rs1, rs2, offset | B-Type | if(rs1 == rs2) PC += offset | Branch if equal |
| BNE rs1, rs2, offset | B-Type | if(rs1 != rs2) PC += offset | Branch if not equal |
| BLT rs1, rs2, offset | B-Type | if(rs1 < rs2) PC += offset | Branch if less than |
| BGE rs1, rs2, offset | B-Type | if(rs1 >= rs2) PC += offset | Branch if greater/equal |
| BLTU rs1, rs2, offset | B-Type | if(rs1 < rs2) PC += offset | Branch if less than (unsigned) |
| BGEU rs1, rs2, offset | B-Type | if(rs1 >= rs2) PC += offset | Branch if greater/equal (unsigned) |

### Jump Instructions (J-Type & I-Type)
| Instruction | Format | Operation | Description |
|------------|--------|-----------|-------------|
| JAL rd, offset | J-Type | rd = PC + 4; PC += offset | Jump and link |
| JALR rd, rs1, offset | I-Type | rd = PC + 4; PC = rs1 + offset | Jump and link register |

### Upper Immediate Instructions (U-Type)
| Instruction | Format | Operation | Description |
|------------|--------|-----------|-------------|
| LUI rd, imm | U-Type | rd = imm << 12 | Load upper immediate |
| AUIPC rd, imm | U-Type | rd = PC + (imm << 12) | Add upper immediate to PC |

## Datapath Components

### 1. Program Counter (PC)
- Holds the address of the current instruction
- Increments by 4 (instruction size) for sequential execution
- Updated by branch/jump instructions

### 2. Instruction Memory
- Stores program instructions
- Read-only in single-cycle design
- Accessed using PC as address

### 3. Register File
- Contains 32 registers (x0-x31)
- x0 is always zero
- Two read ports and one write port
- Supports simultaneous read and write operations

### 4. Arithmetic Logic Unit (ALU)
- Performs arithmetic operations (ADD, SUB)
- Performs logical operations (AND, OR, XOR)
- Performs comparison operations (SLT, SLTU)
- Performs shift operations (SLL, SRL, SRA)
- Generates condition flags (Zero, Negative, etc.)

### 5. Data Memory
- Stores program data
- Supports read (load) and write (store) operations
- Accessed using calculated address from ALU

### 6. Immediate Generator
- Extracts and sign-extends immediate values
- Different extraction logic for each instruction format
- Produces 32-bit immediate value from instruction bits

### 7. Multiplexers
- Select appropriate data sources
- Control signal driven
- Used for:
  - ALU input selection (register or immediate)
  - Write-back data selection (ALU result or memory data)
  - Next PC selection (PC+4, branch target, or jump target)

## Control Unit

The control unit generates control signals based on the opcode and function fields:

### Main Control Signals:
| Signal | Description |
|--------|-------------|
| RegWrite | Enable writing to register file |
| MemRead | Enable reading from data memory |
| MemWrite | Enable writing to data memory |
| MemtoReg | Select data memory output for write-back |
| ALUSrc | Select immediate value as ALU input |
| ALUOp | Specify ALU operation type |
| Branch | Enable branch operation |
| Jump | Enable jump operation |

### ALU Control:
Based on ALUOp and funct fields, generates specific ALU operation:
- 0000: AND
- 0001: OR
- 0010: ADD
- 0110: SUB
- 0111: SLT
- 1100: NOR

## Instruction Examples

### Example 1: ADD instruction
```assembly
ADD x3, x1, x2  # x3 = x1 + x2
```
**Execution steps:**
1. Fetch instruction from memory
2. Decode: Identify R-type, read x1 and x2
3. Execute: ALU adds values from x1 and x2
4. Memory: No memory access
5. Write-back: Store result in x3

### Example 2: ADDI instruction
```assembly
ADDI x3, x1, 10  # x3 = x1 + 10
```
**Execution steps:**
1. Fetch instruction from memory
2. Decode: Identify I-type, read x1, extract immediate (10)
3. Execute: ALU adds x1 value and immediate (10)
4. Memory: No memory access
5. Write-back: Store result in x3

### Example 3: LW instruction
```assembly
LW x3, 8(x1)  # x3 = Memory[x1 + 8]
```
**Execution steps:**
1. Fetch instruction from memory
2. Decode: Identify I-type load, read x1, extract offset (8)
3. Execute: ALU calculates address (x1 + 8)
4. Memory: Read data from calculated address
5. Write-back: Store loaded data in x3

### Example 4: SW instruction
```assembly
SW x2, 8(x1)  # Memory[x1 + 8] = x2
```
**Execution steps:**
1. Fetch instruction from memory
2. Decode: Identify S-type, read x1 and x2, extract offset (8)
3. Execute: ALU calculates address (x1 + 8)
4. Memory: Write x2 data to calculated address
5. Write-back: No write-back to register

### Example 5: BEQ instruction
```assembly
BEQ x1, x2, 16  # if (x1 == x2) PC = PC + 16
```
**Execution steps:**
1. Fetch instruction from memory
2. Decode: Identify B-type, read x1 and x2, extract offset (16)
3. Execute: ALU compares x1 and x2
4. Memory: No memory access
5. Write-back: Update PC based on comparison result

### Example 6: JAL instruction
```assembly
JAL x1, 100  # x1 = PC + 4; PC = PC + 100
```
**Execution steps:**
1. Fetch instruction from memory
2. Decode: Identify J-type, extract offset (100)
3. Execute: Calculate return address (PC + 4) and jump target (PC + 100)
4. Memory: No memory access
5. Write-back: Store return address in x1, update PC to jump target

## Implementation Notes

This 8-bit implementation is a simplified version of the standard 32-bit RISC-V architecture:
- **Data width**: 8 bits instead of 32 bits
- **Instruction width**: May be adapted to fit the 8-bit design
- **Register count**: May be reduced from 32 registers
- **Instruction subset**: Implements a basic subset of RISC-V instructions

### Design Considerations:
1. **Clock Period**: Must accommodate the slowest instruction (typically load)
2. **Critical Path**: Usually includes instruction fetch → decode → ALU → memory → write-back
3. **Resource Usage**: Simple but may be inefficient in hardware utilization
4. **Extensibility**: Easy to add new instructions or features

## References

- [RISC-V Specifications](https://riscv.org/technical/specifications/)
- [RISC-V ISA Manual](https://github.com/riscv/riscv-isa-manual)
- Computer Organization and Design: The Hardware/Software Interface (RISC-V Edition) by Patterson and Hennessy

## License

This project is for educational purposes as part of the CO224 course.
