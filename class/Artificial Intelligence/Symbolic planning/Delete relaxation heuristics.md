---
Class: "[[Artificial Intelligence]]"
Sem: WiSe 26/27
---
---
## Index
- [[#The delete relaxation|The delete relaxation]]
	- [[#The delete relaxation#State dominance|State dominance]]
	- [[#The delete relaxation#Greedy Relaxed Planning|Greedy Relaxed Planning]]
- [[#$h^{+}$ (optimal delete relaxation heuristic)|$h^{+}$ (optimal delete relaxation heuristic)]]
	- [[#$h^{+}$ (optimal delete relaxation heuristic)#How to compute $h^{+}$|How to compute $h^{+}$]]
- [[#$h^{add}$ and $h^{max}$|$h^{add}$ and $h^{max}$]]
	- [[#$h^{add}$ and $h^{max}$#Properties|Properties]]
	- [[#$h^{add}$ and $h^{max}$#Summary|Summary]]
- [[#Relaxed plan heuristic|Relaxed plan heuristic]]
	- [[#Relaxed plan heuristic#The best-supporter functions we will use|The best-supporter functions we will use]]
		- [[#The best-supporter functions we will use#Correctness|Correctness]]
	- [[#Relaxed plan heuristic#Relaxed plan extraction|Relaxed plan extraction]]
		- [[#Relaxed plan extraction#Correctness|Correctness]]
	- [[#Relaxed plan heuristic#Best-supporter functions|Best-supporter functions]]
	- [[#Relaxed plan heuristic#Relaxed plan heuristic|Relaxed plan heuristic]]
		- [[#Relaxed plan heuristic#Properties|Properties]]
- [[#Delete relaxation for FDR|Delete relaxation for FDR]]
---
## The delete relaxation

>[!info] **Definition** (delete relaxation)
>1. For a STRIPS action $a$, by $a^{+}$ we denote the corresponding **delete relaxed action**, or short *relaxed action*, defined by $pre_{a^{+}}:=pre_{a}$, $add_{a^{+}}:=add_{a}$, and $del_{a^{+}}:=\varnothing$
>2. For a set $A$ of STRIPS actions, by $A^{+}$ we denote the corresponding set of relaxed actions, $A^{+}:=\{a^{+}\mid a\in A\}$; similarly, for a sequence $\overrightarrow{a}=\langle a_{1}^{+},\dots,a^{+}_{n} \rangle$ of STRIPS actions, by $\overrightarrow{a^{+}}$ we denote the corresponding sequence of relaxed actions, $\overrightarrow{a^{+}}:=\langle a_{1}^{+},\dots,a_{n}^{+}\rangle$
>3. For a STRIPS planning task $\Pi=(P,A,c,I,G)$ by $\Pi^{+}:=(P,A^{+},c,I,G)$ we denote the corresponding **(delete) relaxed planning task**

>[!info] **Definition** (relaxed plan)
>Let $\Pi=(P,A,c,I,G)$ be a STRIPS planning task, and let $s$ be a state. An (optimal) **relaxed plan for $s$** is an (optimal) plan for $\Pi^{+}_{s}=(P,A,c,s,G)$. A relaxed plan for $I$ is also called relaxed plan for $\Pi$.

>[!example] Relaxed dompteur
>- $P = \{alive,\ haveTiger,\ tamedTiger,\ haveJump\}$
>  - Short: $P = \{A,\ hT,\ tT,\ J\}$
>- **Initial state** $I$: $alive$
>- **Goal** $G$: $alive,\ haveJump$
>- **Actions** $A$:
>
>| Action | pre | add | del |
>|---|---|---|---|
>| $getTiger$ | $alive$ | $haveTiger$ | — |
>| $tameTiger$ | $alive,\ haveTiger$ | $tamedTiger$ | — |
>| $jumpTamedTiger$ | $alive,\ tamedTiger$ | $haveJump$ | — |
>| $jumpTiger$ | $alive,\ haveTiger$ | $haveJump$ | <span style="color:red">$alive$</span> |
>
>>[!done] Relaxed plan for this task
>>$\langle getTiger,\ jumpTiger \rangle$
>>
>>$\langle getTiger,\ tameTiger,\ jumpTamedTiger \rangle$ works as well, but the previous one is "better" :-)

### State dominance

>[!info] **Definition** (dominance)
>Let $\Pi=(P,A,c,I,G)$ be a STRIPS planning tsk, and let $s$, $t$ be states. We say that $t$ **dominates** $s$ if $s\subseteq t$
>
>>[!tldr]
>>Dominance = “more facts true”

>[!info] **Lemma** (dominance)
>Let $\Pi=(P,A,c,I,G)$ be a STRIPS planning task, and let $s$, $t$ be state where **$t$ dominates $s$**. We have:
>1. if $s$ is a goal state, then $t$ is a goal state as well
>2. if $\overrightarrow{a}$ is applicable in $s$, then $\overrightarrow{a}$ is applicable in $t$ as well, and $t[\![\overrightarrow{a}]\!]$ dominates $s[\![\overrightarrow{a}]\!]$
>
>>[!done] Proof
>>(1) is trivial. (2) by induction over the length $n$ of $\overrightarrow{a}$; base case $n=0$ is trivial, inductive case $n\to n+1$ follows directly from induction hypothesis and the definition of $s[\![a]\!]$
>
>>[!tldr]
>>It is always better to have more facts true
>
>^3ca418

>[!info] **Lemma**
>Let $\Pi=(P,A,c,I,G)$ be a STRIPS planning task. Let $s$ be a state and let $a \in A$ be applicable in $s$. Then:
>1. $s[\![a^{+}]\!]$ dominates $s$
>2. if $t$ dominates $s$, then $t[\![a^{+}]\!]$ dominates $s[\![a]\!]$
>
>>[!done] Proof
>>Trivial
>
>>[!tip] Ergo
>>Any real plan also works in the relaxed world
>
>^506c9f

>[!info] **Proposition** (delete relaxation is over-approximating)
>Let $\Pi=(P,A,c,I,G)$ be a STRIPS planning task, let $s$ be a state, and let $\overrightarrow{a}$ be a plan for $\Pi_{s}$. Then $\overrightarrow{a^{+}}$ is a relaxed plan for $s$.
>
>>[!done] Proof
>>Prove by induction over the length of $\overrightarrow{a}$ that $s[\![\overrightarrow{a^{+}}]\!]$ dominates $s[\![\overrightarrow{a}]\!]$; base case is trivial, inductive case follows from (2) above
>
>>[!tip] Ergo
>>We can easily find a relaxed (or prove that none exists)
>>- applying a relaxed action can only ever make more facts true ((1) above)
>>- that cannot render the task unsolvable ([[#^3ca418|Lemma]])

### Greedy Relaxed Planning

>[!important] Pseudocode
>$$
>\begin{array}{l} s^+ := s;\ \vec{a}^+ := \langle\rangle \\ \textbf{while } G \not\subseteq s^+ \textbf{ do}: \\ \quad \textbf{if } \exists a \in A \text{ s.t. } pre_a \subseteq s^+ \text{ and } s^+[\![a^+]\!]  \neq s^+ \quad {\color{red}/\!*\ \text{i.e. } add_a \not\subseteq s^+\ *\!/} \textbf{ then} \\ \quad\quad \text{select one such } a \\ \quad\quad s^+ := s^+[\![a^+]\!];\ \vec{a}^+ := \vec{a}^+ \circ \langle a^+ \rangle \\ \quad \textbf{else return } \text{``}\Pi^+_s \text{ is unsolvable''} \textbf{ endif} \\ \textbf{endwhile} \\ \textbf{return } \vec{a}^+ \end{array}
>$$

>[!info] **Proposition**
>Greedy relaxed planning is **sound**, **complete**, and **terminates in time polynomial** in the size of $\Pi$.
>
>>[!done] Proof
>>- **Soundness:** if $\vec{a}^+$ is returned then, by construction, $G \subseteq s[\![\vec{a}^{+}]\!]$.
>>- **Termination:** every $a \in A$ can be selected at most once, because afterwards $add_a \subseteq s^+$, so $s^+ s[\![a^{+}]\!]= s^+$. Hence at most $|A|$ iterations.
>>- **Completeness:** if "$\Pi^+_s$ is unsolvable" is returned, then no plan exists in $\Pi^+$ for $s^+$ (no applicable action adds anything new, and $G \not\subseteq s^+$). As $s^+$ dominates $s$, no plan can exist in $\Pi^+$ for $s$ either ([[#^3ca418|Lemma]]).

However if we use greedy relaxed planning to generate $h$ (set $h(s)$ to the cost of $\vec{a}^{+}$, or $\infty$ if “$\Pi_{s}^{+}$ is unsolvable is returned“), we may end up selecting **arbitrary actions that aren’t relevant at all** (over-estimating dramatically, not admissible).

To be accurate, a heuristic needs to approximate the *minimum* effort needed to reach the goal.

There is also an issue of “brittleness”; indeed greedy relaxed planning may give drastically different values for very similar states (and even fro the same state if called twice). This is bound to be detrimental for search guidance.

---
## $h^{+}$ (optimal delete relaxation heuristic)

>[!info] **Definition** ($h^{+}$)
>Let $\Pi=(P,A,c,I,G)$ be a STRIPS planning task with state space $\Theta_{\Pi}=(S,A,c,T,I,G)$. The **optimal delete relaxation heuristic $h^{+}$** for $\Pi$ is the function $h^{+}:S\to\mathbb{R}^{+}_{0}\cup \{\infty\}$ where *$h^{+}(s)$ is defined as the cost of an optimal relaxed plan for $s$*
>
>>[!tldr]
>>$h^{+}$ = minimum effort to reach the goal under delete relaxation

>[!info] **Proposition** ($h^{+}$ is consistent)
>Let $\Pi=(P,A,c,I,G)$ be a STRIPS planning task. Then $h^{+}$ is consistent and goal-aware, and thus admissible and safe.
>
>>[!done] Proof
>>**Goal-awareness.** If $G \subseteq s$, the empty sequence $\langle\rangle$ is a relaxed plan for $s$, so $h^{+}(s) = 0$.
>>
>>**Consistency.** Let $a$ be applicable in $s$ and $s'=s[\![ a ]\!]$. We need to show that $h^{+}(s)\leq c(a)+h^{+}(s')$.
>>If $h^{+}(s') = \infty$ there is nothing to show. Otherwise, let $\vec{a}^{+}_{s'}$ be an optimal relaxed plan for $s'$, so $c(\vec{a}^{+}_{s'})=h^{+}(s')$.
>>
>>Construct $\vec{a}^{+}_{s}:=\langle a^{+}\rangle \circ \vec{a}^{+}_{s'}$. This is a relaxed plan for $s$:
>>1. $a^{+}$ is applicable in $s$, ($pre_{a^+} = pre_a$), then let $s'^{+} := s[\![ a^{+} ]\!]$.
>>2. $s'^{+}$ dominates $s'$ (by [[#^506c9f|Lemma (2)]] with $t = s$):
>>3. Since $\vec{a}^{+}_{s'}$ is a relaxed plan for $s'$ and $s'^{+} \supseteq s'$, by [[#^3ca418|Lemma (2)]] $\vec{a}^{+}_{s'}$ is applicable in $s'^{+}$ and $s'^{+}[\![ \vec{a}^{+}_{s'} ]\!] \supseteq s'[\![ \vec{a}^{+}_{s'} ]\!] \supseteq G$.
>>
>>Hence $h^{+}(s)\leq c(\vec{a}^{+}_{s})=c(a)+c(\vec{a}^{+}_{s'})=c(a)+h^{+}(s')$. $\square$
>>

>[!tip]
>The upper bound of $h^{+}$ is the spanning tree
>
>![[Pasted image 20261113201813.png]]

>[!example] TSP
>![[tsp.png|400]]
>
>$$h^{+}(\text{TSP})=\text{minimum spanning tree}$$

>[!example] $h^{+}$ in "Logistics"
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
>>[!info] Real vs. relaxed problem
>>- **Real problem:** actions with $pre$, $add$, ${\color{red}del}$.
>>- **Relaxed problem:** actions with $pre$, $add$ only. Driving away from $X$ does **not** delete "truck at $X$", so the truck is "everywhere it has ever been".
>>
>>$\Rightarrow$ The truck never has to drive back to $A$: "truck at $A$" stays true from the start.
>
>| State $s$ | $h^{+}(s)$ | Optimal relaxed plan (e.g.) |
>|---|---|---|
>| $AC$ | 5 | $\langle drAB, drBC, drCD, loC, ulD \rangle$ |
>| $BC$ | 5 | $\langle drBA, drBC, drCD, loC, ulD \rangle$ |
>| $CC$ | 5 | $\langle drCB, drBA, drCD, loC, ulD \rangle$ |
>| $DC$ | 5 | $\langle drDC, drCB, drBA, loC, ulD \rangle$ |
>| $CT$ | 4 | $\langle drCB, drBA, drCD, ulD \rangle$ |
>| $BT$ | 4 | $\langle drBA, drBC, drCD, ulD \rangle$ |
>| $AT$ | 4 | $\langle drAB, drBC, drCD, ulD \rangle$ |
>| $DT$ | 4 | $\langle ulD, drDC, drCB, drBA \rangle$ |
>| $DD$ | 3 | $\langle drDC, drCB, drBA \rangle$ |
>| $CD$ | 2 | $\langle drCB, drBA \rangle$ |
>| $BD$ | 1 | $\langle drBA \rangle$ |
>| $AD$ | 0 | $\langle\rangle$ (goal) |
>
>```mermaid
>graph LR
>    AC["AC (5)"] -->|drAB| BC["BC (5)"]
>    BC -->|drBC| CC["CC (5)"]
>    CC -->|loC| CT["CT (4)"]
>    CC -->|drCD| DC["DC (5)"]
>    CC -.->|drCB| D1["BC: D"]
>    CT -->|drCD| DT["DT (4)"]
>    CT -->|drCB| BT["BT (4)"]
>    CT -.->|ulC| D2["CC: D"]
>    BT -->|drBA| AT["AT (4)"]
>    BT -->|ulB| BB["BB (5)"]
>    BT -.->|drBC| D3["CT: D"]
>    AT -->|ulA| AA["AA (5)"]
>    AT -.->|drAB| D4["BT: D"]
>    DT -->|ulD| DD["DD (3)"]
>    DT -.->|drDC| D5["CT: D"]
>    DD -->|drDC| CD["CD (2)"]
>    DD -.->|loD| D6["DT: D"]
>    CD -->|drCB| BD["BD (1)"]
>    CD -.->|drCD| D7["DD: D"]
>    BD -->|drBA| AD["AD (0) ✓"]
>    BD -.->|drBC| D8["CD: D"]
>```
>(number in brackets = $h^{+}$, **D** = duplicate state, pruned. dashed edges = pruned duplicates)
>
>**Expansion order:** $AC \to BC \to CC \to CT \to BT \to AT \to DT \to DD \to CD \to BD \to AD$

>[!example] $h^{+}$ in the Blocksworld
>![[Pasted image 20261113203332.png|center|350]]
>
>- ${\text{Optimal plan:}}$ $\langle putdown\_A,\ unstack\_BD,\ stack\_BC,\ pickup\_A,\ stack\_AB \rangle$
>- ${\text{Optimal relaxed plan:}}$ $\langle stack\_AB,\ unstack\_BD,\ stack\_BC \rangle$
>
>>[!question] What can we say about the "search space surface" at the initial state?
>>![[Pasted image 20261113204053.png]]
>>
>>The initial state lies on a **local minimum** under $h^{+}$, together with the successor state $s$ where we stacked $A$ onto $B$ ($h^{+}(I) = h^{+}(s) = 3$).
>>All other direct neighbors of these two states have a **strictly higher** $h^{+}$ value.
>>
>>In this case applying the plain greedy approach would lead to an issue

### How to compute $h^{+}$

>[!info] **Definition** (PlanOpt⁺)
>By **$\text{PlanOpt}^{+}$**, we denote the problem of deciding, given a STRIPS planning task $\Pi=(P,A,c,I,G)$ and $B\in \mathbb{R}_{0}^{+}$, whether there exists a relaxed plan for $\Pi$ whose cost is at most $B$.
>
>>[!tldr]
>>By computing $h^{+}$, we would solve $\text{PlanOpt}^{+}$

>[!info] **Theorem** (Optimal Relaxed Planning is Hard)
>$\text{PlanOpt}^{+}$ is **NP**-complete.
>
>>[!done] Proof
>>**Membership in $NP$.** In a relaxed plan every action is needed at most once (applying it again adds nothing new), so an optimal relaxed plan has length $\le |A|$. Guess an action sequence of length $\le |A|$ and check in polynomial time that it is a relaxed plan with cost $\le B$.
>>
>>**Hardness.** By reduction from SAT. I.e. $\varphi$ è soddisfacibile $\iff$ $\Pi_{\varphi}$ ha un piano rilassato con costo $\le B$
>>
>>>[!example] Example: $X_1 \wedge \neg X_1$ (insoddisfacibile, costo $> B$)
>>>Given a CNF formula with variables $X_1,\dots,X_n$ and clauses $C_1,\dots,C_m$, build (with unit costs):
>>>- **Actions setting variables to T:** $setT_{X_i}$: pre $\emptyset$, add $\{X_i T,\ X_i set\}$
>>>- **Actions setting variables to F:** $setF_{X_i}$: pre $\emptyset$, add $\{X_i F,\ X_i set\}$
>>>- **Actions setting clauses to T:** for every literal $X_i \in C_j$: pre $X_i T$, add $C_j T$; for every literal $\neg X_i \in C_j$: pre $X_i F$, add $C_j T$
>>>- **Goal:** $G = \{X_i set \mid 1 \le i \le n\} \cup \{C_j T \mid 1 \le j \le m\}$
>>>- $B := n + m$ (number of variables + number of clauses)
>>>
>>>Clauses $\{C_1 = \{X_1\},\ C_2 = \{\neg X_1\}\}$, $B = 1 + 2 = 3$.
>>>- $C_1 T$ needs $X_1 T$, $C_2 T$ needs $X_1 F$ $\Rightarrow$ both $setT_{X_1}$ and $setF_{X_1}$ are needed.
>>>- Cheapest relaxed plan: $\langle setT_{X_1},\ setF_{X_1},\ C_1\text{-action},\ C_2\text{-action}\rangle$, cost $4 > 3$.
>>>
>>>$\Rightarrow$ no relaxed plan with cost $\le B$, and the formula is indeed unsatisfiable.
>>>
>>>Note: in the relaxed world $X_1 T$ and $X_1 F$ can both be true (nothing is deleted), so "cheating" is possible, but it costs one more action than $B$ allows.

The delete relaxation heuristic we want is $h^{+}$. Unfortunately, this is hard to compute so the computational overhead is very likely to be prohibitive. All implemented systems using the delete relaxation approximate $h^{+}$ in one or the other way.

---
## $h^{add}$ and $h^{max}$

>[!info] **Definition** ($h^{\text{add}}$)
>Let $\Pi=(P,A,c,I,G)$ be a STRIPS planning task. The **additive heuristic $h^{\text{add}}$** for $\Pi$ is the function $h^{\text{add}}(s):=h^{\text{add}}(s,G)$ where $h^{\text{add}}(s,g)$ is the point-wise greatest function that satisfies
>$$
>h^{\text{add}}(s,g)=\begin{cases} 0 & g\subseteq s \\ \min_{a\in A,\, g'\in add_a}\left[c(a)+h^{\text{add}}(s,pre_a)\right] & g=\{g'\} \\ {\color{red}\sum_{g'\in g}}\, h^{\text{add}}(s,\{g'\}) & |g|>1 \end{cases}
>$$

>[!info] **Definition** ($h^{\text{max}}$)
>Let $\Pi=(P,A,c,I,G)$ be a STRIPS planning task. The **max heuristic $h^{\text{max}}$** for $\Pi$ is the function $h^{\text{max}}(s):=h^{\text{max}}(s,G)$ where $h^{\text{max}}(s,g)$ is the point-wise greatest function that satisfies
>$$
>h^{\text{max}}(s,g)=\begin{cases} 0 & g\subseteq s \\ \min_{a\in A,\, g'\in add_a}\left[c(a)+h^{\text{max}}(s,pre_a)\right] & g=\{g'\} \\ {\color{red}\max_{g'\in g}}\, h^{\text{max}}(s,\{g'\}) & |g|>1 \end{cases}
>$$

>[!example] $bs^{max} \neq bs^{add}$
>![[Pasted image 20261119124240.png|500]]
>- $s = \{x\}$; $G = \{p\}$.
>- $A$:
>	- $mq_1$: pre $x$; add $q_1$; cost $2$
>	- $mq_2$: pre $x$; add $q_2$; cost $2$
>	- $mr$: pre $x$; add $r$; cost $3$
>	- $a$: pre $q_1, q_2$; add $p$; cost $1$ ("assemble $p$ from two parts")
>	- $b$: pre $r$; add $p$; cost $1$ ("build $p$ from one bigger part")
>
>| | x | q₁ | q₂ | r | p |
>|---|---|---|---|---|---|
>| $h^{max}$ | 0 | 2 | 2 | 3 | 3<sub>a</sub> |
>| $h^{add}$ | 0 | 2 | 2 | 3 | 4<sub>b</sub> |
>
>Candidates for $p$:
>- via $a$: $h^{max}$: $1 + \max(2, 2) = \mathbf{3}$, $h^{add}$: $1 + (2 + 2) = 5$
>- via $b$: $h^{max}$: $1 + 3 = 4$, $h^{add}$: $1 + 3 = \mathbf{4}$
>
>→ $bs^{max}_s(p) = a$, but $bs^{add}_s(p) = b$.
>
>>[!note]- Consequence on $h^{FF}$
>>- From $bs^{max}$: $\mathit{RPlan} = \{a, mq_1, mq_2\}$ → $h^{FF}(s) = 1 + 2 + 2 = 5$
>>- From $bs^{add}$: $\mathit{RPlan} = \{b, mr\}$ → $h^{FF}(s) = 1 + 3 = 4$
>>- Here $h^{+}(s) = h^{*}(s) = 4$ (no deletes at all) → $h^{FF}$ from $bs^{max}$ **over-estimates** $h^*$: a concrete case of "$bs$ makes sub-optimal choices".
>
>>[!tldr]
>>$h^{max}$ guarda solo la precondizione più cara, come se le altre fossero gratis. Per questo preferisce l'azione con tante precondizioni economiche. $h^{add}$ le somma tutte, e preferisce l'azione con una sola precondizione un po' più costosa.

### Properties

![[Pasted image 20261113210859.png]]

>[!info] **Proposition** ($h^{\text{max}}$ is Optimistic)
>**${h^{\text{max}} \le h^{+}}$**, and thus **${h^{\text{max}} \le h^{*}}$**.
>
>>[!note] Intuition
>>$h^{\text{max}}$ does not take delete lists into account (the definition contains only $pre$ and $add$), so it approximates $h^{+}$. It simplifies relaxed planning by assuming that, to achieve a set $g$ of subgoals, it suffices to achieve the **single most costly** $g' \in g$. Actual relaxed planning, i.e. $h^{+}$, can only be more expensive.

>[!info] **Proposition** ($h^{\text{add}}$ is Pessimistic)
>For all STRIPS planning tasks $\Pi$, **${h^{\text{add}} \ge h^{+}}$**. There exist $\Pi$ and $s$ so that **${h^{\text{add}}(s) > h^{*}(s)}$**.
>
>>[!note] Intuition
>>- **First claim:** $h^{\text{add}}$ does not take delete lists into account, so it approximates $h^{+}$. It simplifies relaxed planning by assuming that, to achieve a set $g$ of subgoals, we must achieve **every** $g' \in g$ **separately**. Actual relaxed planning can only be less expensive.
>>- **Second claim:** see example on slide 29.

>[!tip] Takeaway
>$\rightarrow$ Both $h^{\text{max}}$ and $h^{\text{add}}$ approximate $h^{+}$ by assuming that singleton subgoal facts are achieved **independently**.
>- $h^{\text{max}}$ estimates *optimistically* by the most costly singleton subgoal.
>- $h^{\text{add}}$ estimates *pessimistically* by summing over all singleton subgoals.

>[!info] **Proposition** ($h^{max}$ and $h^{add}$ agree with $h^{+}$ on $\infty$)
>For all STRPIS planning tasks $\Pi$ and states $s$ in $\Pi$, $h^{+}(s)=\infty \iff h^{max}(s)=\infty \iff h^{add}(s)=\infty$
>
>>[!done] Proof
>>$h^{max}$ and $h^{add}$ agree on states with infinite heuristic value simply because their only difference lies in the use of the max vs. $\\sum$ operations which does not affect this property.
>>
>>- $h^{max}=\infty \implies h^{+}+\infty$ because $h^{max}\leq h^{+}$
>>- $h^{max}(s)<\infty \implies h^{+}(s)<\infty$ because $h^{max}$ can then be used to generate a closed well-founded best-supporter function, from which a relaxed plan can be extracted (cf. [[#Relaxed plan heuristic|relaxed plan heuristic]])
>
>>[!tldr]
>>States for which no relaxed plan exists are easy to recognize, and that is done both by $h^{max}$ and $h^{add}$. Approximation is needed only for the cost of an optimal relaxed plan, if it exists

>[!info] **Proposition** ($h^{max}=h^{1}$)
>>[!done] Proof
>>Say $g=\{g'\}$. By definition, $regr(\{g'\},a)\neq \bot$ if $add_{a} \cap \{g'\}\neq \varnothing$ and $del_a\cap \{g'\}=\varnothing$; then, $regr(g,a)=(\{g'\}\backslash add_{a})\cup pre_{a}$. With $add_{a}\cap del_{a}=\varnothing$ and $\{g'\}\subseteq add_{a}$, this is the same as “$g'\in add_{a}$, and $regr(g,a)=pre_{a}$”
>
>Given this the dynamic programming algorithm to compute $h^{add}$ is substantially the same as the one used for $h^{max}$ (so for $h^{1}$).
>
>>[!important] Pseudocode
>>$$
>>\begin{array}{l}
>>\textbf{new } \text{table } T^{\text{add}}_0(g), \text{ for } g \in P \\
>>\textbf{for all } g \in P: \quad T^{\text{add}}_0(g) := \begin{cases} 0 & g \in s \\ \infty & \text{otherwise} \end{cases} \\
>>\\
>>\textbf{fn } {\color{yellow}\mathit{Cost}_i}(g) := \begin{cases} T^{\text{add}}_i(g) & |g| = 1 \\ {\color{red}\sum_{g' \in g}} T^{\text{add}}_i(g') & |g| > 1 \end{cases} \\
>>\\
>>\textbf{fn } {\color{yellow}\mathit{Next}_i}(g) := \min\left[\mathit{Cost}_i(g),\ \min_{a \in A,\, g \in \mathit{add}_a} \left[ c(a) + \mathit{Cost}_i(\mathit{pre}_a) \right]\right] \\
>>\\
>>i := 0 \\
>>\textbf{do forever}: \\
>>\quad \textbf{new } \text{table } T^{\text{add}}_{i+1}(g), \text{ for } g \in P \\
>>\quad \textbf{for all } g \in P: \quad T^{\text{add}}_{i+1}(g) := \mathit{Next}_i(g) \\
>>\quad \textbf{if } T^{\text{add}}_{i+1} = T^{\text{add}}_i \textbf{ then stop endif} \\
>>\quad i := i + 1 \\
>>\textbf{enddo}
>>\end{array}
>>$$

>[!info] **Proposition**
>Let $\Pi=(P,A,c,I,G)$ be a STRPIS planning task. Then the series $\{Cost_{i}(g)\}_{i=0,\dots}$ converges to $h^{add}(s,g)$, for all $g\subseteq P$

>[!example] $h^{add}$ in “Logistics”
>![[logistics.png|300]]
>
>- $P = \{t\_A, t\_B, t\_C, t\_D, p\_T, p\_A, p\_B, p\_C, p\_D\}$
>- $I = \{tA, pC\}$; $G = \{tA, pD\}$.
>- $A$ (unit-cost):
>	- $dr_{xy}$ ($x,y$ adjactent): pre $t_x$; add $t_y$; del $t_x$
>	- $lo_x$: pre $t_x, p_x$; add $pT$; del $p_x$
>	- $ul_x$: pre $t_x, pT$; add $p_x$; del $pT$
>
>| $i$ | tA | tB | tC | tD | pT | pA | pB | pC | pD |
>|---|---|---|---|---|---|---|---|---|---|
>| 0 | 0 | ∞ | ∞ | ∞ | ∞ | ∞ | ∞ | 0 | ∞ |
>| 1 | 0 | 1<sub>drAB</sub> | ∞ | ∞ | ∞ | ∞ | ∞ | 0 | ∞ |
>| 2 | 0 | 1 | 2<sub>drBC</sub> | ∞ | ∞ | ∞ | ∞ | 0 | ∞ |
>| 3 | 0 | 1 | 2 | 3<sub>drCD</sub> | 3<sub>loC</sub> | ∞ | ∞ | 0 | ∞ |
>| 4 | 0 | 1 | 2 | 3 | 3 | 4<sub>ulA</sub> | $\color{red}5$<sub>ulB</sub> | 0 | $\color{red}7$<sub>ulD</sub> |
>| 5 | 0 | 1 | 2 | 3 | 3 | 4 | $\color{red}5$ | 0 | $\color{red}7$ |
>
>(in $\color{red}\text{red}$ the values different from $T^1_i$, i.e., from $h^{max}$)
>
>So $h^{add}(I) = tA + pD = 0 + 7 = 7$,  while $h^*(I) = 8 > 7 = h^{add}(I) > h^+(I) = 5$.
>
>>[!note]- Azioni per riga
>>- $i=1$: $tB = 1 + tA(0)$ via $drAB$
>>- $i=2$: $tC = 1 + tB(1)$ via $drBC$
>>- $i=3$:
>>	- $tD = 1 + tC(2)$ via $drCD$
>>	- $pT = 1 + [tC(2) + pC(0)] = 3$ via $loC$
>>- $i=4$ (tutte con $ul_x$, pre $\{t_x, pT\}$ → **somma**):
>>	- $pA = 1 + [tA(0) + pT(3)] = 4$ via $ulA$
>>	- $pB = 1 + [tB(1) + pT(3)] = 5$ via $ulB$ (con $h^{max}$: $1 + \max(1,3) = 4$)
>>	- $pD = 1 + [tD(3) + pT(3)] = 7$ via $ulD$ (con $h^{max}$: $1 + \max(3,3) = 4$)
>>	- $pC$: $ulC$ darebbe $1 + [2 + 3] = 6$, ma resta $0$ (min con valore attuale)
>>- $i=5$: nessun cambiamento → punto fisso, stop.
>
>>[!question]- Why is $h^{add}(I) > h^+(I)$? (over-counting)
>>- $h^+(I) = 5$: optimal relaxed plan $drAB, drBC, loC, drCD, ulD$ ($tA$ is never deleted, so there is no need to drive back).
>>- $h^*(I) = 8$: the same plan plus the way back $drDC, drCB, drBA$.
>>- $pD = 7$ sums the supports of the two preconditions of $ulD$ **independently**:
>>	- $tD(3)$: $drAB, drBC, drCD$
>>	- $pT(3)$: $drAB, drBC, loC$
>>	- → $drAB$ and $drBC$ are counted **twice**.
>>- With 100 packages at $C$ and all goals at $D$: $h^{add}(I) = 100 \cdot 7 = 700$, but $h^*(I) = 206$ ($2$ drives to $C$ + $100$ loads + $1$ drive + $100$ unloads + $3$ drives back). For each package, $h^{add}$ counts $5$ drive actions ($2 \times drAB + 2 \times drBC + 1 \times drCD$).
>>
>>>[!tldr]
>>>$tD$ and $pT$ already share the previous steps but by using $h^{add}$ i count it two times (even if it’s not necessary)

### Summary
Summary of typical issues in practice with $h^{add}$ and $h^{max}$:
- both $h^{add}$ and $h^{max}$ can be computed reasonably quickly
- $h^{max}$ is admissible, but is typically far too optimistic
- $h^{add}$ is not admissible, but is typically a lot more informed than $h^{max}$
- $h^{add}$ overcounts by ignoring positive interactions, i.e., sub-plans shared between subgoals (such overcounting can result in dramatic over-estimates of $h^{*}$)

The relaxed plan heuristic keeps $h^{add}$’s informativity but avoids over-counting.

---
## Relaxed plan heuristic
The idea is to first compute a **best-supporter function $bs$**, which for every fact $p \in P$ returns an action that is deemed to be the cheapest achiever of $p$ (within the relaxation). Then *extract a relaxed plan* from that function, by applying $bs$ to singleton subgoals and collecting all the actions.

And now for the details:
- to be concrete: the best-supporter functions we will use
- how to extract a relaxed plan given best-supporter function
- what is a best-supporter function, in general? what properties does it need to have for relaxed plan extraction to work as intended?
- wrapping up: correctness of relaxed plan extraction, correctness of our best-supporter functions, definition of relaxed plan heuristic, discussion of its properties

### The best-supporter functions we will use

>[!info] **Definition** (best supporters for $h^{max}$ and $h^{add}$)
>Let $\Pi=(P,A,c,I,G)$ be a STRIPS planning task, and let $s$ be a state ($bs_{s}$ is the action that minimized the expression for $max$ and for $add$).
>
>The *$h^{max}$ supporter function* $bs_{s}^{max}: \{p \in P \mid 0<h^{max}(s,\{p\})<\infty\} \to A$ is defined by **$bs^{max}_{s}(p)=\text{arg min}_{a\in A,p \in add_{a}}c(a)+h^{max}(s,pre_{a})$**
>
>The *$h^{add}$ supporter function* $bs_{s}^{add}: \{p \in P \mid 0<h^{add}(s,\{p\})<\infty\} \to A$ is defined by **$bs^{add}_{s}(p)=\text{arg min}_{a\in A,p \in add_{a}}c(a)+h^{add}(s,pre_{a})$**
>
>In both definitions $0$ and $\infty$ are avoided for two different reasons. $h(s,\{p\})=0$ means that $p \in s$ (if a fact is true does not need a supporter), while if $h(s,\{p\})=\infty$ means that $p$
 is not reachable in the relaxed task ()
 
>[!example] $h^{add}$ in "Logistics": best-supporter function
>**Heuristic values:**
>
>| | tA | tB | tC | tD | pT | pA | pB | pC | pD |
>|---|---|---|---|---|---|---|---|---|---|
>| $h^{add}$ | 0 | 1 | 2 | 3 | 3 | 4 | 5 | 0 | 7 |
>
>**Yield best-supporter function:**
>
>| | tA | tB | tC | tD | pT | pA | pB | pC | pD |
>|---|---|---|---|---|---|---|---|---|---|
>| $bs^{add}$ | – | $drAB$ | $drBC$ | $drCD$ | $loC$ | $ulA$ | $ulB$ | – | $ulD$ |
>
>>[!note]- How $bs^{add}$ is read off
>>- For each fact $p \notin s$: $bs(p) := \arg\min_{a \in A,\, p \in add_a} \left[ c(a) + h^{add}(s, pre_a) \right]$, i.e. the action that gave $p$ its final value in the table.
>>- For $p \in s$ ($tA$, $pC$): no supporter needed (–), since $h^{add}(s,p) = 0$.
>>- e.g. $bs(pD) = ulD$ because $1 + [tD(3) + pT(3)] = 7 = h^{add}(pD)$.


#### Correctness

>[!info] **Proposition**
>Let $\Pi=(P,A,c,I,G)$ be a STRIPS planning task such that, $\text{for all } a \in A,\ c(a) > 0$. Let $s$ be a state where $h^{+}(s) < \infty$. Then both $bs^{max}_{s}$ and  $bs^{add}_{s}$ are closed well-founded best-supporter functions for $s$.
>
>>[!done] Proof
>>**Closed.** $h^{+}(s) < \infty$ implies $h^{max}(s) < \infty$ ($h^{+}$, $h^{max}$ and $h^{add}$ agree on $\infty$). Hence $bs^{max}_{s}$ is closed: $h^{max}(s,G) < \infty$, so every goal $g \in G \setminus s$ has $0 < h^{max}(s,\{g\}) < \infty$ and $bs^{max}_s(g)$ is defined. Recursively, $h^{max}(s,pre_{a}) < \infty$ for the best supporters $a$, so each of their preconditions not in $s$ is again in the domain of $bs^{max}_s$, and so on.
>>
>>**Well-founded.** If $a = bs^{max}_{s}(p)$, then $a$ is the action yielding
>>$$0 < c(a) + h^{max}(s, pre_{a}) = h^{max}(s,\{p\}) < \infty$$
>>in the $h^{max}$ equation. ${\color{red}\text{Since } c(a) > 0}$, we have ${\color{red}h^{max}(s,pre_{a}) < h^{max}(s,\{p\})}$ and thus, for all $q \in pre_{a}$, ${\color{red}h^{max}(s,\{q\}) < h^{max}(s,\{p\})}$ (because $h^{max}(s, pre_a) = \max_{q \in pre_a} h^{max}(s,\{q\})$).
>>
>>So every step "fact → action → fact" in the support graph ($q \to a$ because $q \in pre_a$, $a \to p$ because $a = bs(p)$) **strictly increases** $h^{max}$:
>>$$q \longrightarrow a \longrightarrow p \quad\Rightarrow\quad h^{max}(s,\{q\}) < h^{max}(s,\{p\})$$
>>Transitively, if the support graph contains a path from fact vertex $r$ to fact vertex $t$, then $h^{max}(s,\{r\}) < h^{max}(s,\{t\})$. A cycle through $r$ would give $h^{max}(s,\{r\}) < h^{max}(s,\{r\})$, which is impossible → there can't be cycles in the support graph, and $bs^{max}_{s}$ is well-founded.
>>
>>**Similar for $bs^{add}_{s}$**: $h^{add}(s,pre_a) = \sum_{q \in pre_a} h^{add}(s,\{q\}) \geq h^{add}(s,\{q\})$ for every $q \in pre_a$ (all terms are $\geq 0$), so again $c(a) > 0$ gives $h^{add}(s,\{q\}) < h^{add}(s,\{p\})$.
>>
>>>[!note] 0-cost actions
>>>One can also use $h^{max}$ and $h^{add}$ for 0-cost actions, by appropriate tie-breaking in cases where $h^{max}(s,\{p\}) = h^{max}(s,pre_{a})$. Details omitted.
>>
>>>[!tldr]
>>>Lungo ogni cammino del support graph il valore $h$ dei fatti **cresce strettamente** (grazie a $c(a) > 0$). Un ciclo dovrebbe tornare al valore di partenza, cosa impossibile. Quindi le $bs$ ricavate da $h^{max}$/$h^{add}$ sono **sempre** well-founded, e closed perché $h^+(s) < \infty$.

### Relaxed plan extraction

>[!important] Pseudocode
>$$
>\begin{array}{l}
>\mathit{Open} := G \setminus s; \quad \mathit{Closed} := \emptyset; \quad \mathit{RPlan} := \emptyset \\
>\textbf{while } \mathit{Open} \neq \emptyset \textbf{ do}: \\
>\quad \textbf{select } g \in \mathit{Open} \\
>\quad \mathit{Open} := \mathit{Open} \setminus \{g\}; \quad \mathit{Closed} := \mathit{Closed} \cup \{g\} \\
>\quad \mathit{RPlan} := \mathit{RPlan} \cup \{{\color{red}bs(g)}\}; \quad \mathit{Open} := \mathit{Open} \cup \left(\mathit{pre}_{bs(g)} \setminus (s \cup \mathit{Open} \cup \mathit{Closed})\right) \\
>\textbf{endwhile} \\
>\textbf{return } \mathit{RPlan}
>\end{array}
>$$
>
>→ Starting with the top-level goals, iteratively close open singleton subgoals by selecting the best supporter. We start from $G\backslash s$ because $s$ is already true, and then we only add the facts that are in $pre_{a}$ and that are not yet true.

>[!question] Fast, but is it correct?
>**Fast:** the number of iterations is bounded by $|P|$, and each one takes near-constant time.
>
>- What if $g \notin add_{bs(g)}$? → doesn't make sense → **Condition (A)**
>- What if $bs(g)$ is undefined? → segmentation fault → **Condition (B)**
>- What if the support for $g$ eventually requires $g$ itself as a precondition (by then already in $\mathit{Closed}$ and the algorithm would not add it again to the list of the $Open$, i.e., $bs(g)=a_{1}$ with $g'\in pre_{a_{1}}$, and $bs(g')=a_{2}$ with $g\in pre_{a_{2}}$)? → this does not yield a relaxed plan → **Condition (C)**

>[!example]
>![[logistics.png|300]]
>- $I = \{tA, pC\}$; $G = \{tA, pD\}$; $A$: $dr_{xy}, lo_x, ul_x$.
>
>| | tA | tB | tC | tD | pT | pA | pB | pC | pD |
>|---|---|---|---|---|---|---|---|---|---|
>| $bs^{add}$ | – | $drAB$ | $drBC$ | $drCD$ | $loC$ | $ulA$ | $ulB$ | – | $ulD$ |
>
>**Extracting a relaxed plan** (start: $\mathit{Open} = G \setminus s = \{pD\}$):
>1. $bs^{add}_s(pD) = ulD$; opens $\color{red}tD, pT$
>2. $bs^{add}_s(tD) = drCD$; opens $\color{red}tC$
>3. $bs^{add}_s(pT) = loC$; opens nothing new ($pC \in s$, $tC \in \mathit{Open}$)
>4. $bs^{add}_s(tC) = drBC$; opens $\color{red}tB$
>5. $bs^{add}_s(tB) = drAB$; opens nothing ($tA \in s$)
>
>>[!note]- Trace of Open / Closed / RPlan
>>| step | $g$ | $\mathit{Open}$ (after) | $\mathit{Closed}$ (after) | added to $\mathit{RPlan}$ |
>>|---|---|---|---|---|
>>| 0 | – | $\{pD\}$ | $\emptyset$ | – |
>>| 1 | $pD$ | $\{tD, pT\}$ | $\{pD\}$ | $ulD$ |
>>| 2 | $tD$ | $\{pT, tC\}$ | $\{pD, tD\}$ | $drCD$ |
>>| 3 | $pT$ | $\{tC\}$ | $\{pD, tD, pT\}$ | $loC$ |
>>| 4 | $tC$ | $\{tB\}$ | $\{pD, tD, pT, tC\}$ | $drBC$ |
>>| 5 | $tB$ | $\emptyset$ | $\{pD, tD, pT, tC, tB\}$ | $drAB$ |
>
>>[!success]- Result
>>$\mathit{RPlan} = \{drAB, drBC, loC, drCD, ulD\}$, so $h^{FF}(I) = 5 = h^+(I)$, whereas $h^{add}(I) = 7$.
>>$drAB$ and $drBC$ are counted **once**: $tC$ is already in $\mathit{Open}$ when $loC$ is selected, so its support is not added a second time. This removes the over-counting of $h^{add}$.

#### Correctness

>[!info] **Proposition**
>Let $\Pi=(P,A,c,I,G)$ be a STRIPS planning task, let $s$ be a state, and let $bs$ be a closed well-founded best-supporter function for $s$. Then the actions set $RPlan$ returned by the relaxed plan extraction can be sequenced into a relaxed plan $\vec{a}^{+}$ for $s$.
>
>>[!done] Proof
>>Order $a$ before $a'$ whenever the support graph contains a path from $a$ to $a'$. Since the support graph is acyclic, such a sequencing $\vec{a}:=\langle a_{0},\dots,a_{n-1} \rangle$ exists (topological ordering).
>>
>>Assume the opposite, i.e., $p \in pre_{a_{0}} \setminus s$ (i.e., $a_{0}$ needs a fact $p$ that is not in $s$. Then $p$ ends up in $Open$ (it is inserted when processing the fact supported by $a_0$, or it is already in $Open$, or already in $Closed$), and every fact in $Open$ is eventually selected, so $RPlan$ contains $bs(p)$ (defined since $bs$ is closed). The support graph then contains the edges $bs(p) \to p$ (because $bs(p)$ supports $p$) and $p \to a_{0}$ (because $p \in pre_{a_0} \setminus s$):
>>$$bs(p)\longrightarrow p \longrightarrow a_{0}$$
>>Since $bs(p) \in RPlan$, we have $bs(p) = a_k$ for some $k \geq 0$:
>>- $k \geq 1$ → the path $a_k \to p \to a_0$ forces $a_k$ to be ordered **before** $a_0$, but $a_k$ comes **after** $a_0$ in the sequence → contradiction with the topological ordering
>>- $k = 0$ → the support graph contains the cycle $a_{0}\to p\to a_{0}$ → contradiction with $bs$ being well-founded (acyclic support graph)
>>
>>Hence $pre_{a_0} \subseteq s$.
>>
>>The same argument shows that $p \in s\cup add_{a_{1}}$ for all $p \in pre_{a_{2}}$: else $bs(p)\neq a_{1}$ would necessarily be ordered before $a_{2}$.
>>
>>Iterate the argument for $pre_{a_{n}}$, $2\leq i\leq n$, with $pre_{a_{n}}:=G$ and using $s\cup add_{a_{1}}\cup\dots \cup add_{i-1}$ in each step.
>>
>>>[!tldr]
>>>**la prima azione dell’ordine topologico è una sorgente del grafo**, e le sorgenti sono esattamente le azioni con tutte le precondizioni in s.

### Best-supporter functions
For relaxed plan extraction to make sense, it requires a *closed well-founded* best supporter function

>[!info] **Definition** (best-supporter function)
>Let $\Pi=(P,A,c,I,G)$ be a STRIPS planning task, and let $s$ be a state. A best-supporter function for $s$ is a partial function $bs:(P\backslash s)\to A$ such that $p \in add_{a}$ whenever $a=bs(p)$.
>
>The **support graph** of $bs$ is the directed graph with vertices $(P\backslash s)\cup A$ and arcs $\{(a,p)\mid a=bs(p)\}\cup \{(p,a)\mid p \in pre_{a}\backslash s\}$.
>
>We say that $bs$ is **closed** if $bs(p)$ is defined for every $p \in (P\backslash s)$ that has a path to a goal $g\in G$ in the support graph.
>
>We say that $bs$ is **well-founded** if the support graph is acyclic.
>
>- “$p \in add_{a}$ whenever $a=bs(p)$” → **Condition (A)**
>- $bs$ is closed → **Condition (B)** (“$bs$ will be defined wherever it takes us to“)
>- $bs$ is well-founded → **Condition (C)**; relaxed plan extraction starts at the goals, and chains backwards in the support graph. If there are cycles, then this backchaining may not reach the currently true state $s$, and thus not yield a relaxed plan

>[!example] Support graphs and Condition (C) in "Logistics"
>- Map: $A - B - C - D$ (line).
>- $I = \{tA\}$; $G = \{tD\}$; $A$: $drXY$ for adjacent $X, Y$ (pre $tX$; add $tY$; del $tX$).
>- **Support graph** of $bs$: vertices $(P \setminus s) \cup A$, arcs
>	- $a \to p$ if $a = bs(p)$ (the supporter produces $p$)
>	- $p \to a$ if $p \in pre_a \setminus s$ ($p$ is needed by $a$)
>
>**How to do it (well-founded)**
>
>| $p$ | $bs(p)$ |
>|---|---|
>| $tB$ | $drAB$ |
>| $tC$ | $drBC$ |
>| $tD$ | $drCD$ |
>
>```mermaid
>graph LR
>  drAB --> tB --> drBC --> tC --> drCD --> tD
>  tB --> drBA
>  tC --> drCB
>  tD --> drDC
>```
>
>- No cycles. Backchaining from $tD$: $tD \leftarrow drCD \leftarrow tC \leftarrow drBC \leftarrow tB \leftarrow drAB$, and $drAB$ needs only $tA \in s$ (no incoming arc), so the chain **reaches $s$**.
>- $\mathit{RPlan} = \{drCD, drBC, drAB\}$, executable in the order $drAB \to drBC \to drCD$ ✓
>
>**How NOT to do it (not well-founded)**
>
>| $p$ | $bs(p)$ |
>|---|---|
>| $tB$ | $\color{red}drCB$ |
>| $tC$ | $drBC$ |
>| $tD$ | $drCD$ |
>
>```mermaid
>graph LR
>  drAB
>  drCB --> tB --> drBC --> tC --> drCD --> tD
>  tC --> drCB
>  tB --> drBA
>  tD --> drDC
>  classDef cyc stroke:#e24b4a,color:#e24b4a
>  class tB,drBC,tC,drCB cyc
>```
>
>- **Cycle**: $tB \to drBC \to tC \to drCB \to tB$ ($tB$ is supported by $drCB$, which needs $tC$, which is supported by $drBC$, which needs $tB$).
>- $drAB$ is now isolated: it is nobody's supporter, so backchaining never gets back to $tA \in s$.
>
>>[!note]- Trace of relaxed plan extraction (not well-founded case)
>>| step | $g$ | $bs(g)$ | $\mathit{Closed}$ (after) | added to $\mathit{Open}$ |
>>|---|---|---|---|---|
>>| 0 | – | – | $\emptyset$ | $\{tD\}$ |
>>| 1 | $tD$ | $drCD$ | $\{tD\}$ | $\{tC\}$ |
>>| 2 | $tC$ | $drBC$ | $\{tD, tC\}$ | $\{tB\}$ |
>>| 3 | $tB$ | $drCB$ | $\{tD, tC, tB\}$ | $\{tC\} \setminus \mathit{Closed} = \emptyset$ |
>>
>>- Returns $\{drCD, drBC, drCB\}$: the algorithm **terminates** (thanks to $\mathit{Closed}$), but the result is **not** a relaxed plan. Starting from $\{tA\}$ no action is applicable: $drCB$ needs $tC$, $drBC$ needs $tB$, $drCD$ needs $tC$.
>>- $bs$ is **closed** (defined on $tB, tC, tD$), and Condition (A) holds, but it is **not well-founded** → Condition (C) is violated.
>>- This $bs$ can never come from $h^{max}$/$h^{add}$: there $h(tB) = 1$ via $drAB$, while $drCB$ would give $1 + h(tC) = 3$, so the argmin always picks $drAB$.

### Relaxed plan heuristic
![[Pasted image 20261119122854.png]]

>[!info] **Definition** (relaxed plan heuristic)
>A heuristic function is called **relaxed plan heuristic**, denoted $h^{FF}$, if, given a state $s$, it returns $\infty$ if not relaxed plan exists, and otherwise returns $\sum_{a\in RPlan} c(a)$ where $RPlan$ is the action set returned by relaxed plan extraction on a closed well-founded best-supporter function for $s$.

Relaxed plan heuristic $h^{FF}$: algorithmically
1. Run $h^{max}$ or $h^{add}$ on $s$. If no relaxed plan exists, then $h^{max} = h^{add} = \infty$ ($h^+$, $h^{max}$, $h^{add}$ agree on $\infty$) → **return $\infty$**.
2. Otherwise, $h^{max}$/$h^{add}$ yields a **closed well-founded** best-supporter function $bs$.
3. Relaxed plan extraction on $bs$ yields a relaxed plan $\mathit{RPlan}$
4. **Return $\sum_{a \in \mathit{RPlan}} c(a)$**.

>[!note] Name
>The name "$h^{FF}$" is historically motivated: FF is the name of the first planning system using this heuristic [Hoffmann and Nebel (2001)].

#### Properties

>[!info] **Proposition** ($h^{FF}$ is pessimistic and agrees with $h^{+}$ on $\infty$)
>For all STRIPS planning tasks $\Pi$, $h^{FF} \geq h^{+}$; for all states $s$, $h^{+}(s) = \infty$ if and only if $h^{FF}(s) = \infty$. There exist $\Pi$ and $s$ so that $h^{FF}(s) > h^{*}(s)$.
>
>>[!done] Proof
>>- **$h^{FF} \geq h^{+}$**: follows from the [[#Relaxed plan heuristic|previous paragraph]]. $\mathit{RPlan}$ can be sequenced into a relaxed plan for $s$, and $h^{+}(s)$ is the cost of an **optimal** relaxed plan, so $h^{FF}(s) = \sum_{a \in \mathit{RPlan}} c(a) \geq h^{+}(s)$.
>>- **Agrees with $h^{+}$ on $\infty$**: direct from the definition. $h^{FF}$ returns $\infty$ exactly when no relaxed plan exists, i.e., when $h^{+}(s) = \infty$.
>>- **Inadmissibility**: when $bs$ makes sub-optimal choices. Since $bs$ picks supporters **one fact at a time**, it can miss an action that achieves several goals at once.
>>
>>>[!example]- Inadmissibility example
>>>$s = \emptyset$, $G = \{g_1, g_2\}$, no deletes:
>>>- $a$: add $\{g_1, g_2\}$, cost $3$
>>>- $b_1$: add $\{g_1\}$, cost $2$
>>>- $b_2$: add $\{g_2\}$, cost $2$
>>>
>>>Per fact: $h(g_1) = \min(3, 2) = 2$ via $b_1$, $h(g_2) = 2$ via $b_2$ → $bs(g_1) = b_1$, $bs(g_2) = b_2$.
>>>$\mathit{RPlan} = \{b_1, b_2\}$ → $h^{FF}(s) = 4 > 3 = h^{*}(s) = h^{+}(s)$ (just apply $a$).
>
>→ Relaxed plan heuristics have the same theoretical properties as $h^{add}$.

>[!question] So what's the point?
>- Example "Logistics" with $100$ packages at $C$ that need to go to $D$: $h^{FF}(I) = 203 \ll 700 = h^{add}(I)$, because relaxed plan extraction counts each drive action only once.
>	- $203 = 3$ drives ($drAB, drBC, drCD$) $+\ 100 \times lo_C + 100 \times ul_D$; here $h^{FF}(I) = h^{+}(I)$, while $h^{*}(I) = 206$ ($+3$ drives back to $A$).
>- $h^{FF} \text{ never overcounts}$ (each action is counted at most once), as relaxed plan extraction returns a **set** of actions.
>- In practice, $h^{FF}$ typically is close to $h^{+}$ and does not over-estimate $h^{*}$.
>	- → $h^{FF}$ can be inadmissible, just like $h^{add}$, but for more subtle reasons.
>
>>[!tldr]
>>Sulla carta $h^{FF}$ ha le stesse proprietà di $h^{add}$: $\geq h^+$, esatto su $\infty$, non ammissibile. In pratica è molto migliore, perché non conta due volte la stessa azione. Sbaglia solo quando $bs$, scegliendo un fatto alla volta, non vede un'azione che soddisfa più goal insieme.

>[!info]- Helpful Actions Pruning (for Reference)
>**Idea**: in search, expand only those actions contained in the relaxed plan.
>- e.g. relaxed plan = "Go to supermarket, Buy milk, …" → from the current state, consider only the actions that appear in it.
>- e.g. *Schedule* domain (many tools, many objects): the relaxed plan doesn't drill holes into objects that need to be painted, so those actions are never expanded.
>
>>[!info] **Definition** (Helpful Actions)
>>Let $h^{FF}$ be a relaxed plan heuristic, let $s$ be a state, and let $\mathit{RPlan}$ be the action set returned by relaxed plan extraction on the closed well-founded best-supporter function for $s$ which underlies $h^{FF}$. Then an action $a$ **applicable to $s$** is called **helpful** if it is contained in $\mathit{RPlan}$.
>
>**Remarks**:
>- Initially introduced in FF [Hoffmann and Nebel (2001)], restricting search to use only the helpful actions.
>- There is **no guarantee** that the actually needed actions will be helpful → this does **not preserve completeness** (cf. slide 61: in the Tiger task, $jumpTiger$ is the only helpful action in $\{alive, haveTiger\}$, but it kills you).
>- Fast Downward uses the term **preferred operators** for similar concepts, for a broad variety of heuristic functions $h$, and offers several ways of using them.
>- Preferred operators may have **more impact on performance** than different heuristic functions [Richter and Helmert (2009)].
>
>>[!tldr]
>>$h^{FF}$ restituisce, oltre al valore, anche un **insieme di azioni**. Le azioni applicabili subito che compaiono in $\mathit{RPlan}$ sono quelle "utili", e si possono espandere solo quelle. In pratica aiuta tantissimo, ma si perde la completezza: l'azione davvero necessaria potrebbe non comparire nel piano rilassato.

---
## Delete relaxation for FDR
In FDR the variables change their values, so in order to apply the delete relaxation, we need to make them accumulate. In practice (in particular in the Fast Downward implementation), simply formulate the algorithms relative to the “FDR facts” $v=d$.

>[!info] **Definition** (delete relaxed FDR)
>Let $\Pi=(V,A,c,I,G)$ be and FDR planning task. Denote by $P_{V}:=\{v=d\mid v\in V, d\in D_{v}\}$ the set of (FDR) facts.
>
>The **relaxed state space** of $\Pi$ is the tuple $\Theta^{+}_{\Pi}=(S^{+},A,c,T,I,S^{+G})$ where:
>- the states (also relaxed states) $S^{+}=2^{P_{V}}$ are the subsets $s^{+}$ of $P_{V}$
>- the action $A$ and cost function $c$ are those of $\Pi$
>- the transitions are $T=\{s^{+}\overset{a}{\to}s'^{+}\mid pre_{a}\subseteq s^{+}, s'^{+}=s^{+}\cup eff_{a}\}$
>- the initial state $I$ is identical to that of $\Pi$
>- the goal states are $S^{+G}=\{s^{+}\in S^{+}\mid G\subseteq s^{+}\}$
>
>An (optimal) relaxed plan for $s^{+}\in S^{+}$ is an (optimal) solution  for $s^{+}$ in $\Theta t^{+}_{\Pi}$. A relaxed plan for $I$ is also called a relaxed plan for $\Pi$.
>
>Let $\Theta_{\Pi}=(S,A,c,T,I,G)$ be a state of $\Pi$. The *optimal delete relaxation heuristic $h^{+}$* for $\Pi$ is the function $h^{+}:S\to \mathbb{R}_{0}^{+}\cup \{\infty\}$ where $h^{+}(s)$ is defined as the cost of an optimal relaxed plan for $s$
>
>>[!tldr]
>>FDR relaxed states may contain several values (facts) for a single variable, and transitions accumulate the values.

