---
Class: "[[Artificial Intelligence]]"
Sem: WiSe 26/27
---
---
## Introduction

>[!tldr]
>Abstracting a transition system means dropping some distinctions between states, while preserving all transitions and goal states.

An **abstraction** of a transition system $\Theta$ is defined by a function $\alpha$ (the *abstraction mapping*), mapping states to *abstract states* (also *block states*). If $\alpha$ maps states $s$ and $t$ to the same abstract state, then $s$ and $t$ are not distinguishable anymore (they are equivalent under $\alpha$).

The **abstract state space $\Theta^{\alpha}$** on the image of $\alpha$ is defined by homomorphically mapping the goal states and transitions of $\Theta$ to $\Theta^{\alpha}$, thus preserving all solutions. The **abstract remaining cost**, i.e., remaining cost in $\Theta^{\alpha}$, is an estimate *$h^{\alpha}$* for remaining cost in $\Theta$. As we preserve all solutions $h^{\alpha}$ is admissible.

---
## Abstraction basics

>[!example] "Logistics mal anders": One Package, Two Trucks
>![[Pasted image 20261124231729.png|300]]
>
>- $V$: $\{p,\ t_{A},\ t_{B}\}$
>	- $D_{p}=\{L,R,A,B\}$
>	- $D_{t_{A}}=D_{t_{B}}=\{L,R\}$
>- $I$: $\{p=L,\ t_{A}=R,\ t_{B}=R\}$
>- $G$: $\{p=R\}$
>- $A$: (notated as "precondition $\Rightarrow$ effect")
>	- $pickup_{xy}$, where $x\in\{A,B\}$, $y\in\{L,R\}$: "$t_{x}=y,\ p=y\Rightarrow p=x$"
>	- $drop_{xy}$, where $x\in\{A,B\}$, $y\in\{L,R\}$: "$t_{x}=y,\ p=x\Rightarrow p=y$"
>	- $move_{xyy'}$, where $x\in\{A,B\}$, $y,y'\in\{L,R\}$, $y\neq y'$: "$t_{x}=y\Rightarrow t_{x}=y'$"
>
>>[!info] Reading the variables
>>- $p=L$ / $p=R$: the package is **at a location**.
>>- $p=A$ / $p=B$: the package is **inside truck** $A$ / $B$.
>>
>>$\Rightarrow$ No separate "in truck" variable: being loaded is just another value in $D_{p}$.
>
>![[Pasted image 20261124232013.png]]
>
>State $p=x$, $t_{A}=y$, $t_{B}=z$ is depicted as $xyz$. Action names not shown, e.g., the transition to $LLL$ to $ALL$ is induced by $pickup_{AL}$

>[!info] **Definition** (abstraction)
>Let $\Theta=(S,A,c,T,I,S^{G})$ be a state space. An **abstraction** of $\Theta$ is a surjective function $\alpha:S\to S^{\alpha}$ also referred to as the *abstraction mapping*. The **abstract state space** induced by $\alpha$, written $\Theta^{\alpha}=(S^{\alpha},A,c,T^{\alpha},I^{\alpha},S^{\alpha G})$ defined by:
>- $I^{\alpha}=\alpha(I)$
>- $S^{\alpha G}=\{\alpha(s)\mid s\in S^{G}\}$ (preserve goal states)
>- $T^{\alpha}=\{(\alpha(s),a,\alpha(s')) \mid (s,a,s')\in T\}$ (preserve transitions)
>
>The size of the abstraction is the number $|S^{\alpha}|$ of abstract states. $\Theta$ is called the concrete state space.
>
>>[!question] Why do we require $\alpha$ to be surjective?
>>So that $\Theta^{\alpha}$ dues not contain superfluous states

### Abstraction heuristic

>[!info] **Definition** (abstraction heuristic)
>Let $\Theta=(S,A,c,T,I,S^{G})$ be a state space, and let $\alpha$ be an abstraction of $\Theta$. The **abstraction heuristic induced by $\alpha$**, written **$h^{\alpha}$**, is the heuristic function $h^{\alpha}:S\to \mathbb{R}^{+}_{0}\cup \{\infty\}$ which maps each state $s\in S$ to $h^{*}_{\Theta^{\alpha}}(\alpha(s))$
>
>>[!tldr]
>>The abstract remaining cost (remaining cost in $\Theta^{\alpha}$) is used as the heuristic estimate for remaining cost in $\Theta$
>
>>[!note]
>>- more actions can lead to the same abstract state (non-determinism)
>>- $\Theta^{\alpha}$ is not a classical state space: if $(s,a,s')\in T$ and $(t,a,t')\in T$ with $\alpha(s)=\alpha(t)$ but $\alpha(s')\ne \alpha(t')$, then $a$ labels two distinct outgoing transitions from the same abstract state, namely $(\alpha(s),a,\alpha(s'))$ and $(\alpha(t),a,\alpha(t'))$
>>- in $h^{*}_{\Theta^{\alpha}}(\alpha(s))$, abstract plans may choose which transition to take, i.e. this non-determinism is handled “optimistically”

>[!example] Abstraction: “Logistics mal anders”
>Concrete state space:
>![[Pasted image 20261124232717.png|550]]
>
>Abstract state space:
>![[Pasted image 20261124232801.png|550]]
>
>$h^{\alpha}(\{p=L,t_{A}=R, t_{B}=R\})=3\neq h^{*}(\{p=L,t_{A}=R, t_{B}=R\})=4$
>
>>[!tip]
>>A transition between concrete states is “**spurious**” if it exists in the abstract but not in the concrete state space. E.g. we can go in a single step from LRR to LLL; same for ALR to RRR etc.

#### Properties

>[!info] **Proposition** ($h^{\alpha}$ is admissible)
>Let $\Theta$ be a state space, and let $\alpha$ be an abstraction of $\Theta$. Then $h^{\alpha}$ is consistent and goal-aware, and thus also admissible and safe
>
>>[!done] Proof
>>Let $\Theta=(S,A,c,T,I,S^{G})$ and $\Theta^{\alpha}=(S^{\alpha},A,c,T^{\alpha},I^{\alpha},S^{\alpha G})$
>>
>>For goal-awareness, we need to show that $h^{\alpha}(s)=0$ for all $s\in S^{G}$. So let $s\in S^{G}$. Then $\alpha(s)\in S^{\alpha G}$ by definition of abstractions, and hence $h^{\alpha}(s)=h^{*}_{\Theta^{\alpha}}(\alpha(s))=0$.
>>
>>For consistency, we need to show that whenever $(s,a,s')\in T$, $h^{\alpha}\leq h^{\alpha}(s')+c(a)$. By definition, $h^{\alpha}(s)=h^{*}_{\Theta^{\alpha}}(\alpha(s))$ and $h^{\alpha}(s')=h^{*}_{\Theta^{\alpha}}(\alpha(s'))$, so we need to show that $h^{*}_{\Theta^{\alpha}}(\alpha(s))\leq h^{*}_{\Theta^{\alpha}}(\alpha(s'))+c(a)$. Since $(s,a,s')$ is a concrete transition, by definition of abstraction we have an abstract transition $(\alpha(s),a,\alpha(s'))$ in $\Theta^{\alpha}$. But then, $h^{*}_{\Theta^{\alpha}}(\alpha(s))\leq h^{*}_{\Theta^{\alpha}}(\alpha(s'))+c(a)$ holds simply because $h^{*}$ is consistent.

### Abstraction as quotient systems

>[!info] **Definition** (quotient system)
>Let $\Theta=(S,A,c,T,I,S^{G})$ be a state space, and $\alpha:S\mapsto S'$ a surjective function.
>- **Induced equivalence relation** $\sim^{\alpha}$: $s\sim^{\alpha}t \iff \alpha(s)=\alpha(t)$
>- **Quotient system** $\Theta/\sim^{\alpha}=(S/\sim^{\alpha},\ A,\ c,\ T/\sim^{\alpha},\ I/\sim^{\alpha},\ S^{G}/\sim^{\alpha})$ where:
>	- $[s]\in S/\sim^{\alpha}$: equivalence classes under $\sim^{\alpha}$
>	- $([s],a,[s'])\in T/\sim^{\alpha} \iff (s,a,s')\in T$
>	- $I/\sim^{\alpha}=[I]$
>	- $S^{G}/\sim^{\alpha}=\{[s]\mid s\in S^{G}\}$
>
>$\Rightarrow$ $A$ and $c$ stay **unchanged**: we only "glue together" states that $\alpha$ doesn't distinguish.

>[!info] **Proposition** ($\Theta/\sim^{\alpha}$ is isomorphic to $\Theta^{\alpha}$)
>Let $\alpha:S\mapsto S'$ be an abstraction of $\Theta$. Then $\Theta/\sim^{\alpha}\cong\Theta^{\alpha}$.
>>[!done] Proof
>>Direct from definition, via the bijection $[s]\leftrightarrow\alpha(s)$.
>>$\Rightarrow$ Same structure, only the names of the abstract states change.

---
## Pathologies
As always there is the eternal trade-off between accuracy and efficiency:
- we want to obtain an informative heuristic
- we want to obtain a small computational overhead

The abstraction function $\alpha$ is a very powerful parameter, allowing to travel the whole way between both extremes

>[!question] What do we mean by “small computational overhead”?
>- *fast computation of $\alpha$* →  for a given state $s$, the abstract state $\alpha(s)$ must be efficiently obtainable
>- *few abstract states* → for a given abstract state $\alpha(s)$, the abstract remaining cost $h^{\alpha}(s)=h^{*}_{\Theta^{\alpha}}(\alpha(s))$ must be efficiently computable

>[!question] How to obtain non-pathological abstractions?
>- Pattern database heuristics (next lecture)
>- Domain abstraction (not covered)
>- Cartesian abstractions (not covered)
>- Merge-and-shrink abstractions (not covered)
### One-state abstraction

![[Pasted image 20261125001819.png|500]]

$$
\alpha(s):=\text{const}
$$
$+$ trivial to compute $\alpha$ and $h^{*}_{\Theta^{\alpha}}$, just one abststract state
$-$ completely uninformative $h^{\alpha}$

### Identity abstraction

![[Pasted image 20261125002030.png|500]]

$$
\alpha(s):=s
$$
$+$ $h^{\alpha}=h^{\alpha}$, trivial to compute $\alpha$
$-$ computing $h^{*}_{\Theta^{\alpha}}$ corresponds to solving the optimal planning problem

### Perfect abstraction

![[Pasted image 20261125002206.png|500]]

$$
\alpha(s):=h^{*}(s)
$$
$+$ $h^{\alpha}=h^{*}$, few abstract states so easy to compute $h^{*}_{\Theta^{\alpha}}$
$-$ computing $\alpha$ corresponds to solving the optimal planning problem (PSPACE-complete)

---
## Additive abstraction

>[!example] The 15-Puzzle
>![[Pasted image 20261125003459.png|500]]
>
>- Abstractions in AI were first introduced as **pattern database heuristics** for the 15-Puzzle.
>- Here we look at it from an **FDR-planning** perspective.
>
>>[!info] FDR-style encoding
>>- **State:** tuple $\langle b,t_{1},\dots,t_{15}\rangle$ with values $\in\{1,\dots,16\}$
>>	- $b$: position of the **blank**
>>	- $t_{i}$: position of **tile** $i$
>>- $V$: $\{b,t_{1},\dots,t_{15}\}$
>>
>
>>[!info] A 15-Puzzle abstraction
>>- $\alpha$ **ignores** the location of tiles $8,\dots,15$. Two states are distinguished **iff** they differ in the position of the blank or of one of the tiles $1,\dots,7$.
>>
>>$$\alpha(\langle b,t_{1},\dots,t_{15}\rangle):=\langle b,t_{1},\dots,t_{7}\rangle$$
>>
>>$\Rightarrow$ $h^{\alpha}$: number of moves needed to bring tiles $1,\dots,7$ **and the blank** to their goal positions.
>
>| | State | # States |
>|---|---|---|
>| **Concrete** $\Theta$ | $\langle b,t_{1},\dots,t_{15}\rangle$ | $16^{16}\approx1.8\cdot10^{19}$ |
>| **Abstract** $\Theta^{\alpha}$ | $\langle b,t_{1},\dots,t_{7}\rangle$ | $16^{8}\approx4.2\cdot10^{9}$ |
>
>(both counts are upper bounds: they also include tuples where two tiles share a position)
>
>>[!info] The abstract state space in detail
>>- **Goal states:**
>>	- $\Theta$: unique goal $\langle16,1,2,\dots,15\rangle$
>>	- $\Theta^{\alpha}$: unique goal $\langle16,1,2,\dots,7\rangle$
>>- **Transitions:** (notated as "pre $\Rightarrow$ eff", $x$ and $y$ neighboring positions in the $4\times4$ grid)
>>	- $\Theta$: $\langle x,t_{1},\dots,t_{i-1},y,t_{i+1},\dots,t_{15}\rangle\to\langle y,t_{1},\dots,t_{i-1},x,t_{i+1},\dots,t_{15}\rangle$ for all $i\in\{1,\dots,15\}$
>>		- FDR: "$b=x,\ t_{i}=y\Rightarrow b=y,\ t_{i}=x$"
>>	- $\Theta^{\alpha}$, moves of tiles $i\in\{1,\dots,7\}$: same as above
>>		- FDR: "$b=x,\ t_{i}=y\Rightarrow b=y,\ t_{i}=x$"
>>	- $\Theta^{\alpha}$, moves of tiles $j\in\{8,\dots,15\}$: $\langle x,t_{1},\dots,t_{7}\rangle\to\langle y,t_{1},\dots,t_{7}\rangle$
>>		- FDR: "$b=x\Rightarrow b=y$"
>>
>>$\Rightarrow$ The ignored tiles become **invisible**: moving them looks like the blank moving "on its own".
>
>>[!question] How to compute the heuristic?
>>- **Computation of $\alpha$:** easy, just project the 16-tuple onto its first 8 components.
>>	- $\Rightarrow$ example of a pattern database heuristic ($\alpha$ is a projection)
>>- **Computation of abstract remaining costs:**
>>	- precompute all abstract remaining costs before search
>>	- store them in a lookup table
>>
>>$\Rightarrow$ During search, computing $h^{*}_{\Theta^{\alpha}}(\alpha(s))$ is just a **table lookup**.

There is a huge number of possible choices for $\alpha$. This choice governs the informedness of the resulting heuristic function.

>[!example]
>With the same amount of memory required for the lookup table for tiles $1,\dots,7$ (168 states), we could store the lookup tables for 16 different abstractions to six tiles ($16^{7}$ states).

Fortunately there is no need to commit to a single $\alpha$, as we can combine several $\alpha$

>[!question] How to admissibly combine multiple abstractions?
>*Maximizing over several abstractions*:
>- each abstraction mapping gives rise to an admissible heuristic
>- by computing the maximum of several admissible heuristics, we obtain another admissible heuristic which dominates these
>- thus, we can always compute several abstractions and maximize over the individual abstract goal distances
>
>**Better idea: summing over several abstraction**:
>- in some cases, the abstraction heuristics are additive → we can take their sum and still remain admissible
>- summation often reads to much higher estimates than maximization so it is important to understand when abstractions are additive

>[!example] Additive abstractions: 15-puzzle
>![[Pasted image 20261125005346.png|500]]
>- 1st abstraction: ignore location of $8,\dots, 15$
>- 2nd abstraction: ignore location of $1,\dots,7$
>
>The sum of the abstraction heuristics is not admissible
>
>![[Pasted image 20261125005517.png|500]]
>- 1st abstraction: ignore location of $8,\dots,15$ **and blank**
>- 2nd abstraction: ignore location of $1,\dots,7$ **and blank**
>
>The sum of the abstraction heuristics is admissible

### Orthogonal abstractions

>[!info] Terminology
>If $s=s'$ in $(s,a,s')$, then the transition is called a self-loop

>[!info] **Definition** (actions affecting an abstraction)
>Let $\alpha$ be an abstraction of $\Theta$, and let $\alpha$ be one of the actions in $\Theta$. We say that $a$ affects $\alpha$ if $\Theta^{\alpha}$ has at leas one non-self-loop transition induced by $a$, i.e. if *there exists a transition $(\alpha(s),a,\alpha(s'))$ with $\alpha(s)\neq \alpha(s')$*.

>[!info] **Definition** (orthogonal abstractions)
>Let $a_{1}$ and $a_{2}$ be abstractions of $\Theta$. We say that $a_{1}$ and $a_{2}$ are **orthogonal** if no action of $\Theta$ affects both $a_{1}$ and $a_{2}$

>[!example] Orthogonal Abstractions: Example 15-Puzzle
>![[Pasted image 20261125010348.png]]
>
>- **Left-hand side:** $\alpha_{left}$ keeps tiles $1,\dots,7$ and blank, $\alpha_{right}$ keeps tiles $8,\dots,15$ and blank
>- **Right-hand side:** $\alpha_{left}$ keeps tiles $1,\dots,7$ only, $\alpha_{right}$ keeps tiles $8,\dots,15$ only (no blank)
>
>>[!question] Are the left-hand side $\alpha_{left}$ and $\alpha_{right}$ orthogonal?
>>**No.**
>>- E.g. the action moving the blank to the right, mapping $s$ to $t$:
>>	- $\alpha_{left}(s)\neq\alpha_{left}(t)$
>>	- $\alpha_{right}(s)\neq\alpha_{right}(t)$
>>- $\Rightarrow$ not a self-loop in **either** abstraction, so it affects both.
>>
>>$\Rightarrow$ The **blank** is shared: every move changes $b$, which both abstractions see.
>
>>[!question] Are the right-hand side $\alpha_{left}$ and $\alpha_{right}$ orthogonal?
>>**Yes.**
>>- Say $a$ affects $\alpha_{left}$ $\Rightarrow$ $a$ moves a tile $t_{i}$ for $i\in\{1,\dots,7\}$.
>>- Neither $t_{i}$ nor the blank are accounted for in $\alpha_{right}$ $\Rightarrow$ $a$ induces only self-loops there (same vice versa).
>>
>>$\Rightarrow$ Every action affects *at most one* of the two abstractions $\Rightarrow$ $h^{\alpha_{left}}+h^{\alpha_{right}}$ is admissible.

### Orthogonality and additivity

>[!info] **Theorem** (orthogonal abstractions are additive)
>Let $\alpha_{1},\dots,\alpha_{n}$ be pairwise orthogonal abstractions for the same state space $\Theta$. Then $\sum_{i=1}^{n}h^{\alpha_{i}}$ is consistent and goal-aware, and thus also admissible and safe.
>
>>[!tldr]
>>"**Self-loops don't count.**" Every transition in an optimal solution path affects at most one of the abstractions, and thus is counted in at most one of the abstraction heuristics.
>
>>[!done] Proof (orthogonal abstractions are additive)
>>Let $\Theta=(S,A,c,T,I,S^{G})$.
>>
>>**Goal-awareness:** we need to show that $\sum_{i=1}^{n}h^{\alpha_{i}}(s)=0$ for all $s\in S^{G}$. So let $s\in S^{G}$. Then, for all $i$, $h^{\alpha_{i}}(s)=0$ because each $h^{\alpha_{i}}$ is goal-aware.
>>
>>**Consistency:** consider any transition $(s,a,s')\in T$. We need to show that
>>$$\sum_{i=1}^{n}h^{\alpha_{i}}(s)\leq\sum_{i=1}^{n}h^{\alpha_{i}}(s')+c(a)$$
>>
>>*Key step:* because of orthogonality, $\alpha_{i}(s)\neq\alpha_{i}(s')$ for **at most one** $i\in\{1,\dots,n\}$.
>>
>>*Case 1:* $\alpha_{i}(s)=\alpha_{i}(s')$ for all $i$ ($a$ is a self-loop everywhere). Then:
>>$$\begin{aligned}
>>\sum_{i=1}^{n}h^{\alpha_{i}}(s)&=\sum_{i=1}^{n}h^{*}_{\Theta^{\alpha_{i}}}(\alpha_{i}(s))\\
>>&=\sum_{i=1}^{n}h^{*}_{\Theta^{\alpha_{i}}}(\alpha_{i}(s'))&&[\alpha_{i}(s)=\alpha_{i}(s')]\\
>>&=\sum_{i=1}^{n}h^{\alpha_{i}}(s')\\
>>&\leq\sum_{i=1}^{n}h^{\alpha_{i}}(s')+c(a)
>>\end{aligned}$$
>>
>>*Case 2:* $\alpha_{k}(s)\neq\alpha_{k}(s')$, and $\alpha_{i}(s)=\alpha_{i}(s')$ for $i\neq k$ ($a$ affects only $\alpha_{k}$). Then:
>>$$\begin{aligned}
>>\sum_{i=1}^{n}h^{\alpha_{i}}(s)&=\sum_{i\neq k}h^{*}_{\Theta^{\alpha_{i}}}(\alpha_{i}(s))+h^{\alpha_{k}}(s)\\
>>&=\sum_{i\neq k}h^{*}_{\Theta^{\alpha_{i}}}(\alpha_{i}(s'))+h^{\alpha_{k}}(s)&&[\alpha_{i}(s)=\alpha_{i}(s')\text{ for }i\neq k]\\
>>&\leq\sum_{i\neq k}h^{*}_{\Theta^{\alpha_{i}}}(\alpha_{i}(s'))+h^{\alpha_{k}}(s')+c(a)&&[h^{\alpha_{k}}\text{ is consistent}]\\
>>&=\sum_{i=1}^{n}h^{\alpha_{i}}(s')+c(a)
>>\end{aligned}$$
>>
>>Consistent and goal-aware $\Rightarrow$ admissible and safe.
>>

>[!info] Capiamo
>![[bangdesk.gif]]
>La consistency chiede che, facendo un passo $a$, la somma delle heuristic **non scenda più di $c(a)$**.
>- grazie all'ortogonalità, un passo concreto "si vede" in al massimo una abstraction: in tutte le altre è un self-loop e quel termine della somma resta identico.
>- il termine che cambia ($h^{\alpha_{k}}$) scende al massimo di $c(a)$, perché ogni $h^{\alpha_{k}}$ da sola è consistente.
>
>$\Rightarrow$ La somma scende al massimo di $c(a)$: è consistente.
>
>Senza ortogonalità invece una stessa azione potrebbe far scendere due termini di $c(a)$ ciascuno: la somma scenderebbe di $2c(a)$, cioè l'azione verrebbe pagata due volte. È esattamente il problema del blank nel 15-Puzzle: ogni mossa cambia $b$, quindi viene contata in entrambe le abstraction.

>[!example] Orthogonality and additivity: yet another "Logistics"
>![[Pasted image 20261125105021.png|300]]
>
>- $V$: $\{p_{1},\ p_{2},\ t\}$
>	- $D_{p_{1}}=D_{p_{2}}=\{L,R,T\}$ ($T$ = inside the truck)
>	- $D_{t}=\{L,R\}$
>- $I$: $\{p_{1}=L,\ p_{2}=L,\ t=R\}$
>- $G$: $\{p_{1}=R,\ p_{2}=R\}$
>
>State $p_{1}=x$, $p_{2}=y$, $t=z$ is depicted as $xyz$.
>
>Concrete state space $\Theta$:
>![[Pasted image 20261125105047.png|500]]
>
>Abstraction $\alpha_{1}$: only consider position of package 1
>![[Pasted image 20261125105116.png|500]]
>
>Abstraction $\alpha_{2}$: only consider position of package 2
>![[Pasted image 20261125105148.png|500]]
>
>>[!question] Why are $\alpha_{1}$ and $\alpha_{2}$ orthogonal?
>>- $load$/$unload$ of package 1: change $p_{1}$ $\Rightarrow$ affect **only** $\alpha_{1}$ (self-loops in $\alpha_{2}$)
>>- $load$/$unload$ of package 2: change $p_{2}$ $\Rightarrow$ affect **only** $\alpha_{2}$ (self-loops in $\alpha_{1}$)
>>- $drive$: changes only $t$, which neither abstraction considers $\Rightarrow$ self-loops in **both**
>>
>>$\Rightarrow$ No action affects both.
>
>>[!question] What does the sum give in $I=LLR$?
>>- $h^{\alpha_{1}}(I)=2$ ($L\to T\to R$: load, unload package 1)
>>- $h^{\alpha_{2}}(I)=2$ (same for package 2)
>>- $h^{*}(I)=6$ (drive to $L$, load both, drive to $R$, unload both)
>>
>>$\Rightarrow$ $h^{\alpha_{1}}(I)+h^{\alpha_{2}}(I)=4\leq6$: admissible. The drives are self-loops in both abstractions, so **no one counts them**.

