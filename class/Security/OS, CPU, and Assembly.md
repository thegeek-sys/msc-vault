---
Class: "[[Security]]"
Sem: WiSe 26/27
---
---
## OS
The OS is a layer of abstraction over the hardware and the process is an abstraction for a program in execution (OS keeps a lot of context per process, e.g., registers, files, etc. and processes are isolated from each other).

Even on a single CPU, processes get a fair share via time slicing by using **multi-processing** (OS enforces isolation and scheduling). Processes can even fork child processes which inherit privileges, file descriptors, and memory layout from the parent.

---
## Virtual memory
Every process has its own virtual address spaces (it sees a private, contiguous memory, regardless of physical layout). Memory is organized in **pages** (typically 4KiB), and the OS and MMU map virtual pages to physical frames. Pages can be swapped out, shared, or marked read-only/non-executable.

This gives isolation between processes, and protection within a process as a page can be readable and writable but not executable (foreshadowing: NX/DEP).

![[Pasted image 20261109233935.png]]

### Canonical addressing
x64 defines 64-bit virtual addresses, but CPUs implement only 48 bits (newer CPUs 57) as the top 16 bits must copy bit 47 (“canonical” addresses).

The address space splits into:
- **user space**  (lower half) → per-process content
- **kernel space** (upper half) → identical for all processes, accessible only in kernel mode

![[Pasted image 20261109234255.png|438]]

#### Privilege rings
x86/x64 CPUs support four privilege levels (rings):
- ring 0 → kernel (full hardware access)
- ring 3 → user application
- ring 1 and 2 → rarely used

![[Pasted image 20261109235621.png|350]]

Privileged instructions and direct hardware access are only allowed in ring 0 and switching rings happens through controlled entry points (system calls, interrupts).

With hardware virtualization (Intel VT-x, AMD-V), the CPU has more than privilege rings. It also has a “switch” that says whether the hypervisor is running (_VMX root mode_) or a virtual machine (_VMX non-root mode_). This isn’t an extra ring but a separate axis. A guest still has its own ring 0 (kernel) and ring 3 (applications); it just runs in non-root mode.

The guest kernel believes it has full control. When it tries something sensitive, like executing `CPUID`, writing to CR3 or accessing an I/O port, the CPU stops it with a **VM exit**. Control passes to the hypervisor, which handles the event and then returns to the guest with a **VM entry** (virtualizating the whole process would be too slow).
All of this is governed by the **VMCS**, an in-memory structure with one per virtual CPU (AMD calls it the VMCB). It holds:
- the guest state to save and restore
- the host state to resume from after an exit
- the rules for what triggers a VM exit
- the reason for the last exit

---
## Process memory layout
Processes use stack to store data structures known at compile time (e.g. code and global and static variable, of those the compiler knows the size and can reserve appropriate memory area in advance).

The heap stores dynamic data structures of which the size or number of data structures is not known in advance (using malloc).

Each subfunction call uses new memory space on the stack, and, as result, stack use is growing with deeper function call nesting. If a subfunction returns, its memory spaces is freed (not erased) and the next subfunction call will overwrite

![[Pasted image 20261110001255.png|198]]

### Memory separation in Windows/Linux
All processes map the kernel, but cannot directly access the kernel land. This allows for efficiency, otherwise kernel switch has to remap and system calls provide the interface between user and kernel land.

System calls can be invoked by an unprivileged processes and traps to OS, i.e., CPU elevates privileges of the process and runs the system call handler which performs operations as requested by process and returns to the user (i.e. CPU downgrades privileges).

![[Pasted image 20261110002301.png|407]]

Executing system calls may be very expensive.

---
## x64 assembly
Intel x86 (32-bit) and x64 (64-bit) are roughly a Von Neumann architecture. In this course we focus on x64 on Linux using Intel syntax which is a CISC architecture (complex instruction set, which means many variable-length instructions).

![[Pasted image 20261110002453.png]]

### Register
There are 16 general-purpose registers: RAX, RBX, RCX, RDX, RSI, RDI, RBP, RSP, R8-R15. Some of them have special roles:
- RIP → instruction pointer (next instruction)
- RSP → stack pointer
- RBP → base/frame pointer

| x64/x86   | Purpose                            |
| --------- | ---------------------------------- |
| `rax/eax` | *A*ccumulator                      |
| `rbx/ebx` | *B*ase address for addressing      |
| `rcx/ecx` | *C*ounter (loop index)             |
| `rdx/edx` | *D*ata (I/O, mul/div)              |
| `rsi/esi` | *S*ource (string source)           |
| `rdi/edi` | *D*estination (string destination) |
| `rbp/ebp` | *B*ase Pointer (frame pointer)     |
| `rsp/esp` | *S*tack Pointer (top of stack)     |

Each 64-bit register has 32/16/8-bit aliases:
- AL $\subset$ AX $\subset$ EAX $\subset$ RAX
- R8B $\subset$ R8W $\subset$ R8D $\subset$ R8

#### The flags register
Arithmetic and logic instructions set **flags** in RFLAGS which are used to drive conditional jumps:
- `ZF` (zero): result was zero
- `SF` (sign): result was negative
- `CF` (carry): unsigned overflow
- `OF` (overflow): signed overflow

Some instructions do not touch flags (e.g., mov, inc/dec leave CF alone).

![[Pasted image 20261110005223.png]]

### Intel assembly notation
Format:
```
<opcode> <op1>, <op2>
```

where `op1` is the destination, and `op2` is the source

Operand types:
- *immediate* → a constant, e.g., `mov, rax, 42`
- *register* → e.g. `mov rax, rbx`
- *memory* → an address, e.g., `mov rax, [rbx]` (load from the address in RBX)

(gdb defaults to a different syntax: operands reversed, registers prefixed)

#### Moving data
- `mov dst, src` is the workhorse; at most one operand may be memory
- zero- vs. sign-extension when widening:
	- `mov eax, ebx` → zero-extends into RAX (writing a 32-bit register clears the top 32 bits)
	- `movzx` / `movsx` → move with explicit zero- / sign-extension
- `xchg a, b` swaps two operands

#### Arithmetic and logic
- Arithmetic → `add`, `sub`, `inc`, `dec`, `mul`/`imul`, `div`/`idiv`
	- multiplication result can exceed the register: `mul` uses RDX:RAX (128-bit)
- Logic → `and`, `or`, `xor`, `not`, `neg`
	- `xor rax, rax` is the idiomatic way to zero a register
- Shifts → `shl`/`sal` (left), `shr` (logical right), `sar` (arithmetic right, keeps sign)

#### Signed vs. unsigned (background)
The CPU uses two’s component for signed values and the same bits are interpreted differently as signed or unsigned (the instructions differ, e.g., `jg` signed vs. `ja` unsigned)

>[!warning]
>Care when widening: use `movsx` to preserve the sign

---
## Control flow in assembly
A program is a set of basic blocks: maximal straight-line instruction sequences, and the control flow connects them (jumps, calls, returns)

There are unconditional jump: `jmp label`, while conditional control flow is the basis of `if`, `switch`, loops (first compare, then jump on the resulting flags).

![[Pasted image 20261110010139.png|147]]

### Comparisons: cmp and test
Both `cmp` and `test` set flags without changing their operands:
- `cmp a, b` → computes `a - b`, sets flags, discards the result (used before signed/unsigned conditional jumps)
- `test a, b` → computes `a AND b`, set flags, discards the result (e.g. `test rax, rax` sets ZF iff RAX is zero, the idiomatic zero check)

Then a conditional jump reads the flags. After `cmp`/`test`, there will be a `j*` instruction jumps depending on the flags:
- `je`/`jz` → equal / zero (`ZF=1`)
- `jne`/`jnz` → not equal / not zero
- signed → `jg`, `jge`, `jl`, `jle`
- unsigned → `ja`, `jae`, `jb`, `jbe`

>[!example]
>```
>	cmp rax, 42
>	jne .else   ; if (rax == 42)
>	...
>.else:
>```

### High-level constructs (background)
Compilers turn source constructs into compare-and-jump patterns
- if/else → a comparison and a conditional jump
- switch → sequential checks, a jump table, or a binary search, depending on the cases
- loops → a body plus a conditional back-jump; while is often compiled to a `do..while` with one jump

Modern compilers optimize heavily: dead-code elimination, constant folding, register reuse (the generated assembly may look quite different from the source).