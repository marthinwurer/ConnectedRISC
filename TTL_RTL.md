# TTL CPU RTL

## Introduction

This file describes the basic Register-Transfer Language (RTL) specification of the Computer Architecture Testbed (CAT) CPU.

CAT is a stored-program load-store 8-bit CPU with a 16 bit address space.
It has 4 general-purpose registers and 4 system registers, all of which are 16 bits.
Instructions are a mix of 2, 1, and 0 operand, and are a fixed length of one byte (8 bits).
There are three addressing modes, but each of them are instruction-specific.
These are register, 16 bit immediate, 8 bit immediate, and Stack-Pointer relative addressing.

## Register Definitions

The CAT has 8 16-bit registers.
4 of these registers are general-purpose registers (GPRs), and are used in most instructions.
The other 4 are system registers (SRs).
The four system registers are the Program Counter (PC), the Stack Pointer (SP), the Status Register, and a reserved register.

## Terminology

The architectural specification will use RTL to define instruction operation.

`B <- A` defines the assignment of the value of `A` to `B`.

`R[n]` defines the value in the `n`th GPR.

`S[n]` defines the value in the `n`th System Register.

`M[n]` defines the value of the byte at the memory address `n`.

`In` defines a sign-extended immediate value of size `n` bytes.

`PC`, `SP`, and `SR` denote the Program counter, the stack pointer, and the status register, respectively.

The status register also has four fields.
These are `Z`ero, `N`egative, `C`arry, and `O`verflow.
They are set after every instruction that would use the ALU.
In-depth definitions will note if these are set. 


## Instruction Definitions

There are 3 instruction formats: 2 operand, 1 operand, and 0 operand. 

2 operand instructions usually use two GPRs as operands.
The exception are the instructions that move data to and from the system registers.

The two operand instructions are:
- Add
- Subtract
- And
- Or
- Xor
- Compare
- Shift Left Logical
- Shift Right Logical
- Move to system register
- Move from system register
- Move
- Load Byte
- Store Byte

1 operand instructions use one GPR as an operand, and may have immediate operands too.

The one-operand instructions are:
- Load Immediate
- Sign Extend
- Jump and link register
- Shift Right Arithmetic once
- Push register
- Pop register
- Stack-relative load
- Stack-relative store
- Byte Swap (Desired, might have to make some room) 

0 operand instructions do not use any registers as operands, but may have immediate operands.

The zero-operand instructions are:
- Noop
- Halt
- Syscall
- Return
- Enable interrupts
- Disable interrupts
- Branch equal/zero
- Branch not equal/nonzero
- Branch if carry
- Branch if overflow
- Branch less than
- Branch greater than or equal
- Branch less than unsigned
- Branch greater than or equal unsigned

### In depth definitions

Each instruction definition consists of a brief description of the instruction,
followed by the assembly representation of the instruction and its RTL definition.

If the status register is set after instruction execution, the writeup will say so.

Unless otherwise noted, each instruction also executes `PC <- PC + 1`.
This increments the `PC` to point to the next instruction.

#### Add

Adds the values in `rd` and `rs` together and stores the result in `rd`.

`add rd rs`

RTL: `Rd <- Rd + Rs`

Status register is set.

#### Subtract

Subtracts the value in `rs` from `rd` and stores the result in `rd`.

`sub rd rs`

RTL: `Rd <- Rd - Rs`

Status register is set.

#### And

Ands the values in `rd` and `rs` together and stores the result in `rd`.

`and rd rs`

RTL: `Rd <- Rd & Rs`

Status register is set.

#### Or

Ors the values in `rd` and `rs` together and stores the result in `rd`.

`or rd rs`

RTL: `Rd <- Rd | Rs`

Status register is set.

#### Xor

Xors the values in `rd` and `rs` together and stores the result in `rd`.

`xor rd rs`

RTL: `Rd <- Rd ^ Rs`

Status register is set.

#### Compare

Subtracts the value in `rs` from `rd` but does not store the result.
The status register is still set.

`cmp rd rs`

RTL: `Rd - Rs`

Status register is set.

#### Shift Left Logical

Shift the value in `rd` left by `rs` places.

`sll rd rs`

RTL: `Rd <- Rd << Rs`

Status register is set.

#### Shift Right Logical

Shift the value in `rd` right by `rs` places.

`srl rd rs`

RTL: `Rd <- Rd >> Rs`

Status register is set.

#### Move to system register

Set the value in the system register `sd` to the value of the general purpose register `rs`.

`mts sd rs`

RTL: `S[sd] <- R[rs]`

#### Move from system register

Set the value in the general purpose register `rd` to the value of the system register `ss`.

`mtr rd ss`

RTL: `R[rd] <- S[ss]`

#### Move

Set the value in `rd` to the value of `rs`.

`mov rd rs`

RTL: `R[rd] <- R[rs]`

#### Load Byte

Set the lower byte of `rd` to the value in memory at the address stored in the register `rs`.

`ldb rd as`

RTL: `R[rd] <- M[rs]`

#### Store Byte

Set the byte in memory at the address stored in register of `rs` to the value of the lower byte in `rd`.

`stb rs ad`

RTL: `M[rs] <- R[rd][7:0]`

#### Load Immediate

Set the register `rd` to the value in the next two instruction words, then increment `PC` by 3.

`ldi rd I16`

RTL:
```
PC <- PC + 1
R[rd][15:8] <- M[PC]
PC <- PC + 1
R[rd][7:0] <- M[PC]
PC <- PC + 1
```

#### Sign Extend

Set the upper bits of `rd` to the value of the 7th bit of `rd`.

`sxt rd`

RTL: `R[rd][15:8] <- R[rd][7]`

#### Jump and link register

Increment the `PC` like usual, then swap it with `rd`.

`jlr rd`

RTL: `PC <- R[rd] AND R[rd] <- PC + 1`

#### Shift Right Arithmetic once

Shift `rd` to the right by one position, with the highest bit staying the same as it was before.

`sra rd`

RTL: `R[rd] <- R[rd] >>> 1`

Status register is set.

#### Push register

Push a register's value onto the stack.

`push rd`

RTL:
```
SP <- SP - 1
M[SP] <- R[rd][15:8]
SP <- SP - 1
M[SP] <- R[rd][7:0]
```

#### Pop register
Pop a register's value from the stack.

`pop rd`

RTL:
```
R[rd][7:0] <- M[SP]
SP <- SP + 1
R[rd][15:8] <- M[SP]
SP <- SP + 1
```

#### Load Stack-Relative

Set the lower byte of `rd` to the value in memory offset from `SP` by the 8-bit sign-extended immediate value.

`lsr rd I8`

RTL: `R[rd][7:0] <- M[SP + I8]`

#### Store Stack-Relative

Set the value in memory offset from `SP` by the 8-bit sign-extended immediate value to the lower byte in `rs`.

`ssr rs I8`

RTL: `M[SP + I8] <- R[rs][7:0]`

#### Byte Swap (Desired, might have to make some room) 

Swap the high and low bytes of `rd`.

`bs rd`

RTL: `R[rd][15:8] <- R[rd][7:0] AND R[rd][7:0] <- R[rd][15:8]`

#### Noop

No op. Just increment the program counter.

`nop`

RTL: `PC <- PC + 1`

#### Halt

Halt execution until processor is restarted or reset.

`hlt`

RTL: `Stop processor`

#### Syscall

Throw a syscall interrupt. TODO with memory management.

#### Return

Return back to user space. TODO with memory management.

#### Enable interrupts

Enable interrupts if in system mode. TODO with memory management.

#### Disable interrupts

Disable interrupts if in system mode. TODO with memory management.

#### Branch equal/zero

Set the `PC` to a sign-extended immediate offset of `PC` if the result of the last operation set the zero flag. 

`beq I8`

RTL: `PC <- PC + I8 if Z = 1 else PC + 1`

#### Branch not equal/nonzero

Set the `PC` to a sign-extended immediate offset of `PC` if the result of the last operation did not set the zero flag. 

`bne I8`

RTL: `PC <- PC + I8 if Z = 0 else PC + 1`

#### Branch if carry

Set the `PC` to a sign-extended immediate offset of `PC` if the result of the last operation set the carry flag. 

`bic I8`

RTL: `PC <- PC + I8 if C = 1 else PC + 1`

#### Branch if overflow

Set the `PC` to a sign-extended immediate offset of `PC` if the result of the last operation set the overflow flag. 

`bio I8`

RTL: `PC <- PC + I8 if O = 1 else PC + 1`

#### Branch less than

Set the `PC` to a sign-extended immediate offset of `PC` if the `O` and `N` flags are not equal.

`blt I8`

RTL: `PC <- PC + I8 if N != O else PC + 1`

#### Branch greater than or equal

Set the `PC` to a sign-extended immediate offset of `PC` if the result of the last operation did not set the negative flag. 

`bge I8`

RTL: `PC <- PC + I8 if N = 0 else PC + 1`

#### Branch less than unsigned

Set the `PC` to a sign-extended immediate offset of `PC` if the result of the last operation did not set the carry flag. 

`blu I8`

RTL: `PC <- PC + I8 if N = 1 else PC + 1`

#### Branch greater than or equal unsigned

Set the `PC` to a sign-extended immediate offset of `PC` if the result of the last operation set the carry flag. 

`bgu I8`

RTL: `PC <- PC + I8 if N = 1 else PC + 1`




## Original notes

Basic architecture description:

Stored program load-store architecture.
8 bit data bus, 16 bit address bus.
4 GPRs, n banks of 3 system registers.
Status register for branching:
Zero
Carry
Sign
Overflow

Multi-cycle machine, target speed 2 MHz.

3 instruction formats:
2 op
1 op
0 op

4 GPRs as parameters to these.
3 user space system registers:
PC
SP
Status

Instructions:

2 op: 16 - 3 = 13 slots
Add
Subtract
Shift left logical
Shift right

And
Or
Xor
Compare

Load byte
Store byte
Move to system register
Move from system register

Move between registers

1 op: 12 - 4 = 8 slots
Load immediate (next two bytes load into register)
Sign extend
Jump and link register (swap pc + 1 with register value)
Shift left arithmetic once (saves an opcode)

Push register
Pop register
Load sp relative
Store sp relative

0 op: 16 slots
Syscall
Return (return to user space from syscall)
Noop
Halt (map to 0x00 so unmapped memory halts processor)

Branch equal/zero
Branch not equal/nonzero
Branch if carry
Branch if overflow

Branch less than
Branch greater than or equal
Branch less than unsigned
Branch greater than or equal unsigned

Enable interrupts
Disable interrupts
One for expansion if needed. (Might want mmu stuff)

Design considerations:
Want only one ALU
8 bit data bus to memory
16 bit address bus

Want to hook up a few basic peripherals:
UART
Paper tape reader/punch
Some kind of persistent storage (nvram? Eeprom?)
Custom vga terminal

Might do s100 bus for that, do port mapping and all that jazz. Definitely use the s100 bus.

Make a basic package manager?

External start signal sets pc to a set value, which should be the start of bios ROM

External halt signal sets 