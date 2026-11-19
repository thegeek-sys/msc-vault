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