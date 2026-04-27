# Roulette

**Category:** Rev | **Difficulty:** Medium

## Overview
This challenge presented a classic reverse engineering puzzle: an ELF binary that initially disguises itself as a simple guessing game. Under the hood, however, it hides a custom Virtual Machine (VM) and a cryptographic state-machine that verifies our input using Cipher Block Chaining (CBC) logic and the Chinese Remainder Theorem (CRT).

## Initial Triage: The Roulette Fake-Out
Opening the decompiled code in Ghidra, the program greets us with a prompt:
`"submit roulette number:"`

It reads up to 256 bytes of input, converts it to an integer, and praises us if we guess `1`. However, this is a red herring. The real flag verification happens entirely separate from the integer conversion. 

The true entry point to the challenge is this length check:
```c
if (lVar9 == 0x6a) { ... }
```
`0x6a` is 106 in decimal. This immediately tells us **the flag must be exactly 106 characters long.**

## The Verification Loop
If the length check passes, the program enters a `do...while` loop that iterates 27 times (`uVar15` goes from 0 to 26). Because $27 \times 4 = 108$, we know the program is processing our 106-byte flag in 4-byte (32-bit) chunks, likely padded with null bytes.

Inside this loop, three main things happen:
1.  **State Initialization:** It sets up an array of "registers" using our current loop counter, a running state variable, and a deeply obfuscated mathematical value.
2.  **VM Execution:** It jumps into a custom bytecode interpreter to manipulate those registers.
3.  **The Cryptographic Check:** It XORs the VM's output against our input chunk and compares it to a hardcoded array.

The check looks like this:
```c
if ((uVar8 ^ uVar16) != (&DAT_00499ce0)[uVar15]) break;
```
Because of the associative property of XOR, if we know the VM's output (`uVar8`) and the expected array, we can recover the required input:
`Input = Expected ^ VM_Output`

## Reverse Engineering the VM
The inner `switch` statement acts as the VM's instruction dispatcher. By analyzing the cases, we can map out the architecture:

* **Instruction Size:** 8 bytes per instruction.
* **Registers:** 8 virtual registers stored on the stack.
* **Pipeline Obfuscation:** The VM uses a clever trick. It reads the *opcode* from `pc + 7`, but it applies the operation to the registers specified in the *next* instruction at `pc + 8`. 

The instruction set decodes as follows:
* **0x00 / 0xFF:** HALT
* **0x01:** `MOV regA, regB`
* **0x02:** `XOR regA, regB`
* **0x03:** `XOR regA, imm32`
* **0x04:** `ADD regA, regB`
* **0x05:** `ADD regA, imm32`
* **0x06:** `MUL regA, imm32`
* **0x07:** `ROL regA, imm8` (Rotate Left)
* **0x08:** `ROL regA, regB`
* **0x09:** `SHR regA, imm8` (Logical Shift Right)

## The Chinese Remainder Theorem (CRT)
Before the VM runs, it calculates an initial register value (`uVar7`) using a function `FUN_00401cd0(A, B, C, D)`. 

Looking at the arguments passed to it, we see modulo operations. This function solves for a value `X` such that:
* `X % B == A`
* `X % D == C`

This is the **Chinese Remainder Theorem**. To emulate the C code in Python, we have to implement a standard CRT solver using the Extended Euclidean Algorithm.

## The CBC Dependency
We cannot just parallelize the 27 iterations and solve them all at once. The initial state of the VM for round $N$ depends heavily on `uVar14`. 

```c
uVar16 = uVar7 * 0x45d9f3b ^ uVar14 ^ uVar16;
uVar14 = (uVar16 << bVar4 | uVar16 >> 0x20 - bVar4) + 0x9e3779b9 + uVar18;
```
Notice that `uVar16` (our recovered input chunk) is fed directly into the calculation for the next round's `uVar14`. This acts like Cipher Block Chaining (CBC). We must write a solver that sequentially recovers a chunk, updates the state using that chunk, and moves to the next.

## Extracting the Data
To build our solver, we needed the raw bytes of the VM instructions and the five data arrays. Instead of fighting Ghidra's UI, we used Ghidra's built-in Python API to dump the memory regions perfectly:

```python
for name, addr, size in [
    ("raw_bytecode", "00499be1", 256),
    ("raw_9de0", "00499de0", 108),
    # ... other arrays
]:
    arr = getBytes(toAddr(addr), size)
    hex_str = "".join(["\\x%02x" % (b & 0xFF) for b in arr])
    print('{} = b"{}"'.format(name, hex_str))
```

## The Final Exploit
With the architecture reversed, the math understood, and the data extracted, the final step was writing a Python emulator. The script initializes the state, iterates 27 times, solves the CRT equations, runs the custom VM, XORs the output to recover 4 bytes of the flag, and updates the state tracker for the next loop.

*(Insert the final Python solver script we built here).*

**Result:** The script successfully emulates the environment forward, unrolling the state dependencies and printing the 106-character flag!
