# ft_libasm

A reimplementation of standard C library functions in x86_64 Assembly (NASM), as part of the École 42 curriculum.

> All content in this document applies exclusively to **x86_64 NASM on Linux**.

---

## Table of Contents

- [Overview](#overview)
- [Compilation](#compilation)
- [Architecture](#architecture)
- [Registers](#registers)
- [Instructions](#instructions)
- [Memory](#memory)
- [Calling Convention](#calling-convention)
- [Syscalls](#syscalls)

---

## Overview

ft_libasm is an introduction to low-level programming through the reimplementation of several libc functions directly in Assembly. The project covers the full compilation pipeline, memory management, register usage, and the Linux x86_64 calling convention.

---

## Compilation

With higher-level languages (HLL), the compilation pipeline goes through three phases:

| Phase | Tool | Input | Output |
|---|---|---|---|
| Compilation | compiler | HLL source | `.s` / `.asm` |
| Assembly | assembler | `.asm` | `.o` |
| Linking | linker | `.o` files | executable |

### Assembling with NASM

```bash
# Linux 64-bit
nasm -f elf64 my_file.asm -o my_file.o

# macOS 64-bit
nasm -f macho64 my_file.asm -o my_file.o
```

### Linking with Clang

Since functions are called from C code, use `clang` (or `gcc`) for linking. The standard `ld` linker does not handle all C-assembled instructions correctly.

```bash
# Link object files directly
clang main.o my_file.o -o a.out

# Link against a static library
clang main.o libasm.a -o a.out
```

> Refer to the project `Makefile` for a complete build reference.

---

## Architecture

### Compilation Pipeline (detailed)

```
[C Source / ASM Source]
        |
        v
  [Compilation]        --> translates HLL to Assembly (.s / .asm)
        |
        v
  [Assembly - NASM]    --> translates Assembly to machine code (.o)
        |
        v
  [Linking - Clang]    --> links all .o files into a single executable
        |
        v
  [Executable]
```

---

## Registers

Registers are the processor's fastest memory, used for syscalls, arithmetic, and general control flow.

### Naming Convention

Registers follow a `[size][name][type]` pattern:

| Prefix | Size |
|---|---|
| `r` | 64-bit |
| `e` | 32-bit |
| *(none)* | 16-bit |

### General Purpose Registers

| Register | Role |
|---|---|
| `rax` | Syscall selector; holds the return value of a syscall; value returned to caller via `RET` |
| `rbx` | General purpose |
| `rcx` | Loop counter — decremented automatically by the `loop` instruction |
| `rdx` | Third argument for syscalls and C calls |
| `rdi` | First argument for syscalls and C calls |
| `rsi` | Second argument for syscalls and C calls |
| `rsp` | Stack pointer — points to the top of the stack |
| `rbp` | Base pointer — used to reference the current stack frame |

### Example — `write(1, "hello world\n", 12)`

```nasm
mov rdi, 1              ; file descriptor (stdout)
mov rsi, hello_str      ; pointer to string
mov rdx, 12             ; byte count
mov rax, 1              ; syscall number for write
syscall
```

> Note: Assembly does not interpret escape sequences. Use their numerical equivalents: `\n` = `10`, null byte = `0`.

---

## Instructions

In NASM, operand assignment goes from **right to left** (opposite of GAS syntax):

```nasm
mov rax, rbx    ; rax <-- rbx
add rax, rbx    ; rax <-- rax + rbx
mul rbx         ; rax <-- rax * rbx
```

### Instruction Families

| Family | Instructions |
|---|---|
| Data movement | `MOV`, `PUSH`, `POP`, `LEA` |
| Arithmetic | `ADD`, `SUB`, `MUL`, `DIV`, `INC`, `DEC` |
| Logic | `AND`, `OR`, `XOR`, `NOT` |
| Control flow | `JMP`, `JE`, `JNE`, `JL`, `JG`, `LOOP` |
| Comparison | `CMP`, `TEST` |
| Function | `CALL`, `RET` |

---

## Memory

RAM is organized into several segments:

```
High addresses
+--------------+
|    Stack     |  <- grows downward (highest address first)
|       v      |
|              |
|       ^      |
|     Heap     |  <- grows upward
+--------------+
|     BSS      |  uninitialized static data
+--------------+
|     Data     |  initialized static data
+--------------+
|     Code     |  executable instructions (.text)
+--------------+
Low addresses
```

### Stack

The stack is a **LIFO** (Last In, First Out) structure used to store temporary data at runtime.

- Addresses start at their **highest value** and decrease as data is pushed
- Managed via `PUSH` (decrements `rsp`, writes data) and `POP` (reads data, increments `rsp`)
- Size is variable at runtime

---

## Calling Convention

In x86_64 Linux (System V AMD64 ABI), arguments are passed through registers in this order:

| Argument | Register |
|---|---|
| 1st | `rdi` |
| 2nd | `rsi` |
| 3rd | `rdx` |
| 4th | `rcx` |
| 5th | `r8` |
| 6th | `r9` |

The return value is placed in `rax`.

This convention applies to both **C function calls** and **syscalls**.

---

## Syscalls

To trigger a syscall, load the syscall number into `rax`, set the arguments in the appropriate registers, then execute `syscall`. The return value is written back into `rax`.

| Function | Syscall Number | Signature |
|---|---|---|
| `read` | `0` | `read(fd, buffer, count)` |
| `write` | `1` | `write(fd, string, count)` |
| `exit` | `60` | `exit(exit_code)` |

### Example — `exit(0)`

```nasm
mov rdi, 0      ; exit code
mov rax, 60     ; syscall: exit
syscall
```
