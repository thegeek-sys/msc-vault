---
Class: "[[Security]]"
Sem: WiSe 26/27
---
---
## Introduction
To prevent memory-corruption attacks, the ideal would be to have no-memory corruption bugs at all. This can be achieved either by fixing all bugs (audit, static and dynamic analysis) or by removing the entire bug class (memory-safe languages).

Until then, the best way is to make exploitation harder even when bugs exist. This can be done in several ways:
- randomize the target (ASLR)
- enforce that control flow stays legitimate (CET, CFI)
- detect illegal memory accesses (MTE)

---
## ASLR
**Address Space Layout Randomization** (*ASLR*) randomizes the base address of code, stack, heap, and libraries at load time, making absolute addresses of variables, shellcode, and ROP gadgets no longer predictable. So the attacker must **guess** an address, which usually crashes the target (64-bit gives enough entropy that blind guessing is impractical).

However a single **leaked pointer**  reveals the base, derandomizing an entire region; this is why format-string and other read primitives are so valuable.

If code uses absolute addresses, it can’t be relocated freely as function and data addresses depend on the (randomized) base.

The solution is to use **Position-Independent Code** (*PIC*/*PIE*) where all addressing is relative to the instruction pointer (RIP-relative on x64) so that the same code runs at any base address (no rewriting needed).

In this case the global data is reached indirectly through a table filled in at load time (the GOT). It consists in a pointers array where each entry contains the read address of an external symbol (variable or function); this array is updated with the read addresses at load time.

>[!info] (background)
>Windows instead uses a **relocation table**: the leader patches absolute addresses at load time

### Global Offset Table (GOT)
The **Global Offset Table** (*GOT*) holds the runtime addresses of global symbols. In practice the code loads an address from the GOT (rather than hard-coding it), while the dynamic loader fills the GOT entries at load time.

There are two indirections to reach a global variable:
1. load its true address from the GOT (RIP-relative)
2. access the variable through that address

![[Pasted image 20261124112543.png]]

#### Procedure Linkage Table (PLT)
Functions from shared libraries are resolved **lazily** (on first call) as resolving everything at startup would be slow.

A call to `func@plt` goes through the PLT: on the first call the PLT invokes the loader, which resolves the address and stores it in the GOT, while later on calls the PLT jumps straight to the address now in the GOT.

This is relevant for attackers because GOT entries are essentially function pointers, and overwriting a GOT entry (a “GOT overwrite“) hijacks a later call.

![[Pasted image 20261124113311.png]]
(the first time the GOT entry contains the address of `PLT[0]` which is used to populate the GOT entry)
### Limits
- **coarse-grained** → only the base is randomized, the internal layout is fixed (one leak derandomizes the whole region)
- fine-grained ASLR and software diversity randomize the internal layout too → permute functions, basic blocks, or register allocation per build/run; probabilistic defenses (raise the cost, don’t give a hard guarantee)
- 32-bit ASLR has too little entropy (brute-forceable)

