# Mercury Instruction Set Architecture

Mercury is a simple homebrew ISA specification inspired by 80s-90s processors.

### Summary
|              |                                                                                                    |
|-------------:|----------------------------------------------------------------------------------------------------|
|     **TYPE** | load-store                                                                                         |
|   **ENDIAN** | little-endian                                                                                      |
|     **WORD** | 16-bits                                                                                            |
|  **ADDRESS** | 32-bits, word-addressable, up to 2<sup>33</sup> bytes, 2<sup>32</sup> words.                       |
|    **STACK** | empty-stack convention, grows up through memory, configurable, 64 KiW wraparound                   |
|    **START** | starts at address `0x10_0000` on start and [`RST`](instructions.md#rst), state must be initialised |
| **REGISTER** | 16 word-size registers (6 general purpose, 6 special, 4 restricted)                                |

- Registers (except `M`, `L` and `Z`) should be assumed uninitialised, including the stack pointer, and should be set by the reset routine at `0x10_0000`
- Registers can pair together. The index of the low register is stored, and the index of the high register is obtained by flipping the lowest bit (adjacent register)
- Stack operations only increment/decrement the low register; the stack wraps at 64 KiW
- `SP` is the default stack pointer, but stack operations can use a separate pointer if needed.
    - ***but*** `INT`, which pushes state to the stack during interrupts, will always use `SP` as its stack pointer

### Units, Word Size and Addressability
As Mercury is a word-addressable architecture and not a byte-addressable one, the actual data covered is twice as large as what one might interpret if using byte-sized units.<br>
Each location in memory is ***16-bits*** wide, and the entire processor normally operates on units of at least 16 bits.

This document (and other related ones) will use the following units:
|   Name   |     Unit     | Size (Words)                     | Size (Bytes)                    |
|:--------:|:------------:|----------------------------------|---------------------------------|
|   bit    |     `b`      | <sup>1</sup>&frasl;<sub>16</sub> | <sup>1</sup>&frasl;<sub>8</sub> |
|  nibble  |     N/A      | <sup>1</sup>&frasl;<sub>4</sub>  | <sup>1</sup>&frasl;<sub>2</sub> |
|   byte   |     `B`      | <sup>1</sup>&frasl;<sub>2</sub>  | 1                               |
|   word   |     `W`      | 1                                | 2                               |
| kibiword |    `KiW`     | 2<sup>10</sup>                   | 2<sup>11</sup>                  |
| mebiword |    `MiW`     | 2<sup>20</sup>                   | 2<sup>21</sup>                  |
| gibiword |    `GiW`     | 2<sup>30</sup>                   | 2<sup>31</sup>                  |
| kiloword | `kW`<br>`KW` | 10<sup>3</sup>                   | 2 · 10<sup>3</sup>              |
| megaword |     `MW`     | 10<sup>6</sup>                   | 2 · 10<sup>6</sup>              |
| gigaword |     `GW`     | 10<sup>9</sup>                   | 2 · 10<sup>9</sup>              |

### Notation
For this document, Nym's notation is used:
- Immediate values have no special prefix
- Binary values are prefixed by `0b`, hexadecimals by `0x`, decimals by nothing
- Registers are prefixed by `%` (e.g. `%A`, `%reg16`) in code
- Memory addressing is expressed with `[...]` (e.g. `[%ML + 6]`) in code
- Intel ordering (`op dst, src`) is used in code (e.g. `mov %A, [%SP - 1]` -> "load the latest word on stack to register `A`")
    - In ALU operations with two operands, the destination register is operand 1, and the source register is operand 2 (e.g. `add %A, %B` -> `A = A + B`)
- `SP` is the default stack pointer for stack instructions, but a different pointer can be specified on the instruction via a dot (e.g. `pop`=`pop.sp`, `pop.uv`)
- Existence of the [`INTV`](instructions.md#intv) pseudo-instruction

## Table of Contents

- [Description](#description)
    - [Initialisation](#initialisation)
    - [Stack](#stack)
    - [Immediates and Endianness](#immediates--endianness)
    - [Registers](#registers)
        - [Register Map](#register-map)
        - [Restricted Registers](#restricted-registers)
            - [Flag Register](#flag-register)
    - [Memory Addressing](#memory-addressing)
    - [Instruction Encoding](#instruction-encoding)
        - [Jump Instruction Encoding](#jump-instruction-encoding)
    - [Interrupts](#interrupts)
        - [Interrupt Rules](#interrupt-rules)
        - [Software Interrupts](#software-interrupts)
        - [Hardware Interrupts](#hardware-interrupts)
            - [Trap Enable Flag](#trap-enable-flag)
    - [Fixed Memory Map](#fixed-memory-map)
- [Example Execution Pipeline](#example-execution-timeline)

## Description

### Initialisation
At startup, the processor executes [0000 (`RST`)](instructions.md#rst).<br>
The program counter (`PC`) is reset to `0x10_0000`, and therefore any startup and reset code ***must*** be written to address `0x10_0000`.<br>
The programmer ***must*** assume that all registers apart from `M`, `L`, and `Z` are uninitialised upon startup, unless otherwise specified by an implementation, and therefore the reset routine should initialise them.

### Stack
The stack grows up through memory space, following the empty-stack convention:<br>
- when pushed, the stack pointer *increments*, and when popped, *decrements*
- the stack pointer points to the first available space

As the stack pointer is just a regular 32-bit register pair, it must be manually set to the desired stack start address during the reset routine.

The default is the register pair `SP` (index 1011), though stack instructions can use any pair. Beware using [restricted registers](#restricted-registers) if you wish to use a separate stack address, as they will lose data.<br>
***IMPORTANT:*** The INT instruction will *always* use `SP` as its stack pointer.

Only the lower word of the register pair (e.g. `P` for `SP`) is incremented or decremented during a stack operation. Therefore, the stack wraps around and has a size of 64 KiW.

### Immediates & Endianness
Immediate values are 16- or 32-bits, stored in little-endian format for the latter.

|  Type   | Literal Value | Memory Representation |
|:-------:|---------------|-----------------------|
| `imm16` | `0xCAFE`      | `00: CAFE`            |
| `imm32` | `0xCAFEBEEF`  | `00: BEEF 01: CAFE`   |

### Registers

There are 16 16-bit registers which can be paired up with their adjacent register to form 32-bit register pairs.<br>
The adjacent (high) register index can be found by flipping the lowest bit (XOR 0b0001) of the base (low) register.<br>
The second letter of a pair is the ***low*** register, and it is used as the base index (e.g. `P` in `SP`).

General-purpose registers (GPRs) are free for use unless specified otherwise by an implementation.<br>
Special registers (SRs) have recommended usages below, but can operate like GPRs.<br>
[Restricted registers (RRs)](#restricted-registers) have limits imposed on them, explained [below](#restricted-registers).

- 6 16-bit GPR (`A-B`, `C-D`, `E-G`)
    - Indices 0000 through 0101
- The 32-bit instruction start pointer (`L-M`)
    - Indices 0110 and 0111 (`L` is the low word, `M` is the high word)
- A 32-bit stack base pointer (`X-Y`)
    - Indices 1000 and 1001
- A 32-bit stack pointer (`S-P`)
    - Indices 1010 and 1011
- A 32-bit OS-reserved register (`U-V`)
    - Indices 1100 and 1101
- A 16-bit [flag register](#flag-register) (`F`)
    - Index 1110
- A 16-bit zero read-only register (`Z`)
    - Index 1111

#### Register Map

|                 |      <kbd>--00</kbd>       |      <kbd>--01</kbd>       |      <kbd>--10</kbd>       |      <kbd>--11</kbd>       |
|:---------------:|:--------------------------:|:--------------------------:|:--------------------------:|:--------------------------:|
| <kbd>00--</kbd> | `A`<br>`BA`<br>0000<br>GPR | `B`<br>`AB`<br>0001<br>GPR | `C`<br>`DC`<br>0010<br>GPR | `D`<br>`CD`<br>0011<br>GPR |
| <kbd>01--</kbd> | `E`<br>`GE`<br>0100<br>GPR | `G`<br>`EG`<br>0101<br>GPR | `L`<br>`ML`<br>0110<br>RR  | `M`<br>`LM`<br>0111<br>RR  |
| <kbd>10--</kbd> | `X`<br>`YX`<br>1000<br>SR  | `Y`<br>`XY`<br>1001<br>SR  | `S`<br>`PS`<br>1010<br>SR  | `P`<br>`SP`<br>1011<br>SR  |
| <kbd>11--</kbd> | `U`<br>`VU`<br>1100<br>SR  | `V`<br>`UV`<br>1101<br>SR  | `F`<br>`ZF`<br>1110<br>RR  | `Z`<br>`FZ`<br>1111<br>RR  |

---
*Key for the above:*
```
16-bit register name
32-bit register name
index
register type
```
---

#### Restricted Registers
- The `ML` register is volatile (changes at every instruction), read-only and refers to the start of the current instruction in memory (= `PC` before fetching instruction).
- The zero register `Z` discards any writes and always outputs 0.
- The flag register `F` stores flags in some bits and may not record other bits.

##### Flag Register
```
15.....8 7......0
-------- NCOI-TSZ
```
|    Flag     |   `N`    |  `C`  |   `O`    |       `I`        |     `T`     |           `S`            | `Z`  |
|:-----------:|:--------:|:-----:|:--------:|:----------------:|:-----------:|:------------------------:|:----:|
| **Meaning** | Negative | Carry | Overflow | Interrupt Enable | Trap Enable | Signal<br>(user-defined) | Zero |

Undefined bits ***are not reserved*** and may be used for other purposes by different implementations.

### Memory Addressing
Memory instructions use effective addresses as their source or destination. These are calculated by adding a register and an optional immediate.

The register can be a single 16-bit register (`%reg16`) or a 32-bit register pair (`%reg32`).<br>
As for the immediate value, it may be absent, 16-bit, or 32-bit.

This gives a total of six possibilities per operand:
- `%reg16`
- `%reg32`
- `%reg16+imm16`
- `%reg32+imm16`
- `%reg16+imm32`
- `%reg32+imm32`

The zero register `Z` is intended to be used when no register offset is desired, and may be assumed as the default in assemblers when no register is specified.

16-bit values are sign-extended to 32-bits to allow proper offsetting.

Due to size limitations, the `mvm(l) mem, mem` operation ("move (long) from memory to memory") only supports two addressing modes:
- `EA = %reg16+imm32`
- `EA = %reg32+imm32`

With these two modes, all other modes can be simulated, at the cost of space.

### Instruction Encoding
Instruction length: between 1 and 5 words

The first word contains an 8-bit opcode and two 4-bit indices for registers.
```
 HI              LO
+--------+----+----+
| OPCODE |SRC |DST |
+--------+----+----+
```

Immediate values are stored in little-endian format after the first word.<br>
If an immediate is present for both the source and destination (i.e. in `mvm(l) mem, mem`),
the source immediate is stored before the destination immediate.
```
+--------+----+----+ +---------------+ +---------------+
| OPCODE |SRC |DST | | SRC IMM (opt) | | DST IMM (opt) |
+--------+----+----+ +---------------+ +---------------+
```

#### Jump Instruction Encoding
For jump instructions, the SRC bits of the word represent a condition code:
|  #   |  Mnemonic  | Checked Flag Condition(s) | Meaning                                           |
|:----:|:----------:|:-------------------------:|---------------------------------------------------|
| 0000 |  JL<br>JN  |            `N`            | Jump if less than<br>Jump if negative             |
| 0001 | JGE<br>JP  |           `~N`            | Jump if greater than or equal<br>Jump if positive |
| 0010 | JEQ<br>JZ  |            `Z`            | Jump if equal<br>Jump if zero                     |
| 0011 | JNE<br>JNZ |           `~Z`            | Jump if not equal<br>Jump if not zero             |
| 0100 |     JV     |            `O`            | Jump if overflow                                  |
| 0101 |    JNV     |           `~O`            | Jump if not overflow                              |
| 0110 |     JC     |            `C`            | Jump if carry                                     |
| 0111 |    JNC     |           `~C`            | Jump if not carry                                 |
| 1000 |    JLE     |          `N\|Z`           | Jump if less than or equal                        |
| 1001 |     JG     |         `~(N\|Z)`         | Jump if greater than                              |
| 1010 |     JI     |            `I`            | Jump if interrupt enabled                         |
| 1011 |    JNI     |           `~I`            | Jump if interrupt disabled                        |
| 1100 |     JS     |            `S`            | Jump if signal                                    |
| 1101 |    JNS     |           `~S`            | Jump if not signal                                |
| 1110 |    JMP     |          Always           | Unconditional jump                                |
| 1111 |    JNR     |           Never           | Never jump, no-op, unused                         |

This gives the first word of a Jcc instruction the format:
```
+--------+----+----+
|JUMP    |COND|REG |
+--------+----+----+
```

### Interrupts
Hardware and software interrupts are both managed via the ***interrupt vector table (IVT)***.

|           |                                                                                                    |
|----------:|----------------------------------------------------------------------------------------------------|
|  **SIZE** | 1 KiW, 512 x 2 W addresses                                                                         |
| **RANGE** | `0x0002_0000-0x0002_01FF` (hardware interrupts)<br>`0x0002_0200-0x0002_03FF` (software interrupts) |

This table is split into two halves, one for ***hardware interrupts (vectors 000-0FF)*** and one for ***software interrupts (vectors 100-1FF)***.<br>
Each entry is the address of the interrupt routine handling the corresponding vector.

#### Interrupt Rules
When multiple types of interrupts are pending, the following order of precedence is followed, from most to least important:
1. Non-Maskable Hardware Interrupt (NMI)
2. Trap Non-Maskable Interrupt
3. Maskable Hardware Interrupts (if interrupt enabled)

Additionally, certain rules must be implementated (except the first implementation of Mercury):
- ***NMI handling must be atomic;*** a running NMI handler cannot be interrupted (other than by a reset), including by another NMI, and even if the `T` and `I` flags are enabled.<br>
    This latch should be lifted by [`RTI`](instructions.md#rti), allowing unhandled NMIs to run back to back before resuming normal operation.<sup>*</sup>
- ***NMIs can indefinitely delay all other interrupts;*** no trap or maskable interrupt or regular instruction is allowed to execute as long as an NMI is *pending* or *active*.
- ***The CPU must latch any trigger of the NMI line and register it as a pending NMI request;*** even if the CPU is currently busy with another NMI or if an NMI trigger is cancelled, the CPU must latch the trigger and call the NMI handler at the next available opportunity.
- ***Once an instruction begins with the Trap Enable (`T`) flag set, the trap request is pending and must be handled before the next non-NMI instruction can begin;*** a latch should be set when an instruction begins in a trapped state, and it may only be lifted once the trap handler is called.
- ***An NMI routine may schedule a trap for the original instruction it interrupted by setting the `T` flag before returning;*** as a trap cannot interrupt an NMI handler, the trap latch will remain set until after the [`RTI`](instructions.md#rti) clears the NMI state, after which the trap handler will immediately run, allowing the trap handler to run as if it executed right after the original instruction.
- ***A trap handler is allowed to set the `T` flag and trigger another trap within itself;*** in such a case, the trap handler must be designed to be reentrant.
- ***A pending trap takes priority over pending maskable interrupts, but not NMIs.***
- ***A trap or maskable interrupt handler is allowed to set the `I` flag to re-enable maskable interrupts while it runs;*** if a maskable interrupt occurs, the handler will be paused while the nested handler executes.<br>
Maskable interrupt handlers who use this behaviour must be designed to be reentrant.

\* ***NOTE:** If an NMI handler calls a software interrupt or any code which runs [`RTI`](instructions.md#rti), NMI atomicity privilege will be lost. Conversely, any code called from a privileged NMI will itself maintain atomicity until the first [`RTI`](instructions.md#rti)*

For the ***first implementation***, the rules are as follows:
- ***Competing hardware interrupts on the same instruction are handled by order of precedence.***
- ***NMIs are not atomic;*** just like trap and maskable interrupts, NMIs don't have special status and can be interrupted.<br>
The programmer must not reenable the trap flag unless interruption is truly desired.
- ***Any handler will trigger a trap within itself if it set the `T` flag;*** handlers should be written appropriate to avoid or to take into account this behaviour as needed.
- TODO: can a Trap be missed if NMI and Trap are both set?


#### Software interrupts
Software interrupts are triggered by [`INT`](instructions.md#int) with an operand formatted as an ***offset*** into the software half of the table.

During a software interrupt, bits 1-9 of the operand passed to [`INT`](instructions.md#int) will be used as the offset from vector 0x100:
| Instruction... | ...will call vector... | ...using the address at |
|:---------------|:-----------------------|:------------------------|
| `INT 0x000`    | 0x100                  | `0x0002_0200`           |
| `INT 0x002`    | 0x101                  | `0x0002_0202`           |
| ...            | ...                    | ...                     |
| `INT 0x1FC`    | 0x1FE                  | `0x0002_03FC`           |
| `INT 0x1FE`    | 0x1FF                  | `0x0002_03FE`           |

***IMPORTANT:*** the operand must be the 8-bit interrupt vector number shifted left once (a 9-bit operand, one address is 2 words) as the first implementation of Mercury did not mask the bits. An incorrectly formatted operand may render a misaligned address or a value outside of the IVT entirely.

It is strongly recommended that implementations mask the bits going forward.

***NOTE:*** some dialects of Mercury may have the [`INTV`](instructions.md#intv) variant of [`INT`](instructions.md#int).<br>
This is a pseudo-instruction which takes an 8-bit software vector number (soft 0x00 = real 0x100, soft 0xFF = real 0x1FF) and translates it to the equivalent [`INT`](instructions.md#int) instruction.

| INTV        | = INT       | = Vector | = Address     |
|:------------|:------------|:---------|:--------------|
| `INTV 0x00` | `INT 0x000` | 0x100    | `0x0002_0200` |
| `INTV 0x01` | `INT 0x002` | 0x101    | `0x0002_0202` |
| ...         | ...         | ...      | ...           |
| `INTV 0xFE` | `INT 0x1FC` | 0x1FE    | `0x0002_03FC` |
| `INTV 0xFF` | `INT 0x1FE` | 0x1FF    | `0x0002_03FE` |

#### Hardware interrupts
The IVT is followed by the 9-bit hardware interrupt dispatcher (HID) at address `0x0002_0400`, which is used like the [`INT`](instructions.md#int) instruction's 9-bit operand.<br>

Maskable hardware interrupts are triggered by activating the Interrupt line of the CPU, which will then select the vector indicated by the HID. These can be disabled by unsetting the Interrupt Enable (`I`) flag.

Non-maskable interrupt (NMI) can also be triggered via the NMI line of the CPU, if present, which will always trigger vector 0x0FF (`0x0002_01FE`)

See [below](#trap-enable-flag) for the semantics of the Trap interrupts.

The HID is controlled by a programmable interrupt controller (PIC) (if present), which will assign it the appropriate handler's offset into the IVT.<br>
When there is no PIC to control the value of the HID, it ***must*** be 0, which points to the default hardware handler (`0x0002_0200`).

The memory range `0x0002_0400` to `0x0002_04FF` is reserved for the HID and other PIC memory-mapped registers, if present.

|     HID...      | ...will call vector... | ...using the address at   |                                        |
|:---------------:|------------------------|---------------------------|----------------------------------------|
|     `0x000`     | 0x000                  | `0x0002_0000`             | default handler (maskable)             |
| `0x002`<br>Trap | 0x001                  | `0x0002_0002`             | trap NMI handler (maskable if via HID) |
|  `0x004-0x1FC`  | 0x002-0x0FE            | `0x0002_0004-0x0002_01FC` | other maskable interrupt handlers      |
| `0x1FE`<br>NMI  | 0x0FF                  | `0x0002_01FE`             | NMI handler (maskable if via HID)      |

##### Trap Enable Flag
When the Trap Enable (`T`) flag is set, the CPU will (and must) call *hardware interrupt vector 1* (`0x0002_0002`) after every instruction.<br>
This interrupt will be called *regardless* of the status of the Interrupt Enable (`I`) flag, and it will be called as long as the `T` flag was set at the ***beginning*** of the current instruction, even if the current instruction unsets it.<br>

### Fixed Memory Map
Mercury leaves memory mapping open. However, the following locations are hardcoded:

<pre>
IVT................00020000-000203FF.....1 KiW
    Hardware IVT...00020000-000201FF...512   W
    Software IVT...00020200-000203FF...512   W
PIC Reserved.......00020400-000204FF...256   W
    HID.....................00020400.....1   W
    Reserved.......00020401-000204FF...255   W

(Re)start...................00100000......   -
</pre>

## Example Execution Timeline
Here is a proposed execution timeline. Assume three latches:
- *Trap Pending*
- *NMI Pending*
- *NMI Active*

<ol>
<h4>FETCH</h4>
    <li>Set the ML 32-bit register to the current value of PC</li>
    <li>Fetch the next instruction (PC is now at the arguments or the next instruction for an argumentless instruction)</li>
    <li>If the Trap Enable <code>T</code> flag is set, set the <i>Trap Pending</i> latch</li>

<h4>DECODE & EXECUTE</h4>
    <li>Decode and execute the instruction (PC is now at the next instruction to execute)</li>

<h4>HANDLE INTERRUPTS - if <i>NMI Active</i> is unset</h4>
    <li>If <i>NMI Pending</i> is set, handle the NMI:
        <ol type="i">
            <li>Unset the <i>NMI Pending</i> latch</li>
            <li>Set the <i>NMI Active</i> latch</li>
            <li>Push the <b>current value of the PC</b> to the stack</li>
            <li>Push the flags register to the stack and reset the flags register</li>
            <li>Jump to the NMI handler (hardware vector 0xFF)</li>
            <li>Goto to step 1</li>
        </ol>
    </li>
    <li>Otherwise, if <i>Trap Pending</i> is set, handle the trap:
        <ol type="i">
            <li>Unset the <i>Trap Pending</i> latch</li>
            <li>Push the <b>current value of the PC</b> to the stack</li>
            <li>Push the flags register to the stack and reset the flags register</li>
            <li>Jump to the Trap handler (hardware vector 0x01)</li>
            <li>Goto to step 1</li>
        </ol>
    </li>
    <li>Otherwise, if the Interrupt Enable <code>I</code> flag is set and the interrupt pin is triggered, handle the maskable interrupt:
        <ol type="i">
            <li>Push the <b>current value of the PC</b> to the stack</li>
            <li>Push the flags register to the stack and reset the flags register</li>
            <li>Read the interrupt hardware interrupt offset from HID (<code>0x00020400</code>)</li>
            <li>Jump to the corresponding handler</li>
            <li>Goto to step 1</li>
        </ol>
    </li>
    <li>Otherwise, goto to step 1</li>
</ol>