---
Class: "[[Security]]"
Sem: WiSe 26/27
---
---
## Index
- [[#ASLR|ASLR]]
	- [[#ASLR#Global Offset Table (GOT)|Global Offset Table (GOT)]]
		- [[#Global Offset Table (GOT)#Procedure Linkage Table (PLT)|Procedure Linkage Table (PLT)]]
	- [[#ASLR#Limits|Limits]]
- [[#Hardware-assisted defenses|Hardware-assisted defenses]]
	- [[#Hardware-assisted defenses#Control-Flow Enforcement Technology (CET)|Control-Flow Enforcement Technology (CET)]]
	- [[#Hardware-assisted defenses#ARM Memory Tagging Extension (MTE)|ARM Memory Tagging Extension (MTE)]]
	- [[#Hardware-assisted defenses#ARM Memory Tagging Extension (MTE)|ARM Memory Tagging Extension (MTE)]]
- [[#Control-Flow Integrity (CFI)|Control-Flow Integrity (CFI)]]
	- [[#Control-Flow Integrity (CFI)#Label granularity (background)|Label granularity (background)]]
- [[#Memory-safe languages|Memory-safe languages]]
	- [[#Memory-safe languages#Rust (ownership and borrowing)|Rust (ownership and borrowing)]]
		- [[#Rust (ownership and borrowing)#The unsafe escape hatch|The unsafe escape hatch]]
	- [[#Memory-safe languages#Hardware route: CHERI (background)|Hardware route: CHERI (background)]]
- [[#The arms race, in a glimpse|The arms race, in a glimpse]]
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

>[!warning]
>If the number of external functions is too small, the address resolution is going to be at runtime (not lazy), but the `PLT` structure still exists

### Limits
There can be various levels of grain for ASLR:
- **coarse-grained** → only the base is randomized, the internal layout is fixed (one leak derandomizes the whole region, it will not change e.g. the distance between `libc` and `system`)
- **fine-grained** → randomize the internal layout too; permute functions, basic blocks, or register allocation per build/run; probabilistic defenses (raise the cost, don’t give a hard guarantee)

32-bit ASLR has too little entropy (brute-forceable)

---
## Hardware-assisted defenses
### Control-Flow Enforcement Technology (CET)
**Control-Flow Enforcement Technology** (*CET*) Intel hardware feature against control-flow hijacking, two parts:
- **Shadow stack** (backward edges) → the CPU keeps a protected copy of return addresses, and `ret` faults if the on-stack address doesn’t match the shadow copy (hardware version of the [[Code reuse and defenses#Shadow stack|shadow stack]])
- **Indirect Branch Tracking** (forward edges) → indirect jumps/calls may only land on an `endbranch` instruction, so it prevents jumping into the middle of a function (kills many gadgets)

>[!info] Capiamo
>![[bangdesk.gif]]
>
>L'idea: destinazioni "marcate"
>Il compilatore inserisce un'istruzione speciale, **`endbr64`** (o `endbr32` a 32 bit), all'inizio di ogni punto che può legittimamente essere la destinazione di un salto indiretto:
>
>```asm
>funzione:
>    endbr64          ; "qui si può arrivare con un salto indiretto"
>    push rbp
>    ...
>```
>
>La CPU applica una regola semplice: **dopo un `call` o `jmp` indiretto, l'istruzione successiva deve essere `endbr64`.** In pratica:
>
>1. Dopo un salto indiretto la CPU entra nello stato "WAIT_FOR_ENDBRANCH".
>2. Se la prima istruzione alla destinazione è `endbr64`, lo stato torna a IDLE e l'esecuzione prosegue normalmente.
>3. Se è qualsiasi altra istruzione, la CPU solleva un'eccezione Control Protection e il processo viene terminato.
>
>Così, se l'attaccante prova a saltare a metà di una funzione dove non c'è un `endbr64`, il programma va in crash invece di eseguire il gadget.

ARM has an analogous feature: Pointer Authentication (PAC) signs pointers (similar to XORing-return address)

### ARM Memory Tagging Extension (MTE)
![[Pasted image 20261124170049.png|431]]

### ARM Memory Tagging Extension (MTE)

![[Pasted image 20261124170049.png|431]]

**Memory Tagging Extension** (_MTE_) is the ARM hardware feature (ARMv8.5) that "colors" both memory and pointers, and checks on every access that the colors match:
- **memory tag** → every 16-byte region (granule) gets a 4-bit tag, stored in a separate hardware-managed memory
- **pointer tag** → stored in the unused top bits of the pointer (bits 56-59, the CPU ignores the top byte when translating the address)

On `malloc` the allocator picks a random tag, writes it in the granules and returns a pointer carrying the same tag; on every load/store the CPU compares the pointer's tag with the memory's tag, and a mismatch causes a fault.

It catches both:
- **spatial** errors (out-of-bounds) → adjacent allocations get different tags, so an overflow into the neighbor faults
- **temporal** errors (use-after-free) → on `free` the memory is re-tagged, so a dangling pointer still carries the old tag and faults

Since it's in hardware the overhead is low, so it can stay enabled in production (deployed on recent Android devices, e.g. Pixel 8+).

>[!warning]
>It's a probabilistic defense: with only 4 bits there are 16 tags, so a wrong pointer has ~1/16 chance of matching by accident (adjacent regions are guaranteed different though). It also can't see overflows that stay inside the same granule/object, and a leaked tag lets the attacker forge a valid pointer.

---
## Control-Flow Integrity (CFI)
**Control-Flow Integrity** (_CFI_) is a general defense against code-reuse attacks (introduced by Abadi et al., CCS 2005). The idea is to restrict indirect control flow to the **legitimate** targets from the program's CFG, since a gadget chain follows edges that don't exist in the CFG.

The problematic transfers are only the indirect ones, as the target is read from a register or from memory and so it can be controlled by the attacker:
- **forward edges** → indirect calls and jumps (`call rax`, function pointers, vtables, GOT); restrict to valid function entries/address-taken functions
- **backward edges** → returns (`ret` reads the address from the stack); restrict to valid return sites (ideally the actual caller)

![[Pasted image 20261124173605.png|399]]

For returns static CFI is weak (a `ret` can go after _any_ call site of that function), so backward edges are better protected with a shadow stack, which checks the exact return address.
Direct calls/jumps are fixed at compile time (and code is read-only), so they need no check.

The enforcement is done by assigning **labels** to valid targets (an ID placed at the destination), and checking the label before each indirect transfer: if the label at the destination is not the expected one → fault.

>[!warning]
>The static analysis can't know exactly where an indirect call will go, so the allowed set is an over-approximation: targets are grouped in equivalence classes (if two call sites share a target, their classes are merged). Everything inside the over-approximation is still usable by the attacker (valid for the CFG but wrong at that moment).

### Label granularity (background)
There can be various levels of precision:
- **coarse-grained** → one check for a set of targets (low overhead), but merged set allows unintended edges within the set (bypassed in the literature)
- **fine-grained** → the exact set computed for each call site (high overhead, mostly research)

Returns are the hardest case for static labels as any function may call a given target, so the return-site label set is large. Modern deployments pair forward-edge CFI with a hardware shadow stack (CET) for backward edges.

Real deployments: LLVM CFI, Windows Control Flow Guard (CFG)

---
## Memory-safe languages
Every mitigation so far tolerates the bug and tries to stop exploitation, but the bug is still there; attackers keep finding bypasses. The alternative would be to use a language where the bug class cannot occur.

Managed languages (Java, Go, Python) have a garbage collector and bounds checks, at a
runtime cost, while languages like Rust have memory safety without a garbage collector, at (close to) C performance,

The arms race ends not with a better mitigation, but by removing the vulnerability.

### Rust (ownership and borrowing)
Memory safety in Rust is enforced at compile time by the type system, not at runtime

To achieve this goal we have two principles:
- **ownership** → every value has a single owner; when the owner goes out of scope, the value is freed (no double-free, no leak of the simple kind)
- **borrowing** → references are checked by the borrow checker (either one mutable reference, or any number of immutable references, never both); no dangling pointers (a reference cannot outlive its owner)

```rust
let mut v = vec![1, 2, 3];
let r = &v[0]; // immutable borrow
v.push(4); // COMPILE ERROR: cannot mutate while borrowed
println!("{}", r);
```

Furthermore we have bounds-checked array accesses by default. The result of all of this is: no buffer overflows, no use-after-free, no data races, with no GC

>[!info] Capiamo
>![[bangdesk.gif]]
>
>Rust gestisce la memoria **senza garbage collector e senza `free` manuale**, controllando tutto a compile time (il *borrow checker*).
>
>**Ownership** → ogni valore ha **un solo owner**; quando l'owner esce dallo scope il valore viene liberato automaticamente. L'ownership si può trasferire (*move*), ma poi la variabile vecchia non è più utilizzabile:
>```rust
>let a = String::from("ciao");
>let b = a;            // move: ora l'owner è b
>println!("{}", a);    // ❌ non compila
>```
>
>**Borrowing** → si può usare un valore senza possederlo tramite riferimenti, con una regola: **tanti lettori (`&T`) oppure un solo scrittore (`&mut T`), mai entrambi**:
>```rust
>let mut v = vec![1, 2, 3];
>let primo = &v[0];    // prestito condiviso
>v.push(4);            // ❌ non compila: push potrebbe riallocare → primo diventerebbe dangling
>println!("{}", primo);
>```
>
>**Lifetime** → un riferimento non può sopravvivere al valore a cui punta.
>
>Così spariscono a compile time double free, use-after-free, dangling pointer e data race, senza costi a runtime. Per i casi che il compilatore non riesce a verificare c'è `unsafe`, che lascia la responsabilità al programmatore.

#### The unsafe escape hatch
Some code genuinely needs operations the checker can’t verify (e.g., raw pointers, calling C, hardware access), so `unsafe` blocks opt out of some checks.

Now the programmer promises to uphold the invariants the compiler can’t (memory-safety bugs in Rust almost always live in unsafe code).

The good practice is keep unsafe small, audited, and wrapped in a safe API. It’s not a silver bullet, but it shrinks the trusted, dangerous surface dramatically.

>[!question] Does it work? The evidence
>Google, Android (2019–2025), as new code shifted to Rust, memory-safety bugs fell from 76% of vulnerabilities (2019) to under 20% (2025) and Rust code shows a ~1000x lower memory-safety vulnerability density than the C/C++ code.
>
>Microsoft: historically ~70% of its CVEs were memory-safety issues and industry and governments now push memory-safe languages (US CISA/NSA "memory safety roadmaps")

### Hardware route: CHERI (background)
CHERI: capability hardware that makes pointers unforgeable. A pointer becomes a capability: address plus bounds plus permissions, tagged in hardware and every access is bounds- and permission-checked by the CPU

Retrofits spatial (and some temporal) memory safety onto existing C/C++ (no language change, recompile with a CHERI toolchain). ARM’s Morello is a prototype CPU; research and early adoption ongoing

---
## The arms race, in a glimpse
The Arms Race, in One Slide
- Attack: stack buffer overflow → inject and run shellcode
- Mitigation: NX (no executable stack)
- Bypass: code reuse (return-to-libc, ROP)
- Mitigation: stack canaries, ASLR
- Bypass: info leaks, format-string writes, brute force
- Mitigation: CET, MTE, CFI (raise the bar further)
- Elimination: memory-safe languages (remove the bug class)