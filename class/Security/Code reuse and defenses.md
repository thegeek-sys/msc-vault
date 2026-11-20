---
Class: "[[Security]]"
Sem: WiSe 26/27
---
---
## Non-executable stack
Classic shellcode works only because the stack is executable, so if we mark the stack (and heap, and data) **non-executable** we can avoid it.

To do so we have two options:
- hardware support → the NX bit (AMD), XD bit (Intel); Windows calls it DEP
- make a page writable or executable, not both (W^X)

In such a way the CPU refuses to execute injected shellcode on the stack, so the overflow still corrupts memory, but the payload can’t run as code.

To overcome this countermeasure attackers stopped injecting code, and started reusing existing code.

---
## Code reuse attacks

>[!info] Code injection vs. code reuse
>![[Pasted image 20261119171853.png|300]]
>
>Think of the program as a control-flow graph (CFG).
>
>With **code injection** you add a *new node* to the CFG.
>- the attacker’s own shellcode
>- blocked by NX
>
>With **code reuse** you add a *new edge/path* through existing nodes
>- no new code, so NX does no help
>- the attacker stitches together code that is already there
>
>We can just to any block that contains some code, it doesn’t have to belong directly to the program we are trying to exploit but it just need to be linked to the program.

### Return-to-libc

![[Pasted image 20261119182522.png|230]]

This attack consists in reusing an entire existing function (e.g., `system("bin/sh")`), indeed `libc` is linked into almost every process, and is full of useful functions.

The overflow steps up the stack so that `ret` jumps to `system` and the argument (a pointer to `"/bin/sh"`) is placed where the calling convention expects it.

In this way we defeat NX: `system` is legitimate, executable code.

### Return-oriented programming (ROP)

![[Pasted image 20261119184843.png|269]]

The **return-oriented programming** generalize return-to-libc (reuse fragments of functions, not whole functions).

A **gadget** is a short instruction sequence ending in `ret` e.g., `pop rdi; ret` loads a value into `RDI`, then returns. This isn’t a function that someone wrote by hand: it is a piece of code already present in the binary or in the `libc`, that the attacker exploits for the collateral effect it produces.

The attacker, thanks to the overflow, place on the stack a sequence of gadget’s addresses. In this way each `ret` does the `pop` of the next address on the stack and jumps to it, making the stack a list of gadget to execute instead of “data”.


>[!info]
>È il `ret` che fa da “motore” che avanza da un gadget al successivo. Per questo si chiama _return_-oriented programming: il flusso è guidato da una catena di `ret`.

With enough gadgets, ROP is Turing-complete. Given gadgets to set registers, write to memory, and call functions, the attacker can in principle express any computation, without ever injecting a single new instruction. The typical goal is to set up and call `execve` (to get a shell) or `mprotect` (to make the stack executable again and jump back to injected shellcode).

![[Pasted image 20261119190936.png]]

The stack holds gadget addresses and data; each ret advances to the next gadget.

![[Pasted image 20261119191257.png]]

x64 instructions are variable-length (1–15 bytes) with no alignment requirement, so jumping into the middle of an instruction decodes into a different one, hugely increasing the number of available gadgets. For this reason architectures with fixed-length aligned instructions (e.g., ARM) have far fewer gadgets

>[!tip]
>`ropgadgets` ci permette di trovare i gadget dentro un programma.

#### Finding gadgets
Tools scan a binary or library for gadgets (e.g., `ropgadgets`, `ropper` search by pattern, integrated into pwndbg/GEF).

```python
from pwn import *
context.arch = 'amd64'
elf = ELF('./vuln')
rop = ROP(elf)
rop.raw(rop.find_gadget(['pop rdi', 'ret']))
rop.raw(next(elf.search(b'/bin/sh')))
rop.call(elf.sym.system)

payload = b’A’*72 + rop.chain()
```

---
## Stack canaries
The idea is to place a secret canary value between the local buffers and the saved RBP/return address.