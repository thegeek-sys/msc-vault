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
![[Pasted image 20261022150759.png|500]]

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
![[Pasted image 20261022150826.png|500]]
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
>![[Pasted image 20261022150935.png|450]]
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
>![[Pasted image 20261022151019.png|350]]
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
![[Pasted image 20261022151044.png|center|500]]

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

---
## Search concepts

>[!info] Terminology
>- *search node $n$* → contains a state reached by the search, plus information about how it was reached
>- *path cost $g(n)$* → the cost of the path reaching $n$
>- *optimal cost* $g^*$ → the cost of the optimal plan; for a state $s$, $g^{*}(s)$ is the cost of a cheapest path reaching $s$
>- *node expansion* → generating all successors for a node, by applying all actions applicable to the node’s state $s$; afterwards the state $s$ itself is also said to be expanded
>- *search strategy* → method for deciding which node to expand next
>- *open list* → set of all nodes that currently are candidates for expansion
>- *closed list* → set of all states that were already expanded; used only in *graph search*, not in *tree search*

>[!warning] Tree search vs. graph search
>The difference between tree search and graph search is in the duplicate elimination.
>
>**Tree search**:
>- is another way of saying “do not use duplicate elimination”
>- search space is treated as a tree (do not consider that the same state can be reached from multiple predecessors)
>- the same state may appear in many search nodes
>- the main advantage is that there is no need for a closed list, hence lower memory consumption possible
>
>>[!example] Tree search procedure
>>$$
>>\begin{array}{l}
>>\textbf{function } \textsf{GenericTreeSearch}(\textit{problem}) \textbf{ returns } \text{a plan, or } \textit{unsolvable} \\
>>\quad \textit{node} \leftarrow \text{a node } n \text{ with } n.\textit{State} = \textit{problem}.\textit{InitialState} \\
>>\quad \textit{open} \leftarrow \text{a queue with only element } n \\
>>\quad \textbf{loop do} \\
>>\quad\quad \textbf{if } \textsf{Empty?}(\textit{open}) \textbf{ then return } \textit{unsolvable} \\
>>\quad\quad n \leftarrow \textsf{Pop}(\textit{open}) \\
>>\quad\quad \textbf{if } \textit{problem}.\textsf{GoalTest}(n.\textit{State}) \textbf{ then return } \textsf{Plan}(n) \\
>>\quad\quad \textbf{for each } \text{action } a \textbf{ in } \textit{problem}.\textsf{Actions}(n.\textit{State}) \textbf{ do} \\
>>\quad\quad\quad n' \leftarrow \textsf{ChildNode}(\textit{problem}, n, a) \\
>>\quad\quad\quad \textsf{Insert}(n', \textit{open})
>>\end{array}
>>$$
>
>**Graph search**:
>- is another way of saying “use duplicate elimination”
>- a closed list of already expanded states is kept (for each generated state $s'$ is verified whether is in the closed list or not, if affermative is discarded)
>
>>[!example] Graph search procedure
>>$$
>>\begin{array}{l}
>>\textbf{function } \textsf{GenericGraphSearch}(\textit{problem}) \textbf{ returns } \text{a plan, or } \textit{unsolvable} \\
>>\quad \textit{node} \leftarrow \text{a node } n \text{ with } n.\textit{State} = \textit{problem}.\textit{InitialState} \\
>>\quad \textit{open} \leftarrow \text{a queue with only element } n \\
>>\quad \textit{closed} \leftarrow \text{empty set of states} \\
>>\quad \textbf{loop do} \\
>>\quad\quad \textbf{if } \textsf{Empty?}(\textit{open}) \textbf{ then return } \textit{unsolvable} \\
>>\quad\quad n \leftarrow \textsf{Pop}(\textit{open}) \\
>>\quad\quad \textbf{if } \textit{problem}.\textsf{GoalTest}(n.\textit{State}) \textbf{ then return } \textsf{Plan}(n) \\
>>\quad\quad \textit{closed} \leftarrow \textit{closed} \cup \{n.\textit{State}\} \\
>>\quad\quad \textbf{for each } \text{action } a \textbf{ in } \textit{problem}.\textsf{Actions}(n.\textit{State}) \textbf{ do} \\
>>\quad\quad\quad n' \leftarrow \textsf{ChildNode}(\textit{problem}, n, a) \\
>>\quad\quad\quad \textbf{if } n'.\textit{State} \notin \textit{closed} \textbf{ then} \\
>>\quad\quad\quad\quad \textsf{Insert}(n', \textit{open})
>>\end{array}
>>$$

>[!info] Criteria for evaluating search strategies
>**Guarantees**:
>- *completeness* → guarantees to find a plan when there is one?
>- *optimality* → returned plans guaranteed to be optimal? If so is called optimal, otherwise satisficing
>
>**Complexity**:
>- *time complexity* → how long doe it take to find a plan? (measured in states)
>- *space complexity* → how much memory does the search require? (measured in states)
>
>**Typical state space features governing complexity**:
>- *branching factor $b$* → the maximum number of successors of any state
>- *goal depth $d$* → the number of actions required to reach the shallowest goal state

---
## Blind search

>[!info] Preliminaries
>**Blind search vs. informed search**:
>- *blind search* does not require any input beyond the simulator API
>- *heuristic search* leverages a heuristic function $h$ that maps states to quality estimates (remaining cost in classical planning, expected cost in MDPs, expected utility in games)
>- *pros and cons* → heuristic search typically is much more effective in practice, but somebody’s have to come up with/implement $h$; in symbolic planning $h$ is generated automatically from the symbolic problem specification, but one can also learn heuristic search information through supervised learning or reinforcement learning during the search itself (MCTS)
>
>**Blind search strategies covered**:
>- *BFS*, *DFS*
>- *iterative deepening search* → combines advantages of BFS and DFS
>- *uniform-cost search* → optimal for non-unit costs
>
>**Blind search strategies not covered**:
>- *bi-directional search* → two separate search spaces, one forward from the initial space, the other backward from the goal; stops when the two search spaces overlap

### Iterative deepening search

![[Pasted image 20261022165758.png|500]]

>[!important] Pseudocode
>$$
>\begin{aligned}
>&\textbf{function } \textsf{IterativeDeepeningSearch}(\textit{problem}) \textbf{ returns } \text{a solution, or failure} \\
>&\quad \textbf{for } \textit{depth} = 0 \textbf{ to } \infty \textbf{ do} \\
>&\quad\quad \textit{result} \leftarrow \textsf{DepthLimitedSearch}(\textit{problem}, \textit{depth}) \\
>&\quad\quad \textbf{if } \textit{result} \neq \mathit{cutoff} \textbf{ then return } \textit{result} \\
>\\
>&\textbf{function } \textsf{DepthLimitedSearch}(\textit{problem}, \textit{limit}) \textbf{ returns } \text{a plan, or unsolvable/cutoff} \\
>&\quad \textit{node} \leftarrow \text{a node } n \text{ with } n.\textit{State} = \textit{problem}.\textit{InitialState} \\
>&\quad \textbf{return } \textsf{RecursiveDLS}(\textit{node}, \textit{problem}, \textit{limit}) \\
>&\\
>&\textbf{function } \textsf{RecursiveDLS}(n, \textit{problem}, \textit{limit}) \textbf{ returns } \text{a plan, or unsolvable/cutoff} \\
>&\quad \textbf{if } \textit{problem}.\textsf{GoalTest}(n.\textit{State}) \textbf{ then return } \text{the empty action sequence} \\
>&\quad \textbf{if } \textit{limit} = 0 \textbf{ then return } \mathit{cutoff} \\
>&\quad \mathit{cutoffOccurred} \leftarrow \text{false} \\
>&\quad \textbf{for each } \text{action } a \textbf{ in } \textit{problem}.\textsf{Actions}(n.\textit{State}) \textbf{ do} \\
>&\quad\quad n' \leftarrow \textsf{ChildNode}(\textit{problem}, n, a) \\
>&\quad\quad \textit{result} \leftarrow \textsf{RecursiveDLS}(n', \textit{problem}, \textit{limit} - 1) \\
>&\quad\quad \textbf{if } \textit{result} = \mathit{cutoff} \textbf{ then } \mathit{cutoffOccurred} \leftarrow \text{true} \\
>&\quad\quad \textbf{else if } \textit{result} \neq \textit{unsolvable} \textbf{ then return } a \circ \textit{result} \\
>&\quad \textbf{if } \mathit{cutoffOccurred} \textbf{ then return } \mathit{cutoff} \\
>&\quad \textbf{else return } \textit{unsolvable}
>&\end{aligned}
>$$

>[!tldr]
>For each limit ($l=0,1,2,\dots$) execute a DLS (DFS limited by $l$). If you encounter a goal state stop, if you obtain a `cutoff` go to the next limit.
>
>It has no memory so for each limit it generates the whole graph again.
>
>IDS combines the advantages of breadth-first and depth-first search. It is the preferred blind search method in large state spaces with unknown plan depth.
>
>By using IDS you get the shortest path to goal state, but it doesn’t mean it’s the cheapest one.

#### Guarantees and complexity

>[!question] Guaranees
>- Is it optimal? Yes (for unit cost, extension to general action cost possible)
>- Is it complete? Yes

Time complexity:
$$
(d)b + (d - 1)b^2 +\dots+ 3b^{d-2} + 2b^{d-1} + 1b^{d}\in O(b^{d})
$$

>[!tldr]
>- $(d)b$ → the first layer (has at max $b$ nodes) is generated $d$ times
>- $(d - 1)b^2$ → the second layer (has at max $b^{2}$ nodes) is generated $d-1$ times
>- $\dots$
>- $3b^{d-2}$ → the $d-2$-th level (has at max $d-2$ nodes) is generated $3$ times
>- $2b^{d-1}$ → the $d-1$-th level (has at max $d-1$ nodes) is generated $2$ times
>- $1b^{d}$ → the $d$-th level (has at max $d$ nodes) is generated $1$ times (only in the last iteration)
>
>>[!info]
>>This algorithm has the same time complexity as the BFS

### Unit-cost search

>[!important] Pseudocode
>$$
>\begin{aligned}
>&\textbf{function } \textsf{UniformCostSearch}(\textit{problem}) \textbf{ returns a plan, or unsolvable} \\
>&\quad \textit{node} \leftarrow \text{a node } n \text{ with } n.\textit{State} = \textit{problem}.\textit{InitialState} \\
>&\quad \textit{open} \leftarrow \text{a priority queue ordered by ascending } g,\text{ only element } n \\
>&\quad \textit{closed} \leftarrow \text{empty set of states} \\
>&\quad \textbf{loop do} \\
>&\quad\quad \textbf{if } \textsf{Empty?}(\textit{open}) \textbf{ then return } \textit{unsolvable} \\
>&\quad\quad n \leftarrow \textsf{Pop}(\textit{open}) \\
>&\quad\quad \textbf{if } \textit{problem}.\textsf{GoalTest}(n.\textit{State}) \textbf{ then return } \textsf{Plan}(n) \\
>&\quad\quad \textit{closed} \leftarrow \textit{closed} \cup \{n.\textit{State}\} \\
>&\quad\quad \textbf{for each } \text{action } a \textbf{ in } \textit{problem}.\textsf{Actions}(n.\textit{State}) \textbf{ do} \\
>&\quad\quad\quad n' \leftarrow \textsf{ChildNode}(\textit{problem}, n, a) \\
>&\quad\quad\quad \textbf{if } n'.\textit{State} \notin \textit{closed} \cup \textsf{States}(\textit{open}) \textbf{ then} \\
>&\quad\quad\quad\quad \textsf{Insert}(n', g(n'), \textit{open}) \\
>&\quad\quad\quad \textbf{else if } \exists n'' \in \textit{open} \textbf{ such that } n''.\textit{State} = n'.\textit{State} \textbf{ and } g(n') < g(n'') \textbf{ then} \\
>&\quad\quad\quad\quad \textbf{replace } n'' \textbf{ in } \textit{open} \textbf{ with } n'
>\end{aligned}
>$$

>[!tldr]
>Duplicates in open list replaced in case of cheaper path.
>
>
>```
>if n'.State ∉ closed ∪ States(open) then
>    Insert(n', g(n'), open)
>```
>
>State $n'$ is neither in the already expanded states (`closed`) neither in the ones that are waiting to be expanded (`States(open)`), so add it to the `open` list
>
>```
>else if ∃ n'' ∈ open such that n''.State = n'.State and g(n') < g(n'') then
>    replace n'' in open with n'
>```
>
>If there is another node `n''` with the same state as `n'` waiting to be expanded (in `open`), but the cost from $I$ to `n'` is less then the cost from $I$ to `n''`, replace `n''` with `n'` in the `open` list


>[!example]
>![[Pasted image 20261022183633.png|300]]
>
>| Step | Expanded node     | Generated nodes                                          | Open list (after step)           |
>| ---- | ----------------- | -------------------------------------------------------- | -------------------------------- |
>| 0    | (start)           | (initial node)                                           | [Sibiu (0)]                      |
>| 1    | Sibiu             | Rimnicu (g=80), Fagaras (g=99)                           | [Rimnicu (80), Fagaras (99)]     |
>| 2    | Rimnicu           | Pitesti (g=80+97=177)                                    | [Fagaras (99), Pitesti (177)]    |
>| 3    | Fagaras           | Bucharest (g=99+211=**310**)                             | [Pitesti (177), Bucharest (310)] |
>| 4    | Pitesti           | Bucharest (g=177+101=**278**) → replace the one with 310 | [Bucharest (278)]                |
>| 5    | Bucharest (g=278) | positive goal test, stop                                 |                                  |

#### Guarantees and complexity
- *completeness* → yes, thanks to duplicate elimination (each state is expanded at most one time, no infinite cycles) and our assumption that the state space is finite
- *time complexity* → $O(b^{1+\lfloor g^{*}/\epsilon \rfloor})$ where $\epsilon$ is the cost of the cheapest action
- *space complexity* → same as time complexity

In the worst case, UCS expands all nodes with cost $\le g^*$. Every action costs at least $\epsilon$, so a path of cost $\le g^*$ contains at most $\lfloor g^*/\epsilon \rfloor$ actions. The explored tree therefore has depth at most $\lfloor g^*/\epsilon \rfloor$.

The "+1" in the exponent is the same one that appears in BFS when the goal test is applied at expansion time ($O(b^{d+1})$): to expand the nodes of the last level, their children are generated one level further down. Hence:

$$
O\!\left(b^{\,1+\lfloor g^*/\epsilon \rfloor}\right)
$$

where $b$ is the maximal branching factor, $g^*$ is the cost of an optimal plan, and $\epsilon > 0$ is the minimal action cost.

>[!example]-
>![[IMG_3740.jpeg|center|350]]
>
>$$b^{\,1+\lfloor g^*/\epsilon \rfloor}=2^{1+\lfloor 5/1 \rfloor }=2^{6}$$
>
>The complexity for the two graphs is the same. The complexity is the worst scenario, with a tree without duplicates. This can’t be $b^{d}$ as before because $d=1$ in this case.

>[!info] Lemma
>Uniform-cost search is equivalent to Dijkstra’s algorithm on the state space graph.
>
>(Obvious from the definition of the two algorithms.)
>
>>[!warning] Differences
>>The only differences are:
>>1. we generate only a part of that graph incrementally, whereas Dijkstra inputs and processes the whole graph
>>2. we stop when we expand any goal state (rather than a fixed target state given in the input)

>[!info] Theorem
>Uniform-cost search is optimal.
>
>(Because Dijkstra’s algorithm is optimal.)

