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

`R[n]` defines the `n`th GPR.

`S[n]` defines the `n`th System Register.

`M[n]` defines the memory address at address `n`.

`In` defines a sign-extended immediate value of size `n`.

## Instruction Definitions

There are 3 instruction formats: 2 operand, 1 operand, and 0 operand. 

2 operand instructions usually use two GPRs as operands.
The exception are the instructions that move data to and from the system registers.

1 operand instructions use one GPR as an operand, and may have immediate operands too.

0 operand instructions do not use any registers as operands, but may have immediate operands.










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