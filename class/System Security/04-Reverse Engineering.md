---
Class: "[[System Security]]"
Sem: WiSe 26/27
---
---
## Disassembly
![[Pasted image 20261116164659.png]]
### Linear sweep (objdump)
Decodes instruction after instruction from the start of the section. Data or junk bytes between code desynchronize it
### Recursive traversal
Follows jumps and calls from known entry points but misses code reached only through indirect jumps.

---
## Basic blocks
A **basic block** is a maximal sequence of conscutive instructions such that:
- control flow can *enter* the sequence only at its first instruction
- control flow can *leave* the sequence only after its last instruction

**Leaders** are the first instructions of basic blocks:
1. The entry point of the function is a leader
2. Every target of a conditional or unconditional jump is a leader
3. Every instruction that immediately follows a jump or ret is a leader

![[Pasted image 20261116165001.png]]

### Control-flow graph
![[Pasted image 20261116165706.png]]

- `jmp` → direct unconditional jump
- `taken` → conditional branch; if the jump happens
- `fall-through` → consecutive block/conditional jump not taken (i.e. next block after a conditional jump)

### Call graph
The calls are the edges while the nodes are functions.

>[!example]
>$$main\to foo \to bar$$

### Dominators
- **$d \text{ dom } n$** → every path from $B_{0}$ to $n$ contains $d$
- **$d \text{ strictly dominates } n$** → $d \text{ dom }n$ and $d\neq n$
- **$\text{idom}(n)=d$** →  strict dominator of $n$ that every other strict dominator of $n$ dominates
- **dominator tree** → tree of the $\text{idom}$

### Loops
- **$h$ is the header** → dominates every node of the loop and is its only entry point
- **$n \to h$ is a back edge** → $h \text{ dom }n$ (where $h$ is a header)
- **natural loop** → is the set of nodes composed by the two nodes of a back edge $n\to h$ and all the nodes that can reach $n$ without passing through $h$

>[!warning]
>If there is more than one entry point (i.e., no header) of the loop there are no back edges nor natural loops.

>[!example]
>![[Pasted image 20261116180458.png|500]]

