---
Class: "[[Security]]"
Sem: WiSe 26/27
---
---
## Index
- [[#Non-executable stack|Non-executable stack]]
- [[#Code reuse attacks|Code reuse attacks]]
	- [[#Code reuse attacks#Return-to-libc|Return-to-libc]]
	- [[#Code reuse attacks#Return-oriented programming (ROP)|Return-oriented programming (ROP)]]
		- [[#Return-oriented programming (ROP)#Finding gadgets|Finding gadgets]]
- [[#Stack canaries|Stack canaries]]
	- [[#Stack canaries#Variants|Variants]]
	- [[#Stack canaries#Limits|Limits]]
- [[#Return-address XORing|Return-address XORing]]
- [[#Shadow stack|Shadow stack]]
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
The idea is to place a secret canary value between the local buffers and the saved RBP/return address (the prologue writes it, and the epilogue checks it before `ret`), so that a linear overflow that reaches the return address must overwrite the canary first creating a mismatch that leads to an abort (`__stack_chk_fail`).

gcc/LLVM store the reference canary in thread-local storage (`fs:0x28`).

>[!info] Capiamo
>![[bangdesk.gif]]
>
>Lo shellcode viene comunque scritto nel buffer: il canary non impedisce nessuna scrittura. Funziona perché sta **tra il buffer e il return address**, quindi un overflow lineare che vuole arrivare al return address deve per forza passarci sopra e modificarlo. Nell’epilogo, prima del `ret`, la funzione confronta il valore sullo stack con quello originale (in `fs:0x28`). Se sono diversi chiama `__stack_chk_fail`, che termina il processo. Il `ret` non viene mai eseguito, quindi il return address manipolato non viene mai usato.

### Variants
- **random canary** → a random value chosen at program start (the attacker must guess or leak it, but a random 64-bit canary is infeasible to guess in one try, for each new process a new canary is generated)
- **terminator canary** → bytes that terminate C string functions (e.g., `0x00`, `0x0a`, `0xff`); a string-based overflow cannot write past them without stopping

These two techniques are often combined (a random canary with a NUL low byte).

### Limits
The canary only protect the saved RBP/return address, *not* local variables or functions pointers and neither *non-linear* writes.

A format-string `%n` write, or an arbitrary-write primitive, jumps straight to the return address completely bypassing the canaries

Furthermore it is **leakable**: any info-leak (e.g., format-string `%s`) reveals the canary.

Finally, on **forking servers** the children keep the parent's canary, which enables a **byte-by-byte brute force**: a *crash vs. no-crash* oracle, ~256 tries per byte.

![[Pasted image 20261120121725.png|555]]

>[!info] Capiamo
>Un canary a 64 bit non si indovina: $2^{64}$ tentativi. Il brute force funziona solo se puoi riprovare molte volte **contro lo stesso valore**, ed è proprio quello che un server con `fork()` ti regala: ogni figlio è una copia del padre, quindi tutti condividono **lo stesso canary**. Se un figlio crasha, il padre ne genera un altro col canary immutato (al contrario di un re-`exec`, che lo rigenererebbe daccapo).
>
>Non leggi il canary, lo **indovini un byte alla volta**. Il trucco: fai traboccare l'overflow solo fino al byte che stai testando, così i byte più profondi restano intatti — e quindi corretti — e l'unica variabile è quello che stai indovinando. Byte sbagliato → `__stack_chk_fail` → crash; byte giusto → nessun crash. Fissato un byte, allunghi l'overflow di uno e passi al successivo.
>
>Costo: al massimo $256$ tentativi per byte su $8$ byte,
>$$256 \times 8 \approx 2048 \approx 2^{11} \quad\text{invece di}\quad 2^{64}$$
>cioè da **esponenziale** a **lineare**.

---
## Return-address XORing
Another counter measure, similar to stack canaries, is to XOR the saved return address with a random secret (generated new for each new execution of the process). In practice, the prologue stores `rip xor key` and the epilogue XORs again  before the `ret`.

An attacker who overwrites the saved address cannot predict the result after the decryption, as the XOR clobbers their value into a wrong (likely invalid) address. This method also relies on keeping the key secret

```asm
myfun:
	push rbp
	mov rbp, rsp
	mov rcx, [xorsecret]
	xor [rbp+8], rcx ; encrypt rip
	xor rcx, rcx ; wipe key
	...
	mov rcx, [xorsecret]
	xor [rbp+8], rcx ; decrypt rip
	xor rcx, rcx
	leave
	ret
```

>[!info] XORing vs. stack canary
>Both rely on a **random secret** chosen at program start and shared by all functions for the whole execution, so both must be kept secret.
>- **Where the secret lives**: the canary is an *extra value* placed between the buffers and the saved RBP/return address; XORing adds nothing to the stack and **encrypts the return address itself** (`rip ⊕ key`).
>- **How the attack is stopped**: the canary *detects* the overflow (mismatch → `__stack_chk_fail` → abort, `ret` never runs); XORing doesn't detect anything, it **corrupts** the attacker's value at decryption, so `ret` jumps to an unpredictable (likely invalid) address and the program crashes.
>- **What is protected**: the canary also covers the saved RBP, since it sits below it; XORing protects only the return address.

---
## Shadow stack
The last countermeasure we analyze is the **shadow stack**. This method consists in storing the return address twice:
- on the normal stack (which the attacker can corrupt)
- on a separate, protected *shadow stack* (the address is chosen randomly far from the stack at runtime for each new execution).


![[Pasted image 20261120122941.png|568]]

In practice, on `call` the return address is pushed to both, and on `ret` these two addresses are compared. If there is a mismatch the process aborts. In such way, it is possible to detect any corruption of the saved return address (there is also hardware support with Intel).
