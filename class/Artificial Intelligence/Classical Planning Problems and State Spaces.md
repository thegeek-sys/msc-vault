---
Class: "[[Artificial Intelligence]]"
---
---
## Introduction
To talk about **classical planning** we first have to introduce some terminologies:
- *deterministic* → for each action there is only one outcome
- *stochastic* → the effects of the actions can be uncertain or probabilistic
- *single-agent* → just one agent plans his actions
- *multi-agent* → multiple agents interact with the environment

And classical planning is *deterministic*, *stochastic*, *fully-observable* (you can acknowledge at every moment in which state it is), *static* (the changes can only by applied by us) and *discrete* (usually finite. It is indeed the simplest form of sequential decision making and the **plan** is the path from the unique initial state to a goal state.

Furthermore we can consider this kind of problems “relatively” simple (their general formulation is PSPACE-complete) but relevant. In further pages there will be stochastic multi-agent settings, which require (much) more complex plans prescribing how to act in every possible state (i.e. how to reach to every possible event/opponent move).

The **search** in this case consists of exploring the *state space*, even if there are many other search paradigms (encoding into constrains satisfaction, BDD representation) that we won’t consider.

---
## Some example of classical planning problems
### Finding a route in a map
- starting from an initial state → departure city
- apply actions → using a road segment
- reach a goal state → destination city
- performance measure → shortest time, shortest path, etc.

### The “15-puzzle”
TODO: foto slide 6
- starting from an initial state → left
- apply actions → moving a tile
- reach a goal state → right
- performance measure: minimize summed-up action costs → each move has cost $1$, so we minimize the number of moves

### FreeCell
- starting from an initial state → cards on table
- apply actions → moving a card
- reach a goal state → all cards “home”
- performance measure: minimize summed-up action costs → each move has cost $1$, so we minimize the number of moves

### Finding bugs (in automata network)
TODO: foto slide 8
- starting from an initial state → both idle
- apply actions → automaton transitions
- reach a goal state → error (both in critical section $CS$)
- performance measure: minimize summed-up action costs → each move has cost $1$, so we minimize the length of the error path

---
## Classical planning terminology
We need to distinguish between **satisficing planning** and **optimcal planning**:
- **satisficing planning**
	- input → a planning task $\Pi$
	- output → a plan for $\Pi$, or unsolvable if no plans for $\Pi$ exists
- **optimal planning**
	- input → a planning task $\Pi$
	- output → a *cost-optimal* plan for $\Pi$, or unsolvable if no plan for $\Pi$ exists

---
## State spaces
Every problem $\Pi$ specifies a state space $\Theta$.

>[!info] Def. state spaces
>A state space is a tuple $\Theta=(S,A,c,T,I,S^{G})$:
>- $S$ → finite set of *states*
>- $A$ → finite set of *actions*
>- $c:A\to \mathbb{R}^{+}_{0}$ → *cost function* 
>- $T\subseteq S\times A\times S$ → *transition relation*. We require that $T$ is *deterministic* (i.e. $\forall\, s\in S,a\in A$ there is at most one state $s'$ such that $(s,a,s')\in T$). If such $(s,a,s')$ exists, then $a$ is *applicable* to $s$ and results in state $s'$. We denote by *$A(s)$* the set of actions applicable to $s$
>- $I\in S$ → *initial state*
>- $S^{G}\subseteq S$ → set of *goal states*
>
>We say that $\Theta$ *has the transition* $(s,a,s')$ if $(s,a,s')\in T$. We also write *$s\xrightarrow{a}s'$*, or *$s\to s'$* when not interested in $a$.
>
>We say that $\Theta$ has *unit cost* if, $\forall \,a \in A,\,c(a)=1$. We will often give each action $a\in A$ a *name*, and identify $a$ with that name.
>
>>[!abstract] Terminology
>>- $s'$ *successor* of $s$ if $s\to s'$, $s$ predecessor of $s'$ if $s\to s'$
>>- $s_{n}$ *reachable* from $s_{0}$ if there exists a sequence of transition $s_{0}\xrightarrow{a_{1}}s_{1},\dots,s_{n-1}\xrightarrow{a_{n}}s_{n}$
>>	- $a_{1},\dots,a_{n}$ → is the *path* from $s$ to $s'$
>>	- $s_{0},\dots,s_{n}$ → is also called *path* from $s$ to $s'$
>>	- the *cost* of that path $\sum^{n}_{i=1} c(a_{i})$
>>- $s'$ *reachable* (without reference state) means reachable from $I$
>>- $s$ is *solvable* if some $s'\in S^{G}$ is reachable from $s$, else $s$ is a *dead end*
>>
>>>[!info] Def. plans
>>>Let $\Theta=(S,A,c,T,I,S^{G})$ be a state space, and let $s\in S$. A **plan** for $s$ is a path from $s$ to some $s'\in S^{G}$. The plan is *optimal* if its cost is minimal among all plans for $s$. A plan for $I$ is called *plan for $\Theta$*. If a plan exists then $\Theta$ is *solvable*


>[!example] Directed graph
>Directed graph + mark-up for initial state and goal states
>TODO: foto slide 16
>
>>[!question]
>>- Does this $\Theta$ have unit cost? No
>>- Which actions are applicable to the initial state? actA, actB, actC
>>- Is $T$ deterministic? No, on $s_{8}$ and $s_{10}$, actG induces more than one outgoing transition
>>- What for do we allow $0$-cost actions? Negligible cost (e.g. switch light on)
>>- Are all states in $\Theta$ reachable? No, $s_{7}$ is not reachable
>>- Are all states in $\Theta$ solvable? No, $s_{2}$ is not solvable
>>- What are the optimal plans for $\Theta$? Any path that starts with actB, applies actE $n\in \{0,2,4,\dots\}$ times, then applies actC then actE and then no actions other than actG

>[!example] Find a route to Bucharest (formalization)
>- $S=\{at(x)\mid x \text{ city in Europe}\}$
>- $A=\{move(x,y) \mid x, y \text{ linked by a road segment}\}$
>- $c$ maps each $move(x,y)$ to the length of the road segment
>- $T=\{(at(x), move(x,y), at(y))\mid x,y \text{limited by a road segment}\}$
>- $I=at(\text{SB})$
>- $S^{G}=\{at(\text{Bucharest})\}$

>[!example] Koalas and gorillas
>TODO: foto slide 22
>$$\Pi:\text{ cross river without being beaten up}$$
>
>Starting with everybody on the right bank we need to use the boat which carries $1\leq x\leq 2$ animals to get everybody to the left bank. If koalas are outnumbered by gorillas on either bank is game over.
>
>We consider only the states at the end of each boat ride, not the situation during the boat ride. At the end of each move, everybody leaves the boat (in other words, the animals in the boat count as being on the target river bank); and the game is over in case that results in more gorillas than koalas on either bank. Moves after which the game would be over are disallowed (i.e. these action are not applicable).
>
>>[!tldr] Formalization
>>- $S=\{(K,G,B) \mid 0\leq K,G\leq 3, 0\leq B\leq 1\}$ where $K, G, B$ respectively represent the number of koalas, gorillas and boats currently in the bank
>>- $I=(3,3,1)$
>>- $G^{G}=\{(0,0,0),(0,0,1)\}$
>>- $c: \text{unit } 1$
>>- $A=$ if $B=1$, subtract $1$ or $2$ from $(K+B)$ and set $B:=0$, if $B=0$, add $1$ or $2$ to $(K+G)$ and set $B:=1$. Both subject to having, after the move, $)\leq K,G\leq 3$ as well as $K\geq G$ if $K>0$, and $3-K\geq 3-G$ if $3-K>0$
>>- $T=$ accordingly

However in some problems state spaces may be huge. In particular, the state spaces is typically exponentially large in the size of its specification via the problem $\Pi$; indeed huge space $\Theta$ are (almost always) specified by small $\Pi$ and so we wouldn’t be able to easily represent them. For this reason we need some **specifications**.

---
## Specifications
Specifications are formal descriptions of what a system must do or the properties it must satisfy, necessary in order to solve a problem.

Let’s present the three possible ones.

### Simulator specification
A **simulator** for a problem $\Pi$ implements the procedures necessary to generate paths in the state space (we don't need to represent the whole state space in advance, we generate successor states on demand by applying actions to the current state using the simulator).

>[!info] Problem simulator API
>- `InitialState()` → returns the initial state of the problem
>- `GoalTest(s)` → returns a Boolean, `true` iff state $s$ is a goal state
>- `Cost(a)` → returns the cost of action $a$
>- `Actions(s)` → returns the set of actions that are applicable to state $s$
>- `ChildState(s,a)` → requires that action $a$ is applicable to state $s$ (i.e., there is a transition $s\xrightarrow{a}s'$). Returns the outcome state $s'$
>
>In this case “specifying the problem” means programming the simulator.

A problem simulator provides procedures allowing to explore the state space incrementally and to determine goal states and plan cost. For this reason it’s required to respect the minimal functionality (i.e., only implement essential operations), but often is much easier to obtain than symbolic specification.

### Symbolic (aka declarative) specification
A **symbolic specification** of $\Pi$ comes from a declarative language.

>[!info] Symbolic specification of a problem
>There are many ways to do this. Here’s one:
>- $P$ → set of Boolean variables (*prepositions*)
>- $I\subseteq P$ → indicating which propositions are true in the initial state
>- $G\subseteq P$ → where $s$ is a goal state iff $G\subseteq s$ (all prepositions are satisfied)
>- $A$ → set of actions $a$, each with *precondition $pre_{a}$*, *add list $add_{a}$*, and *delete list $del_{a}$*; $a$ applicable to $s$ iff $pre_{a}\subseteq s$, outcome state is $(s \cup add_{a}) \setminus del_{a}$
>- $c$ → maps each $a\in A$ to its cost $c(a)$
>
>This language is called “STRIPS”. In this case “specifying the problem” means writing STRIPS

This specification is strictly more powerful than simulator, as it allows to implement the simulator and allows to analyze the problem however often is more difficult to obtain than a simulator.

Furthermore symbolic specification enable general planning, but is not used for search algorithms.
TODO: foto slide 29

>[!example]
>$$
>\begin{aligned}
>P &= \{p, q, g\} \\
>I &= \emptyset \\
>G &= \{g\}
>\end{aligned}
>$$
>
>| Azione | PRE         | ADD    | DEL       |
>| ------ | ----------- | ------ | --------- |
>| $a_p$  | $\varnothing$ | $\{p\}$ | $\varnothing$ |
>| $a_q$  | $\varnothing$ | $\{q\}$ | $\varnothing$ |
>| $a_g$  | $\{p\}$     | $\{g\}$ | $\varnothing$ |
>
>> [!note] obs
>> I don’t actually need $q$, and then neither $a_{q}$.
>
>Starting from $S = \emptyset$, I apply $a_p$:
>
>$$
>\text{apply}(a_p, S) = (S \cup \{p\}) \setminus \varnothing = \{p\}
>$$
>
>> [!info] Closed-world assumption
>> We assume that what is inside $S$ is true, what is outside is false
>
>$$
>S_0 = \varnothing \xrightarrow{a_p} \{p\} \xrightarrow{a_g} \{p, g\} \supseteq G
>$$
>
>The plan is $\pi = \langle a_p, a_g \rangle$: $a_g$ is applicable because $\text{PRE}(a_g) = \{p\} \subseteq \{p\}$.

### Enumerative specification
The **enumerative specification** describes $\Pi$ simply in terms of its state space.

>[!info] Enumerative specification of a problem
>$$\Pi=\Theta$$
>
>We simply input the state space graph (in some implementation).

In this case “specifying the problem” means writing down the state space, but it’s impossible for large state spaces. However it can be solved easily in the size of the state space (Dijkstra’s algorithm).

Enumerative specifications do not, in difference to simulator and symbolic specification, compactly describe large state spaces. They are used if the state spaces are “small” (only 100.000s of states) and the runtime is very limited. This is typically the case in route planning.