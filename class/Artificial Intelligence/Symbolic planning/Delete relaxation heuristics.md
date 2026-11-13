---
Class: "[[Artificial Intelligence]]"
Sem: WiSe 26/27
---
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
>>**Membership.** In a relaxed plan every action is needed at most once (applying it again adds nothing new), so an optimal relaxed plan has length $\le |A|$. Guess an action sequence of length $\le |A|$ and check in polynomial time that it is a relaxed plan with cost $\le B$.
>>
>>**Hardness.** By reduction from SAT.
>>
>>>[!example] Example: $X_1 \wedge \neg X_1$
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