---
Class: "[[Artificial Intelligence]]"
Sem: WiSe 26/27
---
---
## Index
- [[#Forward search|Forward search]]
- [[#Backward search|Backward search]]
	- [[#Backward search#STRIPS|STRIPS]]
- [[#Pro and contra|Pro and contra]]
- [[#How to relax in planning?|How to relax in planning?]]
---
## Introduction

>[!info] **Definition** (search space)
>A **search space** is defined by the following three operations:
>- `start()` → generate the start search state
>- `is-target(s)` → test whether a given search state is a target state
>- `succ(s)` → generates the successor states $(a,s')$ of the search state $s$, along with the actions through which they are reached

>[!warning]
>Different search frameworks instantiate this template in different ways. Here we specify forward search and backward search.
>
>There are other search spaces for symbolic planning (in particular constraint-based) but we do not consider these here.

---
## Forward search

>[!info] **Definition** (Search Space for Forward Search)
>Let $\Pi=(P,A,c,I,G)$ be a STRIPS planning task. The **forward search space** of $\Pi$ is given by:
>- $\text{start}()=I$
>- $\text{is-target}(s)=\begin{cases}\textbf{true} & \text{if } G\subseteq s \\ \textbf{false} & \text{otherwise}\end{cases}$
>- $\text{succ}(s)=\{(a,s')\mid \Theta_{\Pi} \text{ has the transition } s\xrightarrow{a}s'\}$
>
>The *same definition applies to FDR* tasks $\Pi=(V,A,c,I,G)$.

Start from initial state, and apply actions until a goal state is reached.

---
## Backward search

>[!info] **Definition** (Search Space for Backward Search)
>Let $\Pi=(P,A,c,I,G)$ be a STRIPS planning task. The **backward search space** of $\Pi$ is given by:
>- $\text{start}()=G$
>- $\text{is-target}(g)=\begin{cases}\textbf{true} & \text{if } g\subseteq I \\ \textbf{false} & \text{otherwise}\end{cases}$
>- $\text{succ}(g)=\{(a,g')\mid g'=regr(g,a)\}$
>
>The *same definition* applies to FDR tasks $\Pi=(V,A,c,I,G)$.

Start at goal, and **regress** over actions to produce **subgoals**, until a subgoal is contained in (= is true in) the initial state. If $g'=regr(g,a)$, then for all $s'$ with $g'\subseteq s'$ we have $g\subseteq s'[\![ a]\!]$.

>[!tip] Intuition
>The idea is that $a$ can make the conjunctive subgoal $g$ true if (1) it achieves part of $g$; (2) it contradicts none of $g$ (cannot make any part of $g$ false); and (3) the new subgoal we would have to solve is not self-contradictory.

>[!info] **Definition** (FDR regression)
>Let $(V,A,c,I,G)$ be an FDR planning task, $g$ be a partial variable assignment, and $a\in A$. We say that $g$ is **regressable** by $a$ if:
>1. $e\hspace{-0.1em}f\hspace{-0.1em}f_{a}\cap g\neq \varnothing$
>2. there is no $v\in V$ s.t. $v\in Vars[e\hspace{-0.1em}f\hspace{-0.1em}f_{a}]\cup Vars[g]$ and $e\hspace{-0.1em}f\hspace{-0.1em}f_{a}(v)\neq g(v)$
>3. there is no $v\in V$ s.t. $v\not\in Vars[e\hspace{-0.1em}f\hspace{-0.1em}f_{a}]$, $v\in Vars[pre_{a}]\cap Vars[g]$ and $pre_{a}(v)\neq g(v)$
>
>In that case, the *regression* of $g$ via $a$ is **$regr(g,a)=(g\backslash e\hspace{-0.1em}f\hspace{-0.1em}f_{a})\cup pre_{a}$**; else $regr(g,a)$ is undefined, written $regr(g,a)=\bot$

>[!info] Capiamo
>![[bangdesk.gif]]
>
>1. deve aggiungere qualcosa per raggiungere $g$
>2. ciò che l’azione cambia, non deve cambiarlo nel valore sbagliato (altrimenti dopo l’azione l’obiettivo è falso)
>3. se un’azione ha bisogno di qualcosa e non lo cambia, quel qualcosa sarà così anche dopo; non può contraddire ciò che vuoi alla fine

>[!example] Backward Search on "TSP"
>![[tsp.png|400]]
>
>- $V$: $at: \{Sy, Ad, Br, Pe, Da\}$; $v_{x}: \{T,F\}$ for $x\in\{Sy, Ad, Br, Pe, Da\}$
>- $A$: $drive_{xy}$ where $x$, $y$ have a road
>- $I$: $at=Sy$, $v_{Sy}=T$, $v_{x}=F$ for $x\neq Sy$
>- $G$: $at=Sy$, $v_{x}=T$ for all $x$
>
>Regression tree (writing $v_{x}$ for $v_{x}=T$; edge $x\to y$ = $drive_{xy}$):
>```mermaid
>graph TD
>    R["{at=Sy, v_Sy, v_Br, v_Ad, v_Pe, v_Da}"]
>    N1["{at=Br, v_Br, v_Ad, v_Pe, v_Da}"]
>    N2["{at=Ad, v_Br, v_Ad, v_Pe, v_Da}"]
>    N3["{at=Sy, v_Ad, v_Pe, v_Da}"]
>    N4["{at=Br, v_Ad, v_Pe, v_Da}"]
>    N5["{at=Ad, v_Ad, v_Pe, v_Da}"]
>    N6["{at=Da, v_Br, v_Pe, v_Da}"]
>    N7["{at=Pe, v_Br, v_Pe, v_Da}"]
>    N8["{at=Sy, v_Br, v_Pe, v_Da}"]
>    N9["{at=Ad, v_Br, v_Pe}"]
>    N10["{at=Ad, v_Br, v_Da}"]
>    N11["{at=Br, v_Br, v_Pe, v_Da}"]
>    N12["{at=Ad, v_Br, v_Pe, v_Da}"]
>
>    R -->|Br→Sy| N1
>    R -->|Ad→Sy| N2
>    N1 -->|Sy→Br| N3
>    N3 -->|Br→Sy| N4
>    N3 -->|Ad→Sy| N5
>    N2 -->|Da→Ad| N6
>    N2 -->|Pe→Ad| N7
>    N2 -->|Sy→Ad| N8
>    N6 -->|Ad→Da| N9
>    N7 -->|Ad→Pe| N10
>    N8 -->|Br→Sy| N11
>    N8 -->|Ad→Sy| N12
>```
>
>>[!warning]
>>![[bangdesk.gif]]
>>
>>Non necessariamente quando facciamo la regression dell’azione Br→Sy significa che esattamente in quella mossa avevamo visitato Sy (potrebbe esser già stata visitata da azioni precedenti), ma essendo in backward search non ci interessa e ci basta controllare che $v_{Sy}$ è un effetto dell’azione Br→Sy

### STRIPS

>[!info] **Definition** (STRIPS regression)
>Let $(P,A,c,I,G)$ be a STRIPS planning task, $g\subseteq P$, and $a\in A$. We say that $g$ is regressable over $a$ if:
>- $add_{a}\cap g\neq \varnothing$
>- $del_{a}\cap g=\varnothing$
>
>In that case, the *regression* of $g$ over $a$ is $regr(g,a)=(g\backslash add_{a})\cup pre_{a}$; else $regr(g,a)$ is undefined, written $regr(g,a)=\bot$

>[!warning] Note the difference to FDR
>In (2), instead of “contradicting variable values” we only look at the action’s immediate deletes. This is weaker because we fail to see, e.g., that different truck positions yield contradiction as well.
>
>(3) here is missing completely because in STRIPS there is no possibility for a subgoal to be “self-contradictory”. This is weaker because we fail to see, e.g., that subgoal requiring several different truck positions are self-contradictory.
>
>FDR backward search is a lot more effective than STRIPS backward search, because its search space contains less unreachable states.

>[!example] Backward Search on "TSP" (STRIPS)
>![[tsp.png|400]]
>
>- $P$: $at_{x}$ for $x\in\{Sy, Ad, Br, Pe, Da\}$; $v_{x}$ for $x\in\{Sy, Ad, Br, Pe, Da\}$
>- $A$: $drive_{xy}$ where $x$, $y$ have a road
>- $I$: $at_{Sy}, v_{Sy}$
>- $G$: $at_{Sy}, v_{x}$ for all $x$
>
>Regression tree (edge $x\to y$ = $drive_{xy}$; red = subgoals with two $at$ facts, i.e. the truck in two places at once):
>```mermaid
>graph TD
>    R["{at_Sy, v_Sy, v_Br, v_Ad, v_Pe, v_Da}"]
>    N1["{at_Br, v_Br, v_Ad, v_Pe, v_Da}"]
>    N2["{at_Ad, v_Br, v_Ad, v_Pe, v_Da}"]
>    N3["{at_Sy, at_Ad, v_Sy, v_Br, v_Ad, v_Da}"]
>    N4["{at_Sy, at_Ad, v_Sy, v_Br, v_Ad, v_Pe}"]
>    N5["{at_Sy, at_Pe, v_Sy, v_Br, v_Pe, v_Da}"]
>    N6["{at_Sy, at_Da, v_Sy, v_Br, v_Pe, v_Da}"]
>    D["..."]
>
>    R -->|Br→Sy| N1
>    R -->|Ad→Sy| N2
>    R -->|Ad→Pe| N3
>    R -->|Ad→Da| N4
>    R -->|Pe→Ad| N5
>    R -->|Da→Ad| N6
>    N3 -.-> D
>    N4 -.-> D
>    N5 -.-> D
>
>    classDef bad fill:#fdd,stroke:#c00,color:#c00
>    class N3,N4,N5,N6,D bad
>```

---
## Pro and contra
The two frameworks have dual strengths and weaknesses.

>[!example]
>![[Pasted image 20261109095205.png]]
>
>Does forward search explore $s7$? No
>Does forward search explore $s2$? Yes
>
>>[!warning]
>>Forward search explores only reachable states, but may explore unsolvable ones.
>
>Does backward search explore $s7$? Yes
>Does backward search explore $s2$? No
>
>>[!warning]
>>Backward search explores only solvable states, but may explore unreachable ones.

>[!tip] Claim: forward search is hopeless if there are too many options
>![[Pasted image 20261109095616.png|480]]
>
>Yes, but only if we do blind search. Heuristic search tends to het rid of irrelevant options easily

>[!question] So which one should we use?
>**In favor of forward search**:
>Backward search has in the past often had serous trouble getting lost in gazillions of solvable but unreachable states. Forward search allows easy formulation of searches for more complex forms of sequential decision making (e.g. uncertainty, games). Basically all current heuristic search planners, including Fast Downward, use forward search.
>
>We focus on forward search throughout this course.
>
>**That said**: Forward search and backward search are incomparable, neither dominates the other. Which one works better in a given task depends on a lot of factors, and there is no comprehensive understanding of this.

---
## How to relax in planning?

![[Pasted image 20261109100734.png]]

1. you have a class **$\mathcal{P}$** of problems, whose perfect heuristic **$h^{*}_{\mathcal{P}}$** you wish to estimate
2. you define a class **$\mathcal{P}^{r}$** of simpler problems, whose perfect heuristic **$h^{*}_{\mathcal{P}^{r}}$** can be used to estimate **$h^{*}_{\mathcal{P}}$**
3. you define a transformation, **the relaxation mapping $\mathcal{R}$**, that maps instances $\Pi \in \mathcal{P}$ into instances $\Pi^{r}\in \mathcal{P}^{r}$

Given $\Pi \in\mathcal{P}$, you let $\Pi^{r}:=\mathcal{R}(\Pi)$, and estimate $h^{*}_{\mathcal{P}}(\Pi)$ by $h^{*}_{\mathcal{P}^{r}}(\Pi^{r})$.

>[!warning]
>Search uses the real (un-relaxed) $\Pi$. The relaxation is applied **only within the call to $h(s)$**.
>
>![[Pasted image 20261109101525.png]]
>
>Here $\Pi_{s}$ is $\Pi$ with initial state replaced by $s$, i.e., $\Pi=(P,A,I,G)$ changed to $(P,A,s,G)$; the task of finding a plan for reach state $s$.
>
>A common student mistake is to instead apply the relaxation once to the whole problem, then doing the whole search “within the relaxation”.

>[!example]
>![[Pasted image 20261109101400.png|500]]
>
>- **Problem class $\mathcal{P}$**: Route finding
>- **Perfect heuristic $h^{*}_{\mathcal{P}}$ for $\mathcal{P}$**: Length of a shortest route
>- **Simpler problem class $\mathcal{P}^{r}$**: Route finding on an empty map
>- **Perfect heuristic $h^{*}_{\mathcal{P}^{r}}$ for $\mathcal{P}^{r}$**: Straight-line distance
>- **Transformation $\mathcal{R}$**: Throw away the map

>[!example] A Simple Planning Relaxation: Only-Adds ("Logistics")
>![[logistics.png|300]]
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
>"$drive_{xy}: \Rightarrow truck_{y}$"; "$load_{x}: \Rightarrow pack_{T}$"; "$unload_{x}: \Rightarrow pack_{x}$"
>
>>[!question] Heuristic value for $I$?
>>$1$: a plan for the relaxed task is $\langle unload_{D}\rangle$.

>[!example] How to Relax During Search: Only-Adds
>**Greedy best-first search** (tie-breaking: alphabetic) on the *real* problem; the relaxation is used only to compute $h^{\mathcal{R}}(s)$ for each generated state.
>
>Notation: state $XY$ = truck at $X$, package at $Y$ ($T$ = in the truck). $I = AC$, $G = AD$. Actions: $drXY$, $loX$, $ulX$.
>
>| Step | Expanded state | Generated successors ($h^{\mathcal{R}}$ / relaxed plan) |
>|---|---|---|
>| 0 | — | $AC$: $h=1$, $\langle ulD\rangle$ |
>| 1 | $AC$ | $\xrightarrow{drAB} BC$: $h=2$, $\langle drBA, ulD\rangle$ |
>| 2 | $BC$ | $\xrightarrow{drBC} CC$: $h=2$, $\langle drBA, ulD\rangle$; $\xrightarrow{drBA} AC$: duplicate, prune |
>| 3 | $CC$ | $\xrightarrow{drCD} DC$: $h=2$; $\xrightarrow{loC} CT$: $h=2$; $\xrightarrow{drCB} BC$: duplicate, prune |
>| 4 | $CT$ | $\xrightarrow{drCB} BT$: $h=2$; $\xrightarrow{drCD} DT$: $h=2$; $\xrightarrow{ulC} CC$: duplicate, prune |
>| 5 | $BT$ | $\xrightarrow{drBA} AT$: $h=1$; $\xrightarrow{ulB} BB$: $h=2$; $\xrightarrow{drBC} CT$: duplicate, prune |
>| 6 | $AT$ | $\xrightarrow{ulA} AA$: $h=1$; $\xrightarrow{drAB} BT$: duplicate, prune |
>| 7 | $AA$ | $\xrightarrow{drAB} BA$: $h=2$; $\xrightarrow{loA} AT$: duplicate, prune |
>| 8 | $BA$ | $\xrightarrow{drBC} CA$: $h=2$; $\xrightarrow{drBA} AA$: duplicate, prune |
>| … | … | … |
>
>```mermaid
>graph LR
>    AC["AC (h=1)"] -->|drAB| BC["BC (h=2)"]
>    BC -->|drBC| CC["CC (h=2)"]
>    BC -->|drBA| D1["AC (dup)"]
>    CC -->|drCD| DC["DC (h=2)"]
>    CC -->|loC| CT["CT (h=2)"]
>    CC -->|drCB| D2["BC (dup)"]
>    CT -->|drCB| BT["BT (h=2)"]
>    CT -->|drCD| DT["DT (h=2)"]
>    CT -->|ulC| D3["CC (dup)"]
>    BT -->|drBA| AT["AT (h=1)"]
>    BT -->|ulB| BB["BB (h=2)"]
>    BT -->|drBC| D4["CT (dup)"]
>    AT -->|ulA| AA["AA (h=1)"]
>    AT -->|drAB| D5["BT (dup)"]
>    AA -->|drAB| BA["BA (h=2)"]
>    AA -->|loA| D6["AT (dup)"]
>    BA -->|drBC| CA["CA (h=2)"]
>    BA -->|drBA| D7["AA (dup)"]
>    CA -.-> MORE["..."]
>
>    classDef dup fill:#fdd,stroke:#c00,color:#c00
>    class D1,D2,D3,D4,D5,D6,D7 dup
>```
>
>>[!warning] Observation
>>The heuristic misleads the search: $AT$ and $AA$ get $h=1$ even though the package is moving *away* from $D$. Only-adds ignores preconditions, so "$ulD$" always looks like one step away as long as the package isn't at $D$, regardless of where truck and package actually are.
>
>>[!info] Capiamo
>>Le azioni applicate sono quelle del problema originale, non quelle del problema rilassato (devono rispettare $pre_{a}$ e $del_{a}$)


