---
Class: "[[Artificial Intelligence]]"
Sem: WiSe 26/27
---
---
## Index
- [[#Introduction|Introduction]]
- [[#STRIPS|STRIPS]]
- [[#PDDL|PDDL]]
	- [[#PDDL#Domain files|Domain files]]
	- [[#PDDL#Problem files|Problem files]]
- [[#FDR|FDR]]
	- [[#FDR#Semantics|Semantics]]
- [[#STRIPS vs. FDR|STRIPS vs. FDR]]
	- [[#STRIPS vs. FDR#Conversions|Conversions]]
		- [[#Conversions#FDR-2-STRIPS|FDR-2-STRIPS]]
		- [[#Conversions#STRIPS-2-FDR (naive translation) (reference only)|STRIPS-2-FDR (naive translation) (reference only)]]
		- [[#Conversions#STRIPS-2-FDR (clever translation)|STRIPS-2-FDR (clever translation)]]
- [[#Complexity|Complexity]]
---
## Introduction

>[!info] Classical planning
>- **deterministic** vs. stochastic
>- **single-agent** vs. multi-agent

The symbolic specification of $\Pi$ comes in a problem description language

>[!info] Symbolic specification of a problem
>Here's one way to do it:
>- **$P$** → set of Boolean variables (*prepositions*)
>- **$I\subseteq P$** → indicate which proposition are true in the initial state
>- **$G\subseteq P$** → $s$ is a goal state iff $G\subseteq s$
>- **$A$** → set of actions $a$, each which *precondition $pre_{a}$*, *add list $add_{a}$*, and *delete list $del_{a}$*; $a$ applicable to $s$ iff $pre_{a}\subseteq s$, outcome state is $(s \cup add_{a}\backslash del_{a})$
>- **$c$** → maps each $a\in A$ to its cost $c(a)$

This language is called "STRIPS" and specifying the problem means writing STRIPS. It is strictly more powerful than simulators as it allows to implement the simulator API but also to analyze the problem.

The techniques successful for satisficing vs. optimal planning are almost disjoint for symbolic classical planning, but satisficing planning is much more effective in practice.

---
## STRIPS

>[!info] **Definition** (STRIPS planning task)
>A **STRIPS planning task** is a tuple $\Pi=(P,A,c,I,G)$ where:
>- $P$ is a finite set of *facts* (or *propositions*)
>- $A$ is a finite set of *actions*; each $a\in A$ is a triple *$a=(pre_{a}, add_{a}, del_{a})$* of subsets of $P$ referred to as the action's precondition, add list, and delete list respectively; we require that $add_{a}\cap del_{a}=\varnothing$
>- $c:A\longrightarrow \mathbb{R}^{+}_{0}$ is the *cost function*
>- $I\subseteq P$ is the *initial state*
>- $G\subseteq P$ is the *goal*
>
>>[!question] Why do we use a function for cost $c$ instead of a relation?
>>It's a simplification, indeed the cost of an action may vary based on other parameters (e.g. travel time of a road may vary based on the time of the day)

>[!example]
>![[Pasted image 20261105015559.png|400]]
>
>- $P: \{at_{x}, visited_{x}\mid x \in \{Sydney, Adelaide, Brisbane, Perth, Darwin\}\}$
>- $I: \{at_{Sydney}, visited_{Sydney}\}$
>- $G: \{at_{Sydney}, visited_{x} \mid x \in \{Sydney, Adelaide, Brisbane, Perth, Darwin\}\}$
>- $a\in A: drive_{xy}$ where $x$, $y$ have a road
>	- $pre_{a}: \{at_{x}\}$
>	- $add_{a}: \{at_{y},visited_{y}\}$
>	- $del_{a}: \{at_{x}\}$
>- $c$:
>$$
>c(drive_{xy}) =
>\begin{cases}
>1   & \{x,y\} = \{Sydney, Brisbane\} \\
>1.5 & \{x,y\} = \{Sydney, Adelaide\} \\
>3.5 & \{x,y\} = \{Adelaide, Perth\} \\
>4   & \{x,y\} = \{Adelaide, Darwin\}
>\end{cases}
>$$

>[!info] **Definition** (STRIPS State Space)
>Let $\Pi=(P,A,c,I,G)$ be a STRIPS planning task. The **state space** of $\Pi$ is the tuple *$\Theta_{\Pi}=(S,A,c,T,I,S^{G})$* where:
>- $S=2^{P}$ (set of parts)
>- $T=\{s\xrightarrow{a}s'\mid a\in A(s),\ s'=s[\![a]\!]\}$, where
>	- $A(s)=\{a\in A\mid pre_{a}\subseteq s\}$, and for $a\in A(s)$:
>	- $s[\![a]\!]=(s\cup add_{a})\setminus del_{a}$ (for $a\notin A(s)$, $s[\![a]\!]$ is undefined, $s[\![a]\!]=\bot$)
>- $S^{G}=\{s\in S\mid G\subseteq s\}$
>
>We define (optimal) plans for $\Pi$ as (optimal) plans for $\Theta_{\Pi}$.
>
>For $\vec{a}=\langle a_{1},\dots,a_{n}\rangle$ (apply a set of actions sequentially):
>$$s[\![\vec{a}]\!]:=\begin{cases} s & n=0 \\ s[\![\langle a_{1},\dots,a_{n-1}\rangle]\!][\![a_{n}]\!] & n>0 \end{cases}$$
>
>>[!question] Is $\Theta_{\Pi}$ deterministic?
>>Yes: the successor state $s'$ in $s\xrightarrow{a}s'$ is uniquely determined by $s$ and $a$, through $s'=s[\![a]\!]$.

>[!example] STRIPS Encoding of Small "TSP"
>![[Pasted image 20261105021552.png|400]]
>
>- $P: \{at_{x}, visited_{x}\mid x \in \{Sydney, Adelaide, Brisbane\}\}$
>- $I: \{at_{Sydney}, visited_{Sydney}\}$
>- $G: \{visited_{x} \mid x \in \{Sydney, Adelaide, Brisbane\}\}$ (Note: no $at_{Sydney}$)
>- $a\in A: drive_{xy}$ where $x$, $y$ have a road
>	- $pre_{a}: \{at_{x}\}$
>	- $add_{a}: \{at_{y},visited_{y}\}$
>	- $del_{a}: \{at_{x}\}$
>- $c$:
>$$
>c(drive_{xy}) =
>\begin{cases}
>1   & \{x,y\} = \{Sydney, Brisbane\} \\
>1.5 & \{x,y\} = \{Sydney, Adelaide\}
>\end{cases}
>$$
>
>![[Pasted image 20261105022345.png]]
>
>Optimal plan (cost $1+1+1.5=3.5$): $\langle drive_{SyBr},\ drive_{BrSy},\ drive_{SyAd}\rangle$.
>
>>[!question] Is this actually the state space?
>>No, only the reachable part. E.g., $\Theta_{\Pi}$ also includes the states $\{v_{Sy}\}$ and $\{at_{Sy}, at_{Br}\}$.

---
## PDDL
The **Planning Domain Definition Language** (*PDDL*) is a language which variants are used by all implemented planning systems. Its syntax is inspired by Lisp programming language.

```pddl
(and (or (on A B) (on A C))
	 (or (on B A) (on B A))
	 (or (on C A) (on A B)))
```

The planner inputs is separated into a **domain file** (predicates, types, action schemas) and a problem file (objects, initial state, goal).

>[!node] Some examples to get your feet wet
>http://fai.cs.uni-saarland.de/hoffmann/PlanningForDummies.zip

### Domain files
A PDDL domain file consist of:

```pddl
(define (domain <name>))
```

- a *requirements* definition (use `:ald :typing` by default)
- definitions of *types* (each object variable has a type)
- definitions of *predicates*
- definitions of *action schemas*

>[!example] Blocksworld
>A set of blocks sits on a table, and a robot arm can move them one at a time. Blocks can be stacked on top of each other, and the arm can hold at most one block. The goal is to rearrange the blocks into a target configuration (e.g., a tower $A$ on $B$ on $C$).
>
>*Domain file types and predicates*:
>```pddl
>(define (domain Blocksworld)
>	(:requirements :adl :typing)
>	; block is a subtype of object
>	; blueblock, smallblock are subtypes of block
>	(:types block - object
>	        blueblock smallblock - block)
>	; on: x is a smallblock, y is a block
>	(:predicates (on ?x - smallblock ?y - block)
>	             (ontable ?x - block) ; x is on the table
>	             (clear ?x - block))  ; nothing on top of x
>)
>```
>
>*Action schema*:
>```pddl
>(:action fromtable
>	:parameters (?x - smallblock ?y - block)
>	:precondition (and (not (= ?x ?y))
>	                   (clear ?x)
>	                   (ontable ?x)
>	                   (clear ?y))
>	:effect (and (not (ontable ?x))
>	             (not (clear ?y))
>	             (on ?x ?y)))
>```

### Problem files
A PDDL problem file consists of:

```pddl
(define (problem <name>)
(:domain <name>))
```

- definitions of objects belonging to each type
- definition of the inital state (list of *ground predicates* initially true)
- definition of the goal (a formula like action preconditions)

>[!example] Blocksworld
>*Problem file*:
>```pddl
>(define (problem example)
>	(:domain Blocksworld)
>	(:objects a b c - smallblock)
>			  d e - block
>			  f - blueblock)
>	(:init (clear a) (clear b) (clear c)
>		   (clear d) (clear e) (clear f)
>		   (ontable a) (ontable b) (ontable c)
>		   (ontable d) (ontable e) (ontable f))
>	(:goal (and (on a d) (on b e) (on c f)))
>)
>```

>[!example] Run of FF
>In sub-directory "hanoi" of http://fai.cs.uni-saarland.de/hoffmann/PlanningForDummies.zip
>
>Executing `../ff -o domain.pddl -f p-n3.pddl` gives:
>```
>ff: found legal plan as follows
>step 0: MOVE D1 D2 PEG3
>	 1: MOVE D2 D3 PEG2
>	 2: MOVE D1 PEG3 D2
>	 3: MOVE D3 PEG1 PEG3
>	 4: MOVE D1 D2 PEG1
>	 5: MOVE D2 PEG2 D3
>	 6: MOVE D1 PEG1 D2
>0.00 seconds total time
>```

---
## FDR

>[!info] **Definition** (FDR planning task)
>A finite-domain representation planning task, short FDR planning task, is a tuple $\Pi=(V,A,c,I,G)$ where:
>- $V$ is a finite set of **state variables**, each $v\in V$ with a *finite domain* $D_{v}$. We refer to (partial) functions on $V$, mapping each $v\in V$ into a member of $D_{v}$, as (partial) variable assignments.
>-  $A$ is a finite se of **actions**; each $a \in A$ is a pair $(pre_{a}, eff_{a})$ of a partial variable assignments referred to as the action's preconditions and effects; we require that $pre_{a}(v)\neq e\hspace{-0.1em}f\hspace{-0.1em}f_{a}(v)$ whenever both are defined.
>- $c:A\to \mathbb{R}^{+}_{0}$ is the **cost function**
>- $I$ is a complete variable assignment called the **initial state**
>- $G$ is a partial variable assignment called the **goal**

>[!tip]
>In FDR, (partial) variable assignments represent states (e.g. $I$), conditions in $pre_{a}$ and $G$, and effect instructions in $e\hspace{-0.1em}f\hspace{-0.1em}f_{a}$

>[!example] FDR Encoding of "TSP"
>![[Pasted image 20261108165625.png|400]]
>
>- $V$: $at: \{Sydney, Adelaide, Brisbane, Perth, Darwin\}$; $visited_{x}: \{T, F\}$ for $x \in \{Sydney, Adelaide, Brisbane, Perth, Darwin\}$
>- $I$: $at = Sydney$, $visited_{Sydney} = T$, $visited_{x} = F$ for $x \neq Sydney$
>- $G$: $at = Sydney$, $visited_{x} = T$ for all $x$
>- $a\in A: drive_{xy}$ where $x$, $y$ have a road
>	- $pre_{a}: at = x$
>	- $e\hspace{-0.1em}f\hspace{-0.1em}f_{a}: at = y,\ visited_{y} = T$
>- $c$:
>$$
>c(drive_{xy}) =
>\begin{cases}
>1   & \{x,y\} = \{Sydney, Brisbane\} \\
>1.5 & \{x,y\} = \{Sydney, Adelaide\} \\
>3.5 & \{x,y\} = \{Adelaide, Perth\} \\
>4   & \{x,y\} = \{Adelaide, Darwin\}
>\end{cases}
>$$

### Semantics

>[!info] Notation
>Pairs $(v,d)$ are **facts**, also written $v=d$. We identify partial variable assignements $p$ with fact sets. We write $Vars[p]:={v\in V\mid p(v) \text{ is defined}}$

>[!info] **Definition** (FDR State Space)
>Let $\Pi=(V,A,c,I,G)$ be an FDR planning task. The **state space** of $\Pi$ is the tuple *$\Theta_{\Pi}=(S,A,c,T,I,S^{G})$* where:
>- $S$ consists of the complete variable assignments
>- $T=\{s\xrightarrow{a}s'\mid a\in A(s),\ s'=s[\![a]\!]\}$, where
>	- $A(s):=\{a\in A\mid pre_{a}\subseteq s\}$, and for $a\in A(s)$:
>$$
>s[\![a]\!](v):=
>\begin{cases}
>e\hspace{-0.1em}f\hspace{-0.1em}f_{a}(v) & v\in Vars[e\hspace{-0.1em}f\hspace{-0.1em}f_{a}] \\
>s(v) & v\notin Vars[e\hspace{-0.1em}f\hspace{-0.1em}f_{a}]
>\end{cases}
>$$
>	- (for $a\notin A(s)$, $s[\![a]\!]:=\bot$)
>- $S^{G}=\{s\in S\mid G\subseteq s\}$

>[!tip]
>In $s[\![a]\!]$, instead of “adding/deleting” facts, we overwrite the previous variable values by $e\hspace{-0.1em}f\hspace{-0.1em}f_{a}$

>[!example]
>(using $v\_x$ for visited $visited_{x}=T$, and omitting $visited_{x}=F$)
>
>![[Pasted image 20261108170551.png]]
>
>This is only the reachable part of the state space: e.g., $\Theta_{\Pi}$ also includes the state $\{at=Sy,v_{Br}\}$ (but neither $\{v_{Sy}\}$ nor $\{at=Sy,at=Br\}$ compare).

---
## STRIPS vs. FDR

>[!question] How do people use FDR?
>Our surface language is PDDL, which corresponds to STRIPS. However many planning tools are based on **Fast Downward** which reads PDDL input, and then conducts a “*clever*” STRIPS-2-FDR translation.
>
>That translation involves a PSPACE-complete sub-problem.

>[!question] Why does FD do that?
>Because of the practical efficiency of FDR! FDR captures more structure in the planning task, and avoids myriads of unreachable states in regression

>[!question] So why does anybody use STRIPS?
>It’s a legacy system

### Conversions
- **FDR-2-STRIPS** → for each variable $v$ with domain $\{d_{1},\dots ,d_{k}\}$, make $k$ STRIPS facts “$v=d_{1}$“, …, “$v=d_{k}$”
- **STRIPS-2-FDR** → naive vs. clever variants

>[!question] What role does all this play here?
>Some techniques are easier to introduce in STRIPS, some are easier in FDR, so we will keep both around. Both STRIPS and FDR are used in practice. The programming exercises are in FD, hence are based on FDR. Relevance of FDR-2-STRIPS: If we introduce a technique X in STRIPS, then X in FDR (and hence your FD-based code) is equivalent to “convert-FDR-2-STRIPS-then-do-X”.

#### FDR-2-STRIPS

>[!info] **Definition** (FDR-2-STRIPS)
>Let $\Pi=(V,A,c,I,G)$ be an FDR planning task. The **STRIPS conversion** of $\Pi$ is the STRIPS task *$\Pi^{\text{STR}}=(P_{V},A^{\text{STR}},c,I,G)$* where:
>- $P_{V}=\{v=d\mid v\in V,\ d\in D_{v}\}$ is the set of (STRIPS) *facts*
>- $A^{\text{STR}}=\{a^{\text{STR}}\mid a\in A\}$ where $pre_{a^{\text{STR}}}=pre_{a}$, $add_{a^{\text{STR}}}=e\hspace{-0.1em}f\hspace{-0.1em}f_{a}$, and
>$$
>del_{a^{\text{STR}}}=\bigcup_{(v=d)\in e\hspace{-0.1em}f\hspace{-0.1em}f_{a}}
>\begin{cases}
>\{v=pre_{a}(v)\} & \text{if } pre_{a}(v) \text{ is defined} \\
>\{v=d'\mid d'\in D_{v}\setminus\{d\}\} & \text{otherwise}
>\end{cases}
>$$
>- The cost function $c$ is defined by $c(a^{\text{STR}}):=c(a)$ for all $a^{\text{STR}}\in A^{\text{STR}}$
>- $I$ and $G$ are identical to those of $\Pi$

The adds establish the new variable values of $e\hspace{-0.1em}f\hspace{-0.1em}f_{a}$; the deletes make sure to erase the previous values of those variables.

>[!tip] Take-home message
>FDR variable/value pairs $\approx$ STRIPS facts!

>[!info] **Proposition**
>Let $\Pi=(V,A,c,I,G)$ be an FDR planning task, and let $\Pi^{\text{STR}}$ be its STRIPS conversion. Then $\Theta_{\Pi}$ is isomorphic to the sub-system of $\Theta_{\Pi^{\text{STR}}}$ induced by those $s\subseteq P_{V}$ where, for each $v\in V$, $s$ contains exactly one fact of the form $v=d$. All other states in $\Theta_{\Pi^{\text{STR}}}$ are unreachable.

>[!example] FDR-2-STRIPS on "TSP"
>![[Pasted image 20261108172033.png|400]]
>
>- **FDR $V$**: $at: \{Sydney, Adelaide, Brisbane\}$; $visited_{x}: \{T, F\}$ for $x \in \{Sydney, Adelaide, Brisbane\}$
>- **STRIPS $P$**: $at = x$, $visited_{x} = T$, $visited_{x} = F$ for $x \in \{Sydney, Adelaide, Brisbane\}$
>
>- **FDR $dr_{xy}$**: $pre = \{at = x\}$, $e\hspace{-0.1em}f\hspace{-0.1em}f = \{at = y,\ visited_{y} = T\}$
>- **STRIPS $dr_{xy}$**:
>	- $pre = \{at = x\}$
>	- $add = \{at = y,\ visited_{y} = T\}$
>	- $del = \{at = x,\ visited_{y} = F\}$

#### STRIPS-2-FDR (naive translation) (reference only)

>[!info] **Definition** (STRIPS-2-FDR)
>Let $\Pi=(P,A,c,I,G)$ be a STRIPS planning task. The **FDR conversion** of $\Pi$ is the FDR task *$\Pi^{\text{FDR}}=(V_{P},A^{\text{FDR}},c,I^{\text{FDR}},G^{\text{FDR}})$* where:
>- $V_{P}=\{v_{p}\mid p\in P\}$ are the variables, all with domain $\{T,F\}$
>- $A^{\text{FDR}}=\{a^{\text{FDR}}\mid a\in A\}$ where $pre_{a^{\text{FDR}}}=\{v_{p}=T\mid p\in pre_{a}\}$ and $e\hspace{-0.1em}f\hspace{-0.1em}f_{a^{\text{FDR}}}=\{v_{p}=T\mid p\in add_{a}\}\cup\{v_{p}=F\mid p\in del_{a}\}$
>- The cost function $c$ is defined by $c(a^{\text{FDR}}):=c(a)$ for all $a^{\text{FDR}}\in A^{\text{FDR}}$
>- $I^{\text{FDR}}=\{v_{p}=T\mid p\in I\}\cup\{v_{p}=F\mid p\notin I\}$; and $G^{\text{FDR}}=\{v_{p}=T\mid p\in G\}$

All variables here have two possible values only, so this does not benefit at all from the added expressivity of FDR. Hence the designation "naïve".

>[!info] **Proposition**
>Let $\Pi=(P,A,c,I,G)$ be a STRIPS planning task, and let $\Pi^{\text{FDR}}$ be its FDR conversion. Then $\Theta_{\Pi}$ is isomorphic to $\Theta_{\Pi^{\text{FDR}}}$.

>[!example]- STRIPS-2-FDR, Naïve: Small "TSP"
>- **STRIPS $P$**: $at_{x}, visited_{x}$ for $x\in\{Sydney, Adelaide, Brisbane\}$
>- **FDR $V$**: $at_{x}, visited_{x}: \{T,F\}$ for $x\in\{Sydney, Adelaide, Brisbane\}$
>
>- **STRIPS $dr_{xy}$**: $pre=\{at_{x}\}$, $add=\{at_{y},visited_{y}\}$, $del=\{at_{x}\}$
>- **FDR $dr_{xy}$**:
>	- $pre=\{at_{x}=T\}$
>	- $e\hspace{-0.1em}f\hspace{-0.1em}f=\{at_{y}=T,\ visited_{y}=T,\ at_{x}=F\}$

#### STRIPS-2-FDR (clever translation)
To do a clever translation we need to find sets $\{p_{1},\dots,p_{k}\}$ of STRIPS facts so that every reachable state $s$ makes exactly one $p_{i}$ true.

Deciding whether this holds, for a given $\{p_{1},\dots,p_{k}\}$ is PSPACE-complete, but one can design fast algorithms finding *some* such sets.

For each set $\{p_{1},\dots,p_{k}\}$ found, make one FDR variable $v$ with domain $\{d_{1},\dots,d_{k}\}$.

>[!note]
>This is implemented in the pre-processor of Fast Downward



>[!example] STRIPS-2-FDR Naïve vs. Clever: Small "TSP"
>![[Pasted image 20261108172033.png|400]]
>
>- **STRIPS $P$**: $at_{x}, visited_{x}$ for $x\in\{Sydney, Adelaide, Brisbane\}$
>- **Naïve $V$**: $at_{x}, visited_{x}: \{T,F\}$ for $x\in\{Sydney, Adelaide, Brisbane\}$
>- **Clever $V$**: $at: \{Sydney, Adelaide, Brisbane\}$; $visited_{x}: \{T,F\}$ for $x\in\{Sydney, Adelaide, Brisbane\}$
>
>The naïve version is merely STRIPS in disguise. The clever version is more natural, and is explicit about the "truck position".

---
## Complexity

>[!question] Why complexity analysis?
>- is saves you from spending lots of time trying to invent algorithms that do not exist
>- killer app in symbolic planning (tractable fragments implies efficiently computable heuristic functions). Moreover identifying special cases (“tractable fragments”) can be solved in polynomial time making possible to relax the input into the special case to obtain a heuristic function

>[!example] “Only-adds” relaxation
>![[Pasted image 20261108173910.png|300]]
>
>- $P$: $\{truck_{x}\mid x\in\{A,B,C,D\}\}\cup\{pack_{x}\mid x\in\{A,B,C,D,T\}\}$
>- $I$: $\{truck_{A},\ pack_{C}\}$
>- $G$: $\{truck_{A},\ pack_{D}\}$
>- $A$: (notated as "precondition $\Rightarrow$ adds, $\neg$ deletes")
>	- $drive_{xy}$, where $x$, $y$ have a road: "$truck_{x}\Rightarrow truck_{y},\ \neg truck_{x}$"
>	- $load_{x}$: "$truck_{x},\ pack_{x}\Rightarrow pack_{T},\ \neg pack_{x}$"
>	- $unload_{x}$: "$truck_{x},\ pack_{T}\Rightarrow pack_{x},\ \neg pack_{T}$"
>
>**Only-Adds Relaxation**: drop the preconditions and deletes.
>
>"$drive_{xy}: \Rightarrow truck_{y}$"; "$load_{x}: \Rightarrow pack_{T}$"; "$unload_{x}: \Rightarrow pack_{x}$"
>
>Say we want to use this for generating a heuristic function: we solve the relaxed problem on state $s$ to obtain $h(s)$.
>
>>[!done] Solving
>>Our problem:
>>- given STRIPS task $\Pi=(P,A,I,G)$ find action sequence $\overrightarrow{a}$ leading from $I$ to a state that contains $G$, when pretending that preconditions and deletes are empty; for admissibility of $h$, guarantee that $\overrightarrow{a}$ is a shortest such action sequence.
>>
>>**Solution 1** (simplest possible approach):
>>$$
>>\begin{array}{l}
>>\vec{a}:=\langle\rangle \\
>>\textbf{while } G\neq\varnothing \textbf{ do} \\
>>\quad \text{select } a\in A \\
>>\quad G:=G\setminus add_{a} \\
>>\quad \vec{a}:=\vec{a}\circ\langle a\rangle;\ A:=A\setminus\{a\} \\
>>\textbf{endwhile} \\
>>\textbf{return } h:=|\vec{a}|
>>\end{array}
>>$$
>>>[!question] Is this $h$ admissible?
>>>No. Selecting an arbitrary action each time, $\vec{a}$ may be longer than needed.
>>
>>**Solution 2**:
>>$$
>>\begin{array}{l}
>>\vec{a}:=\langle\rangle \\
>>\textbf{while } G\neq\varnothing \textbf{ do} \\
>>\quad \text{select } a\in A \text{ s.t. } |add_{a}| \text{ is maximal} \\
>>\quad G:=G\setminus add_{a} \\
>>\quad \vec{a}:=\vec{a}\circ\langle a\rangle;\ A:=A\setminus\{a\} \\
>>\textbf{endwhile} \\
>>\textbf{return } h:=|\vec{a}|
>>\end{array}
>>$$
>>
>>>[!question] $h$ admissible?
>>>No, large $add_{a}$ doesn't help if the intersection with $G$ is small.
>>
>>**Solution 3:**
>>$$
>>\begin{array}{l}
>>\vec{a}:=\langle\rangle \\
>>\textbf{while } G\neq\varnothing \textbf{ do} \\
>>\quad \text{select } a\in A \text{ s.t. } |add_{a}\cap G| \text{ is maximal} \\
>>\quad G:=G\setminus add_{a} \\
>>\quad \vec{a}:=\vec{a}\circ\langle a\rangle;\ A:=A\setminus\{a\} \\
>>\textbf{endwhile} \\
>>\textbf{return } h:=|\vec{a}|
>>\end{array}
>>$$
>>
>>>[!question] $h$ admissible?
>>>Still no. Example: $G=\{A,B,C,D,E,F\}$; $add_{a_{1}}=\{A,B,C\}$; $add_{a_{2}}=\{D,A\}$; $add_{a_{3}}=\{E,B\}$; $add_{a_{4}}=\{F,C\}$.
>>>
>>>Greedy picks $a_{1}$ first (3 goals), then $a_{2},a_{3},a_{4}$ ($h=4$), but $\langle a_{2},a_{3},a_{4}\rangle$ alone already covers $G$ (optimal $=3$).
>>
>>Maybe a post-process removing redundant actions would do the trick?

>[!info] Morale
>![[bangdesk.gif]]
>Non esiste un trucco semplice: trovare la soluzione ottima del problema only-adds è NP-hard, e la slide successiva lo dimostra probabilmente con una riduzione da Set Cover. Ne seguono due conseguenze:
>- **Il planning è intrinsecamente difficile.** Se è NP-hard perfino la versione senza precondizioni e senza delete, il problema completo lo è ancora di più. Per STRIPS generale, decidere se esiste un piano è PSPACE-complete.
>- **Ottimo e satisficing sono mondi diversi.** Trovare _una_ soluzione rilassata è facile, perché basta un greedy polinomiale. Trovare quella _ottima_ è difficile. È lo stesso discorso dell’introduzione, dove si diceva che il satisficing planning è molto più efficace in pratica.

>[!info] **Definition** (PlanEx)
>By **PlanEx**, we denote the problem of deciding a STRIPS/FDR planning task $\Pi$, whether or not there exists a plan for $\Pi$.
>
>Corresponds to satisficing planning.

>[!info] **Definition** (PlanLen)
>By **PlanLen**, we denote the problem of deciding, given a STRIPS/FDR planning task $\Pi$ and an integer $K$, whether or not there exists a plan $\Pi$ of length at most $K$.
>
>Corresponds to optimal planning.

>[!info] **Theorem** (complexity of PlanEx)
>PlanEx is **PSPACE-complete**
>
>Because Actions $\approx$ transition rules on small “memory” (the fact set $P/$the variables $V$) $\approx$ Turing machine with polynomially bounded space.
>
>>[!info] **Corollary**
>>PlanLen is **PSPACE-complete**
>>

In general, PlanEx and PlanLen have the same complexity, however, within particular applications, bounded length plan existence (optimal planning) is often harder than plan existence (satisficing planning).

In many planning benchmark domains, PlanLes is NP-complete while PlanEx is in P (e.g. blocksworld).

>[!tldr]
>In practice, optimal planning is (almost) never easy.

>[!example] Elevator control
>![[Pasted image 20261108191143.png|350]]
>
>- $VIP$ → served first
>- $D$ → lift may only go down when inside; similar to $U$
>- $NA$ → never alone; $AT$ is the attendant
>- $A$, $B$ → never together in the same elevator
>- $P$ → normal passenger

