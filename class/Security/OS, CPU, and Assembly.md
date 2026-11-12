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
- **user space** (lower half) → per-process content
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

The guest kernel believes it has full control. When it tries something sensitive, like executing `CPUID`, writing to CR3 or accessing an I/O port, the CPU stops it with a **VM exit**. Control passes to the hypervisor, which handles the event and then returns to the guest with a **VM entry** (virtualizating the whole process would be too slow). All of this is governed by the **VMCS**, an in-memory structure with one per virtual CPU (AMD calls it the VMCB). It holds:
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

|x64/x86|Purpose|
|---|---|
|`rax/eax`|_A_ccumulator|
|`rbx/ebx`|_B_ase address for addressing|
|`rcx/ecx`|_C_ounter (loop index)|
|`rdx/edx`|_D_ata (I/O, mul/div)|
|`rsi/esi`|_S_ource (string source)|
|`rdi/edi`|_D_estination (string destination)|
|`rbp/ebp`|_B_ase Pointer (frame pointer)|
|`rsp/esp`|_S_tack Pointer (top of stack)|

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
- _immediate_ → a constant, e.g., `mov, rax, 42`
- _register_ → e.g. `mov rax, rbx`
- _memory_ → an address, e.g., `mov rax, [rbx]` (load from the address in RBX)

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

> [!warning] Care when widening: use `movsx` to preserve the sign

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

> [!example]
> 
> ```
>   cmp rax, 42
>   jne .else   ; if (rax == 42)
>   ...
> .else:
> ```

### High-level constructs (background)
Compilers turn source constructs into compare-and-jump patterns
- if/else → a comparison and a conditional jump
- switch → sequential checks, a jump table, or a binary search, depending on the cases
- loops → a body plus a conditional back-jump; while is often compiled to a `do..while` with one jump

Modern compilers optimize heavily: dead-code elimination, constant folding, register reuse (the generated assembly may look quite different from the source).

---

## gdb
gdb is the Linux debugger: it lets you step through a program and inspect its state (registers, memory, stack). It works both on source and on assembly, and it is essential for exercises and exploitation.

> [!warning] gdb changes the environment:
> 
> - it **disables ASLR** by default (use `set disable-randomization off` to keep it)
> - environment variables and the stack layout differ slightly from a plain run (gdb adds variables like `LINES`/`COLUMNS` and runs the program with its absolute path, so stack addresses shift by a few bytes) → an exploit that works inside gdb may fail outside

gdb uses AT&T syntax by default; to get the Intel syntax used in the course: `set disassembly-flavor intel` (can be put in `~/.gdbinit`).

### Typical workflow
On a small vulnerable program:
1. `break vulnerable_fn` and `run`
2. `info registers` → look at RIP, RSP, RBP
3. `x/20gx $rsp` → view the stack, find the buffer and the return address
4. `disassemble` → locate the `ret`
5. `stepi` over the copy loop, watch the saved return address change
6. `continue` and observe where control goes

### Command cheat sheet

|Command|Meaning|
|---|---|
|`break <fn>` / `*<addr>`|set a breakpoint (on function / address)|
|`run` / `continue`|start / resume|
|`next` / `step`|one source line: over / into calls|
|`nexti` / `stepi`|one instruction (over / into calls)|
|`info registers`|all registers|
|`x/20gx $rsp`|20 8-byte words (giant) at RSP in hex|
|`x/16i $rip`|next 16 instructions|
|`disassemble`|disassemble the current function|
|`backtrace`|the call stack|

The `x` format is `x/<count><format><size>`: format `x` (hex), `d` (decimal), `s` (string), `i` (instruction); size `b` (1 byte), `h` (2), `w` (4), `g` (8).

### Extensions
gdb allows Python plugins, so there are (mutually incompatible) extensions that automatically show stack, registers and disassembly at every step:
- **pwndbg**, **GEF**, **PEDA** → Python plugins with extra commands and syntax highlighting
- **radare2** → standalone reverse engineering framework, can also interface with gdb

They are written for reverse engineering and CTFs and add functionality such as search in memory, ROP gadget finder, scripting, test patterns (cyclic patterns to find the offset of the return address), …

---
## The stack

![[Pasted image 20261112171537.png|552]]

The stack is a **LIFO** region for local variables, arguments and return addresses:
- it grows toward **lower** addresses
- RSP points to the top (lowest used address)

Two primitives:
- `push op` → `rsp -= 8`, then store `op` at `[rsp]`
- `pop op` → load `[rsp]` into `op`, then `rsp += 8`

> [!warning] `push`/`pop` implicitly adjust RSP: forgetting this is a common source of bugs

### Function calls
`call fun` does two things:
1. push the return address (the address of the instruction after the `call`, i.e. the next RIP) onto the stack
2. jump to `fun`

`ret` pops the return address into RIP, returning control to the caller.

>[!important]
>This is the mechanism a **stack buffer overflow** hijacks: the return address is just data on the stack, and `ret` blindly trusts it. Overwrite the saved return address and `ret` jumps wherever the attacker wants. Everything in the next lectures builds on this one fact.

---
## Linux calling convention (System V AMD64 ABI)
The calling convention is the agreement between caller and callee on where arguments, return values and registers go:
- first 6 integer/pointer arguments in registers: **RDI, RSI, RDX, RCX, R8, R9**
- further arguments on the stack (pushed right to left, so the 7th is the closest to the return address); the caller removes them after the call
- return value in **RAX**
- before a `call`, RSP must be 16-byte aligned (some functions, e.g. those using SSE instructions like `printf`, crash otherwise → relevant when building exploits)

Caller-saved vs. callee-saved registers:
- **callee-saved** (preserved across a call): RBX, RBP, RSP, R12–R15 → if the callee uses them, it must restore them before returning
- **caller-saved**: everything else (RAX, RCX, RDX, RSI, RDI, R8–R11) may be clobbered by the callee → if the caller needs them after the call, it saves them itself

### Prologue and epilogue
The function prologue/epilogue manage the frame:
- **prologue** → `push rbp; mov rbp, rsp; sub rsp, <locals>`
    - save the caller's frame pointer, set the new frame pointer, reserve space for the locals
- **epilogue** → `leave; ret`
    - `leave` = `mov rsp, rbp; pop rbp` (discard the locals, restore the caller's frame pointer)

### Stack frame layout

![[Pasted image 20261112171916.png|230]]

Each function has its own **stack frame** (relative to RBP):
- `[rbp]` → caller's saved RBP
- `[rbp+8]` → return address
- `[rbp+16]` → 7th argument (if any), `[rbp+24]` → 8th, …
- negative offsets from RBP → local variables

```
high addresses (0xFF..FF)
   ...
   8th parameter        [rbp+24]
   7th parameter        [rbp+16]
   return addr          [rbp+8]
   saved rbp            [rbp]      <- RBP
   local variable A     [rbp-8]
   local variable B     [rbp-16]
   ...                             <- RSP
   red zone (128 B)
low addresses (0x00..00)
```

Below RSP there is the 128-byte **red zone**: scratch space the ABI guarantees won't be clobbered (e.g. by signal handlers), so **leaf functions** (functions that call no other function) can use it for locals without even moving RSP (no `sub rsp, ...`).

>[!important]
>Locals are below the saved RBP and the return address, while writes into a buffer go toward higher addresses. A buffer overflow therefore overruns a local **upward**, past the saved RBP, into the return address.

> [!example] Worked example: `summing()`
> 
> ```c
> uint64_t summing(uint64_t a, uint64_t b,
>     uint64_t c, uint64_t d, uint64_t e,
>     uint64_t f, uint64_t g, uint64_t h) {
>  uint64_t x = a + b + c;
>  uint64_t y = d + e + f;
>  uint64_t z = g + h;
>  return x + y + z;
> }
> int main() { return summing(1,2,3,4,5,6,7,8); }
> ```
> 
> Arguments 1–6 go in RDI, RSI, RDX, RCX, R8, R9; arguments 7–8 on the stack; the locals `x`, `y`, `z` in the frame of `summing`. The call site (unoptimized) looks like:
> 
> ```
> main:
>   push rbp
>   mov  rbp, rsp
>   push 8              ; h (8th arg), pushed first
>   push 7              ; g (7th arg)
>   mov  r9d, 6         ; f
>   mov  r8d, 5         ; e
>   mov  ecx, 4         ; d
>   mov  edx, 3         ; c
>   mov  esi, 2         ; b
>   mov  edi, 1         ; a
>   call summing        ; pushes the return address
>   add  rsp, 16        ; caller removes the two stack args
>   ...
> ```
> 
> Inside `summing`, after the prologue, `g` is at `[rbp+16]` and `h` at `[rbp+24]`; the return value `x + y + z` is left in RAX.

> [!example] Nested calls: `main` → `funB` → `funA`
>![[Pasted image 20261112173719.png|400]]
> 
> ```c
> void funA(int a, int b) { uint64_t vA, vB; vA = a + b; }
> void funB(int x, int y) { uint64_t v1,v2,v3,v4; funA(v1, v4+v3); v2 = v1 + v3 + v4; }
> int main(...) { funB(1, 2); return 0; }
> ```
> 
> - `funB` frame: return address into `main`, saved RBP of `main`, then its locals; called with RDI=1, RSI=2
> - `funA` frame sits right below: return address into `funB`, saved RBP of `funB`, then `vA`, `vB`; called with RDI=a, RSI=b
> 
> The frames form a chain: every saved RBP points to the frame of the caller, which is how `backtrace` walks the call stack.

### Frame pointer omission (background)
RBP is not mandatory: code can address everything relative to RSP:
- this frees RBP as a general-purpose register
- it is common in optimized x64 code (`-fomit-frame-pointer`, enabled by default from `-O1`)

Consequence: there is no neat `[rbp+8]` for the return address (it sits at some offset from RSP that changes as the function pushes things) and debuggers rely on extra unwind information (DWARF, `.eh_frame` section) to reconstruct frames.

---
## System calls
User code asks the kernel for services via the `syscall` instruction: it is like a function call, but crosses into ring 0.
- syscall number in **RAX**
- arguments in **RDI, RSI, RDX, R10, R8, R9**
- return value in RAX (negative values = `-errno`)

> [!warning] The 4th argument is in **R10**, not RCX: the `syscall` instruction clobbers RCX (it saves the return RIP there) and R11 (it saves RFLAGS there).

> [!example] `write(1, buf, len)` → RAX=1 (`write`), RDI=1 (stdout), RSI=buf, RDX=len, then `syscall`

**Shellcode** is often just a few instructions that set up a syscall, e.g. `execve("/bin/sh", NULL, NULL)`:

```
	xor  rsi, rsi            ; argv = NULL
	xor  rdx, rdx            ; envp = NULL
	mov  rax, 0x68732f6e69622f  ; "/bin/sh" in little endian
	push rax                 ; string on the stack (upper byte 0 = terminator)
	mov  rdi, rsp            ; rdi = pointer to "/bin/sh"
	mov  eax, 59             ; execve
	syscall
```
