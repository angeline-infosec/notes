# [Windows x64 Assembly](https://tryhackme.com/room/win64assembly)

_Introduction to x64 Assembly on Windows._

_Note edited with Claude AI_

## Core ideas

- **Reverse engineering (RE):** deconstructing software, binaries, malware, or hardware without the source code to understand how it works. You start from the result and work out how it got there.
- **Assembly (ASM):** a low-level language that communicates directly with the hardware. Compilers translate high-level code into it, and it is what you recover from an executable.
- **Why it's hard:** compiler-generated assembly is optimised for speed, not readability, so it looks nothing like hand-written code.

## Number systems

| System | Notation | Example |
|---|---|---|
| Decimal | none or `d` suffix | `12d` |
| Binary | `0b` prefix, `b` suffix, or leading-zero padding | `0b0110`, `110b` |
| Hex | `0x` prefix, `h` suffix, or `\x` per byte | `0xFF`, `FFh`, `\x12\x45` |

- Hex digits: `A`=10 through `F`=15. Two hex digits = 1 byte.
- Worked example: `0x4A` = (16 × 4) + 10 = 74.

## Data sizes

| Unit | Size |
|---|---|
| Nibble | 4 bits |
| Byte / char / bool | 8 bits (a bool still takes a full byte) |
| Word | 2 bytes |
| DWORD | 4 bytes |
| QWORD | 8 bytes |

- Signed ints spend one bit on the sign. 32-bit signed is -2,147,483,648 to 2,147,483,647, and 32-bit unsigned is 0 to 4,294,967,295.
- **Offset** = distance from the base address. `BaseAddress+0x4` is 4 bytes in.

## Binary operations

- **NOT** flips the bit.
- **AND** gives 1 only if both bits are 1.
- **OR** gives 1 if either bit is 1.
- **XOR** gives 1 if the bits differ.
- **NAND / NOR** perform the operation, then NOT the result.
- `1011 AND 1100` = `1000`
- `1011 NAND 1100` = `0111`
- False is 0. True is anything non-zero.

## Registers

| Register | Conventional use |
|---|---|
| RAX | Accumulator, function return value |
| RBX | Base register (not the base pointer) |
| RCX | Loop counter, 1st argument (Windows) |
| RDX | Data register, 2nd argument (Windows) |
| RSI / RDI | Source / destination index for string operations |
| RSP | Stack pointer, the top of the stack |
| RBP | Base pointer, the base of the stack frame |
| RIP | Next instruction to run (can't be written directly) |
| R8-R15 | General purpose, integers only |

- Sub-registers: `RAX` (64) > `EAX` (low 32) > `AX` (low 16) > `AH` / `AL` (high / low 8).
- Example with `0x0123456789ABCDEF`: EAX = `0x89ABCDEF`, AX = `0xCDEF`, AH = `0xCD`, AL = `0xEF`.
- R8-R15 suffixes: `D` (32-bit), `W` (16-bit), `B` (8-bit), e.g. `R8D`.
- **Zero extension:** writing to `EAX` zeroes the upper 32 bits of `RAX`. Writing to `AX` or `AL` does **not**. `movzx` always zero-extends.

## Instructions

Operand types: **immediate** (a constant like `5`), **register**, **memory address**.

| Instruction | What it does |
|---|---|
| `mov dst, src` | Copy src into dst |
| `lea dst, [addr]` | Load the *address*, never dereferences |
| `push` / `pop` | Put on / take off the top of the stack |
| `inc` / `dec` | +1 / -1 |
| `add` / `sub` | dst = dst ± src |
| `mul` / `imul` | Multiply RAX by the operand (unsigned / signed), result in RDX:RAX |
| `div` / `idiv` | Divide RAX by the operand (unsigned / signed), result in RDX:RAX |
| `cmp a, b` | Subtract b from a, discard the result, set flags |
| `jcc` | Conditional jump based on flags (`jne`, `jle`, `jg`, ...) |
| `call` / `ret` | Call a function / return to the caller |
| `nop` | Does nothing, used for padding |

**`[ ]` vs `lea` (the classic trap):**

```asm
lea rax, [rcx+8]   ; rax = rcx + 8          (address math, no memory read)
mov rax, [rcx+8]   ; rax = value stored AT rcx + 8   (dereferences)
```

**Compilers flip conditions.** `if (x == 4) func1();` usually compiles as "if `x != 4`, jump past the call":

```asm
mov rax, x
cmp rax, 4
jne exit
call func1
exit:
ret
```

## Flags (RFLAGS)

| Flag | Set when |
|---|---|
| ZF (zero) | Result is zero (so `cmp` equal sets ZF) |
| SF (sign) | Result is negative |
| CF (carry) | Unsigned carry or borrow out of the register |
| OF (overflow) | Signed result too big for the register |

- `cmp` is a subtraction that throws the result away and keeps only the flags.
- Conditional jumps just read the flags. They don't need a `cmp` right before them, because other instructions set flags too.

**Jump variants depend on signedness:**

| Unsigned | Signed |
|---|---|
| `ja` above | `jg` greater |
| `jae` above or equal | `jge` greater or equal |
| `jb` below | `jl` less |
| `jbe` below or equal | `jle` less or equal |

Memory aid: humans think in signed numbers and say "greater/less", so those go with signed.

## Windows x64 calling convention (fastcall)

- **Integer / pointer args:** `RCX`, `RDX`, `R8`, `R9` (first four, left to right).
- **Float args:** `XMM0`-`XMM3`, matched by position. `func(1, 3.14, 6, 6.28)` uses RCX, XMM1, R8, XMM3.
- **5th argument onward:** on the stack, starting at `RSP+0x20`.
- **Shadow space:** the caller always reserves 32 bytes (0x20) for the four register args, even with no args.
- **Too big for a register:** passed by pointer.
- **Return value:** `RAX` (integer, bool, char, pointer) or `XMM0` (float, double).
- **Member functions:** `this` is the hidden first argument, so it arrives in `RCX`.
- **Volatile (callee may destroy):** RAX, RCX, RDX, R8-R11, XMM0-XMM5.
- **Non-volatile (callee must save and restore):** RBX, RBP, RDI, RSI, RSP, R12-R15, XMM6-XMM15.
- **Linux differs:** the System V convention passes integer args in `RDI`, `RSI`, `RDX`, `RCX`, `R8`, `R9`. Keep this in mind for Linux binaries.

## Memory layout

- **Stack:** static, known-size data such as locals. LIFO. It grows toward **lower** addresses (push decreases RSP, pop increases it).
- **Heap:** dynamic, unknown-size data (for example user input). It grows toward **higher** addresses and is slower than the stack.
- **Program image:** the loaded executable (a PE file on Windows).

### Stack frame layout

| Address | Contents | Offset |
|---|---|---|
| Lower | Local variables | `RBP - 8` |
| | Saved RBP | `RBP + 0` |
| | Return address | `RBP + 8` |
| Higher | Function parameters | `RBP + 16` |

- On `call`, the return address is the instruction *after* the call, so execution resumes there.
- RBP is saved so the caller's frame can be restored on return.
- On x64 it's common to address locals and params off `RSP` instead, with RBP used as a general register.

### Endianness

- **Little-endian** (x86/x64): least significant byte stored first, so `0xDEADBEEF` sits in memory as `EF BE AD DE`.
- Big-endian stores it as written.

### Why buffer overflows happen

- Variables are *allocated* from high to low addresses.
- Data is *written* from low to high addresses.
- So writing past the end of an array spills into the variable allocated before it. Room example: `stackArr[2] = {3,4,5}` overwrote `stackVar2` with `5`.

## Room gotchas

- The room says the largest signed 8-bit value is 128. It's **127**. (The wraparound example, `75 + 60` giving `-121`, is correct.)
- The room labels both `DIV` and `IDIV` "unsigned". `IDIV` is the signed one.
- The room's 8-argument example uses `push` for args 5-8, and its `pop` example has a typo (`R12 = 6`, should be 7). In real x64 fastcall, stack args are written to `[RSP+0x20]` and up, not popped off.
- The memory layout diagram is heavily simplified, so don't treat its addresses as real Windows layout.

## Takeaway

Not something I'll use directly as a SOC analyst, but registers, flags, and calling conventions are what make Ghidra and GDB output readable in the Red Queen Protocol room.
