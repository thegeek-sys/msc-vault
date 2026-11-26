---
Class: "[[Artificial Intelligence]]"
Sem: WiSe 26/27
---
---
## Introduction
Pattern databases are a method for designing abstraction functions $\alpha$ and for computing the associated heuristic functions.

>[!example] Pattern databases in a nutshell
>![[Pasted image 20261126180926.png|450]]
>
>”Abstract the planning task by choosing a subset $P$ of variables (the pattern), and ignoring the values of all other variables.

---
## Projections and PDB heuristics

>[!info] **Definition** (projection, PDB heuristic)
>Let $\Pi=(V,A,c,I,G)$ be an FDR planning task with state space $\Theta_{\prod}=(S,L,c,T,I,S^{G})$, and let $P\subseteq V$. For a partial assignment $\varphi$ to $P$. Let $S^{P}$ be the set of variable assignments to $P$.
>
>The **projection $\pi_{P}$**$:S\to S^{P}$ is defined by *$\pi_{P}(s):=s\mid_{P}$*
>
>>[!tldr]
>>$\pi_{P}$ maps two states $s_{1}$ and $s_{2}$ to the same abstract state iff they agree on all variables in $P$
>
>$P$ is the **pattern** of $\pi_{P}$. The abstraction heuristic induced by $\pi_{P}$ on $\Theta_{\Pi}$ is a **pattern database heuristic**, short *PDB heuristic*. We write $h^{P}$ as a short-hand for $h^{\pi_{P}}$, and $\Theta^{P}_{\prod}$ or $\Theta^{P}$ as short-hands for $\Theta^{\pi_{P}}_{\prod}$
>
>$h^{P}$ is stored in a lookup table called a **pattern database** (PDB).

>[!example] “Logistics mal anders”: state space
>![[logistics_mal_anders.png|300]]
>- state variable $package: \{L,R,A,B\}$
>- state variable $truck\;A: \{L,R\}$
>- state variable $truck\;B: \{L,R\}$
>
>![[Pasted image 20261126182040.png|400]]
>
>Abstraction induced by $\pi_{\{package\}}$:
>![[Pasted image 20261126182118.png|400]]
>$$h^{\{package\}}(LRR)=2$$
>
>Abstraction induced by $\pi_{\{package,\;truck\;A\}}$:
>![[Pasted image 20261126182217.png|400]]
>$$h^{\{package,\;truck\;A\}}(LRR)=2$$

### Larger patterns = refinements

>[!info] **Proposition** (larger patterns are refinements)
>Let $\Pi$ be an FDR planning task, and let $P'\subseteq P\subseteq V$. Then $\pi_{P}$ is a refinement of $\pi_{P'}$
>
>>[!done] Proof
>>$\pi_{P'}$ can be viewed as an abstraction of $\Theta^{P}$, i.e., in the above set $\Theta^{\alpha}:=\Theta^{P}$, and set $\alpha'$ bot be the projection of $P$ onto $P'$

>[!info] **Corollary** (larger patterns yield better heuristics)
>Let $\Pi$ be an FDR planning task, and let $P'\subseteq P\subseteq V$. Then $h^{P}$ dominates $h^{P'}$, i.e., $h^{P'}\leq h^{P}$.
>
>>[!done] Proof
>>From the previous proposition ($\pi_{P}$ is a refinement of $\pi_{P'}$) and the theorem *refinements improve the heuristic* ([[Abstraction heuristics]]).
>
>- And, of course, $h^{V}=h^{*}$
>- Pattern size controls the trade-off between accuracy and computational cost
>
>>[!tip]
>>$h^{V}=h^{*}$ because projecting onto **all** variables is the identity abstraction: $\Theta^{\pi_{V}}=\Theta$ (see *Identity abstraction* in [[Abstraction heuristics]]).

### Implementation
Given a pattern $P$, how do you compute $h^{P}$? More precisely: how do you compute a data structure that efficiently represents the function $h^{P}(s)$, for all states $s$?

Here’s how:
1. in a precomputation step, we compute an explicit graph representation for the abstract state space $\Theta^{\pi_{P}}_{\Pi}$, and compute the abstract remaining cost for every abstract state
2. we store the abstract remaining cost in a table; during a search, we use lookup on this table to implement $h^{P}$

#### (I) Precomputation step
Let $\Pi$ be a planning task and $P$ a pattern. Let $\Theta=\Theta_{\Pi}$ and $\Theta'=\Theta^{\pi_{P}}_{\Pi}$. We want to compute a graph representation of $\Theta'$.

>[!question] What’s the issue?
>*$\Theta'$ is defined through a function on $\Theta$*: each concrete transition induces an abstract transition, each concrete goal state induces an abstract goal state.
>
>*In principle, we can compute $\Theta'$ by iterating over all transitions/goal states of $\Theta$. BUT*: this would take time $\Omega(| | \Theta| |)$, which comes down to solving the original (concrete, not abstract) planning task in the first place, using blind search.
>
>>[!done]
>>We need a way of computing $\Theta'$ in time polynomial in $| | \Pi | |$ and $| |\Theta' | |$
>
>>[!info] Capiamo
>>![[bangdesk.gif]]
>>
>>Per definizione $\Theta'$ si ottiene da $\Theta$: ogni transizione astratta nasce da una transizione concreta. Costruirlo "alla lettera" vorrebbe dire **enumerare tutto $\Theta$**, cioè fare una blind search esaustiva sul problema originale. A quel punto il task lo avremmo già risolto (visto che esploro il grafo mi basterebbe trovare i goal states), e l'euristica non servirebbe a niente.
>>
>>$\Rightarrow$ Serve un modo per costruire $\Theta'$ **senza mai toccare $\Theta$**, guardando solo la descrizione del task ($\|\Pi\|$, piccola) e lo spazio astratto ($\|\Theta'\|$, piccolo): è la **syntactic projection** $\Pi|_{P}$.

>[!info] **Definition** (syntactic projection)
>Let $\Pi=(V,A,c,I,G)$ be an FDR planning task, and let $P\subseteq V$. The **syntactic projection** of $\Pi$ to $P$ is the FDR planning task $\Pi|_{P}=(P,\ A|_{P},\ c,\ I|_{P},\ G|_{P})$ where:
>- $A|_{P}:=\{a|_{P}\mid a\in A\}$
>	- $pre_{a|_{P}}:=(pre_{a})|_{P}$
>	- $eff_{a|_{P}}:=(eff_{a})|_{P}$
>
>>[!tldr]
>>$\Pi|_{P}$ removes the variables outside $P$ from all constructs in the planning task description $\Pi$.

>[!info] **Theorem** (syntactic projection is equivalent to projection)
>Let $\Pi$ be an FDR planning task, and let $P\subseteq V$. Then $\Theta_{(\Pi|_{P})}$ is identical to $\Theta^{\pi_{P}}_{\Pi}$ except that actions $a$ in the latter become actions $a|_{P}$ in the former.
>
>>[!done] Proof
>>Easy from definition.
>
>>[!tldr]
>>The state space of the syntactic projection is (modulo action renaming) the same as the abstract state space of the projection.

^22841a
>[!info] Capiamo
>![[bangdesk.gif]]
>
>Le variabili fuori da $P$ non si tolgono dal grafo, ma dalla **descrizione del task**, *prima* di costruire qualsiasi grafo. Poi si fa una BFS su $\Pi|_{P}$, che genera direttamente solo stati astratti: $\Theta$ non viene mai costruito.
>
>Cosa succede togliendo una variabile $v\notin P$:
>- **precondizioni su $v$ spariscono** → l'azione diventa applicabile più spesso (transizioni spurie, $h$ ottimistica)
>- **effetti su $v$ spariscono** → se l'azione cambiava solo variabili fuori da $P$, diventa un **self-loop**
>- **goal su $v$ spariscono** → se nessuna variabile del goal è in $P$, ogni stato è goal e $h=0$
>
>$\Rightarrow$ Il teorema garantisce che il risultato è esattamente $\Theta^{\pi_{P}}$, ma ottenuto in tempo poly$(\|\Pi\|+\|\Theta'\|)$.

Using the [[#^22841a|Theorem]], we can compute pattern databases for FDR tasks $\Pi$ and patterns $P$. The following algorithm runs in time and space polynomial in $| |\Pi| |+| |\Theta'| |$.

>[!important] Pseudocode
>$$
>\begin{aligned}
>&\textbf{function } \textsf{computePDB}(\Pi, P) \\
>&\quad \Pi' \leftarrow \Pi|_{P} \\
>&\quad \text{Compute } \Theta' \leftarrow \Theta_{\Pi'} \text{ by a complete forward search (e.g., breadth-first)} \\
>&\quad \text{In the explicit graph } \Theta' \text{, add a new node } x \text{ with a } 0\text{-cost incoming edge} \\
>&\quad\quad \text{from every goal node} \\
>&\quad \text{Run Dijkstra starting from } x \text{ and traversing edges backwards, to compute} \\
>&\quad\quad \text{all cheapest paths to } x \text{ and thus the remaining costs } h^{*}_{\Theta'} \text{ in } \Theta' \\
>&\quad \textit{PDB} \leftarrow \text{a table containing all remaining costs in } \Theta' \\
>&\textbf{return } \textit{PDB}
>\end{aligned}
>$$

>[!info] Capiamo (summary algoritmo)
>![[bangdesk.gif]]
>
>1. **Proietta il task**: togli da $\Pi$ le variabili fuori da $P$ → $\Pi|_{P}$
>2. **Costruisci $\Theta'$**: BFS completa su $\Pi|_{P}$, che è piccolo
>3. **Nodo fittizio $x$**: collega ogni goal a $x$ con costo $0$, così "distanza dal goal più vicino" diventa "distanza da $x$"
>4. **Dijkstra all'indietro da $x$**: un'unica esecuzione dà $h^{*}_{\Theta'}$ di **tutti** gli stati astratti
>5. **Salva in tabella**: durante la ricerca $h^{P}(s)$ è solo un lookup

#### (II) Lookup step
At this point we need to create a table (vector) containing one cell for each node in the abstract graph.

Basic observations and method:
- During search, we do not need the actual abstract state space (transitions etc.): the PDB is the only piece of information necessary to represent $h^{P}$
	- $\Rightarrow$ *we can throw away the abstract state space $\Theta'$ once the PDB is computed*
	- $\Rightarrow$ space requirement during search is linear in the number of abstract states $S'$: the PDB has one table entry for each abstract state
- Design a perfect hash function mapping projected states $s|_{P}$ to numbers in the range $\{0,\dots,|S'|-1\}$
	- $\Rightarrow$ index the table PDB by these hash values: *given a state $s$ during search, to compute $h^{P}(s)$, map $\pi_{P}(s)=s|_{P}$ to its hash value and look up the table entry of PDB*


**Perfect hash function** $\approx$ numeral system over variable domains:
- let $P=\{v_{1},\dots,v_{k}\}$ be the pattern
- assume wlog that all variable domains are natural numbers counted from $0$, i.e., $D_{v}=\{0,1,\dots |D_{v}|-1\}$
- for all $i\in \{1,\dots,k\}$, we precompute $N_{i}:=\prod^{i-1}_{j=i}|D_{v_{j}}|$

>[!important] Pseudocode
>$$
>\begin{aligned}
>&\textbf{function } \textsf{PDBheuristic}(s) \\
>&\quad \textit{index} \leftarrow \textstyle\sum_{i=1}^{k} N_{i}\cdot s(v_{i}) \\
>&\textbf{return } \textit{PDB}[\textit{index}]
>\end{aligned}
>$$

>[!note]
>This lookup runs in time and space $O(k)$. This is very fast. For comparison, delete-relaxation heuristics need time $O(||\Pi||)$ per state.

>[!info] Capiamo
>![[bangdesk.gif]]
>Funziona come i numeri in base 10: in $347$ ogni cifra sta tra $0$ e $9$ e pesa $10$ volte la precedente, quindi numeri diversi hanno cifre diverse. Qui ogni variabile è una "cifra" con base $|D_{v_{i}}|$ e peso $N_{i}$.
>
>Tutte le cifre prima di $v_{i}$, messe insieme, valgono al massimo $N_{i}-1$ (come $99<100$): non arrivano mai a "riempire" un'unità di $v_{i}$ $\Rightarrow$ stati diversi hanno sempre indici diversi.
>
>| | $v_{1}$ | $v_{2}$ | $v_{3}$ | $v_{4}$ |
>|---|---|---|---|---|
>| base $\|D_{v_{i}}\|$ | 4 | 2 | 3 | 2 |
>| $N_{i}$ | 1 | 4 | 8 | 24 |
>| $s$ | 3 | 0 | 1 | 1 |
>
>$index(s)=(1\cdot3)+(4\cdot0)+(8\cdot1)+(24\cdot1)=35$
>$N_{i}$ = prodotto delle basi **precedenti** $\Rightarrow$ $N_{i+1}=N_{i}\cdot|D_{v_{i}}|$
>
>Tutte le cifre prima di $v_{i}$, messe insieme, valgono al massimo $N_{i}-1$ (come $99<100$), quindi non arrivano mai a "riempire" un'unità di $v_{i}$ $\Rightarrow$ stati diversi hanno sempre indici diversi.
>
>$\Rightarrow$ I $4\cdot2\cdot3\cdot2=48$ stati astratti finiscono esattamente negli indici $0,\dots,47$: niente collisioni e niente buchi.

>[!example] Lookup step: "Logistics mal anders"
>Abstraction induced by $\pi_{\{package,\ truck\ A\}}$:
>![[Pasted image 20261126193944.png|300]]
>
>- $P=\{v_{1},v_{2}\}$ with $v_{1}=package$, $v_{2}=truck\ A$
>	- $D_{v_{1}}=\{L,R,A,B\}\approx\{0,1,2,3\}$ $\Rightarrow$ $N_{1}=1$
>	- $D_{v_{2}}=\{L,R\}\approx\{0,1\}$ $\Rightarrow$ $N_{2}=|D_{v_{1}}|=4$
>- $index(s)=1\cdot s(package)+4\cdot s(truck\ A)$
>
>| abstract state | LL | RL | AL | BL | LR | RR | AR | BR |
>|---|---|---|---|---|---|---|---|---|
>| **index** | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
>| **value** | 2 | 0 | 2 | 1 | 2 | 0 | 1 | 1 |
>
>>[!tip]
>>Truck $B$ is projected away, so $pickup_{B\dots}$/$drop_{B\dots}$ are always applicable: e.g. $LL$ has value $2$ ($pickup_{BL}$, $drop_{BR}$).

>[!example] Lookup step: Australia
>![[tsp.png|300]]
>
>- $V$: $at:\{Sy,Ad,Br,Pe,Da\}$; $v\_x:\{T,F\}$ for $x\in\{Sy,Ad,Br,Pe,Da\}$
>- $A$: $drive\_xy$ where $x,y$ have a road
>	- costs: $Sy\leftrightarrow Br:1$, $Sy\leftrightarrow Ad:1.5$, $Ad\leftrightarrow Pe:3.5$, $Ad\leftrightarrow Da:4$
>- $I$: $\{at=Sy,\ v\_Sy=T,\ v\_x=F \text{ for } x\neq Sy\}$
>- $G$: $\{at=Sy,\ v\_x=T \text{ for all } x\}$
>
>Pattern $P=\{v_{1}=v\_Br,\ v_{2}=v\_Pe,\ v_{3}=v\_Da\}$:
>- $D_{v\_Br}=D_{v\_Pe}=D_{v\_Da}=\{F,T\}\approx\{0,1\}$ $\Rightarrow$ $N_{1}=1$, $N_{2}=2$, $N_{3}=4$
>- $index(s)=1\cdot s(v\_Br)+2\cdot s(v\_Pe)+4\cdot s(v\_Da)$
>
>| abstract state | FFF | TFF | FTF | TTF | FFT | TFT | FTT | TTT |
>|---|---|---|---|---|---|---|---|---|
>| **index** | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 |
>| **value** | 8.5 | 7.5 | 5 | 4 | 4.5 | 3.5 | 1 | 0 |
>
>>[!tip]
>>$at$ is projected away, so $drive\_xy$ just sets $v\_y=T$, from anywhere. Each value is the sum of the **cheapest road into each unvisited city**: e.g. $FFF=1+3.5+4=8.5$ ($Br$, $Pe$, $Da$).

---
## Orthogonal PDBs

### Pattern collections

>[!question] Pattern collections? Why and how?
>- The **space requirements for a PDB grow exponentially** with the number of state variables in the pattern
>	- $\Rightarrow$ $P$ must be small, severely limiting the usefulness of single-PDB heuristics $h^{P}$ for large planning tasks
>- To overcome this limitation, planners using PDBs can work with **collections** of multiple patterns
>- Given $h^{P_{1}}$ and $h^{P_{2}}$, we can always get an admissible heuristic dominating each of them by $\max\{h^{P_{1}},h^{P_{2}}\}$
>- A combination that would be much preferable: $h^{P_{1}}+h^{P_{2}}$
>	- $\Rightarrow$ but for this to be admissible, **$h^{P_{1}}$ and $h^{P_{2}}$ must be additive**

>[!info] **Lemma** (actions affecting PDBs)
>Let $P$ be a pattern for an FDR planning task $\Pi$, and let $a$ be an action in $\Pi$. Then *$a$ affects $\pi_{P}$ iff there exists a variable $v\in P$ on which $\mathit{eff}_{a}$ is defined*.
>
>>[!done] Proof
>>Consider the syntactic projection $\Pi|_{P}$.
>>
>>**$\Rightarrow$** if $a$ has no effect on $P$, then its effect in $\Pi|_{P}$ is empty, so it can induce only self-loops.
>>
>>**$\Leftarrow$** design a state $s$ in $\Pi|_{P}$ by projecting $\mathit{pre}_{a}$ onto $P$ and filling up the remaining $P$-values to be arbitrary but different from $\mathit{eff}_{a}$. Then $a$ induces the non-self-loop transition $(s,a,s[\![a]\!])$ in $\Pi|_{P}$ (as $\mathit{pre}_{a}(v)\neq \mathit{eff}_{a}(v)$ whenever both are defined).

### Orthogonal patterns

>[!info] **Theorem** (orthogonal patterns)
>Let $P_{1}$ and $P_{2}$ be patterns for an FDR planning task $\Pi$. Then $\pi_{P_{1}}$ and $\pi_{P_{2}}$ are *orthogonal iff there exists no action $a$ in $\Pi$ such that $\mathit{eff}_{a}$ is defined for variables $v,v'$ where $v\in P_{1}$ and $v'\in P_{2}$*.
>
>>[!done] Proof
>>Direct from the lemma above.
>
>In this situation, we also call the patterns $P_{1}$ and $P_{2}$ themselves **orthogonal**.

^51b023

>[!question] Orthogonality and (non-)intersecting patterns
>- If $P_{1}\cap P_{2}\neq\emptyset$, can $P_{1}$ and $P_{2}$ be orthogonal?
>	- **No**, except if all $v\in P_{1}\cap P_{2}$ are not affected by any action, which is pathological
>- If $P_{1}\cap P_{2}=\emptyset$, are $P_{1}$ and $P_{2}$ necessarily orthogonal?
>	- **No**, there may be an action that modifies a variable in $P_{1}$ and a variable in $P_{2}$

>[!example] 
>![[tsp.png|300]]
>
>Every $drive\_xy$ has $\mathit{eff}$: $at=y,\ v\_y=T$.
>
>- $P_{1}=\{v\_Br\}$ and $P_{2}=\{v\_Pe,\ v\_Da\}$ are **orthogonal**
>	- $\Rightarrow$ every action affects at most one of them: driving to $Br$ only touches $P_{1}$, driving to $Pe$/$Da$ only touches $P_{2}$
>- $P'_{1}=\{v\_Br,\ at\}$ and $P_{2}=\{v\_Pe,\ v\_Da\}$ are **not orthogonal**
>	- $\Rightarrow$ e.g. driving to $Pe$ sets $at=Pe$ ($\in P'_{1}$) and $v\_Pe=T$ ($\in P_{2}$): a cross-effect
>
>>[!tip]
>>$at$ is changed by **every** drive action, so any pattern containing $at$ is orthogonal only to patterns with no variable changed by a drive, i.e. basically to nothing useful.

#### Additivity

>[!info] **Terminology**
>Intead of “collection of pairwise orthogonal patterns”, we also say *orthogonal pattern collection* or *orthogonal collection*.

>[!info] **Corollary** (orthogonal patterns are additive)
>Let $\{P_{1}, \dots, P_{k}\}$ be an orthogonal pattern collection for an FDR planning task $\Pi$. Then $\sum^{k}_{i=1}h^{P_{i}}$ is consistent and goal-aware, and thus also admissible and safe.
>
>>[!done] Proof
>>By the [[#^51b023|Theorem]], the abstractions $\pi_{P_{1}},\dots,\pi_{P_{k}}$ are pairwise orthogonal. The claim follows with the [[Abstraction heuristics#^e85763|Theorem “orthogonal abstractions are additive”]]
>
>
>>[!info] Observation
>>A single large pattern is more expensive, but is better informativity-wise.
>>- **At least as good:** for orthogonal $P_{i}$ each of which is contained in $P$, we have $\textstyle\sum_{i=1}^{k}h^{P_{i}}\leq h^{P}$ (see [[#^b79029|next theorem]])
>>- **Potentially better:** e.g. "Logistics mal anders" with $P_{1}=\{package,\ truck\ A\}$, $P_{2}=\{truck\ B\}$:
>>	- $h^{P_{1}}(LRR)=2$
>>	- $h^{P_{2}}(LRR)=0$
>>	- $h^{P_{1}\cup P_{2}}(LRR)=4$
>>
>>>[!tip]
>>$P_{1}\cup P_{2}=V$, so $h^{P_{1}\cup P_{2}}=h^{*}$. Separately, $P_{1}$ ignores where truck $B$ is (so $pickup_{BL}$, $drop_{BR}$ look always applicable), and $P_{2}$ contains no goal variable (so $h^{P_{2}}=0$).
^3f14f9

>[!info] **Theorem** (large patterns dominate orthogonal smaller ones)
>Let $\{P_{1},\dots,P_{k}\}$ be an orthogonal pattern collection for an FDR planning task, and **let $P$ be a pattern with $P_{i}\subseteq P$ for all $i\in\{1,\dots,k\}$. Then $\textstyle\sum_{i=1}^{k}h^{P_{i}}\leq h^{P}$**.
>
>>[!done] Proof
>>Consider the syntactic projection onto $P$, i.e. the FDR planning task $\Pi|_{P}$.
>>
>>Because $P_{i}\subseteq P$, we can perceive each $\pi_{P_{i}}$ as a projection of $\Pi|_{P}$, and hence perceive the corresponding heuristics $h^{P_{i}}$ as abstraction heuristics on $\Pi|_{P}$.
>>
>>Clearly, $\{P_{1},\dots,P_{k}\}$ on $\Pi|_{P}$ is still an orthogonal pattern collection. Hence, as $h^{P}$ is defined as $h^{*}$ on $\Pi|_{P}$, the claim follows from the [[#^3f14f9|theorem orthogonal abstractions are additive]] (the sum is admissible, i.e. $\leq h^{*}$ on $\Pi|_{P}$).

^b79029
### How to exploit orthogonal patterns
Given a **pattern collection** $\mathcal{C}$: how to get the best possible lower bound?
1. Build the **compatibility graph** for $\mathcal{C}$
	- the vertices are the patterns $P\in\mathcal{C}$
	- arcs between pairs of orthogonal patterns
2. Compute all **maximal cliques** (maximal orthogonal sub-collections)
	- computing maximal cliques is **NP**-hard, but this graph will typically be small

>[!example] Abstract example
>FDR task with $V=\{v_{1},v_{2},v_{3}\}$, pattern collection $\mathcal{C}=\{P_{1},\dots,P_{4}\}$ with:
>- $P_{1}=\{v_{1},v_{2}\}$, $P_{2}=\{v_{1}\}$, $P_{3}=\{v_{2}\}$, $P_{4}=\{v_{3}\}$
>
>There are actions affecting each individual variable, and the only actions affecting several variables affect $v_{1}$ and $v_{3}$.
>
>![[Pasted image 20261126204433.png|350]]
>
>$\Rightarrow$ Maximal cliques in the compatibility graph: $\{P_{1}\}$, $\{P_{2},P_{3}\}$, $\{P_{3},P_{4}\}$
>
>>[!tip]
>>- $P_{1}$ intersects $P_{2}$ and $P_{3}$, and has a cross-effect with $P_{4}$ ($v_{1}$ and $v_{3}$) $\Rightarrow$ alone
>>- $P_{2}$ and $P_{4}$ have a cross-effect ($v_{1}$ and $v_{3}$) $\Rightarrow$ not orthogonal
>>- every other pair is orthogonal

#### The cliques heuristic

>[!info] **Definition** (cliques heuristic)
>Let $\Pi$ be an FDR planning task, let $\mathcal{C}$ be a pattern collection for $\Pi$, and let $cliques(\mathcal{C})$ be the set of all maximal cliques in the compatibility graph for $\mathcal{C}$. Then the **cliques heuristic** $h^{\mathcal{C}}$ for $\mathcal{C}$ is defined as
>$$
>h^{\mathcal{C}}(s)=\max_{\mathcal{D}\in cliques(\mathcal{C})}\ \sum_{P\in\mathcal{D}}h^{P}(s)
>$$
>
>>[!tldr]
>>The cliques heuristic maximizes over all largest orthogonal subsets of our pattern collection. It is consistent and goal-aware.

>[!example] Same abstract example
>Maximal cliques in the compatibility graph: $\{P_{1}\}$, $\{P_{2},P_{3}\}$, $\{P_{3},P_{4}\}$
>
>$$
>\begin{aligned}
>h^{\mathcal{C}}&=\max\{h^{P_{1}},\ h^{P_{2}}+h^{P_{3}},\ h^{P_{3}}+h^{P_{4}}\}\\
>&=\max\{h^{\{v_{1},v_{2}\}},\ h^{\{v_{1}\}}+h^{\{v_{2}\}},\ h^{\{v_{2}\}}+h^{\{v_{3}\}}\}
>\end{aligned}
>$$

>[!example] The Australia example again
>![[tsp.png|300]]
>
>- $V$: $at:\{Sy,Ad,Br,Pe,Da\}$; $v\_x:\{T,F\}$ for $x\in\{Sy,Ad,Br,Pe,Da\}$
>- $A$: $drive\_xy$ where $x,y$ have a road
>	- costs: $Sy\leftrightarrow Br:1$, $Sy\leftrightarrow Ad:1.5$, $Ad\leftrightarrow Pe:3.5$, $Ad\leftrightarrow Da:4$
>- $I$: $\{at=Sy,\ v\_Sy=T,\ v\_x=F \text{ for } x\neq Sy\}$
>- $G$: $\{at=Sy,\ v\_x=T \text{ for all } x\}$
>
>Observations:
>- $P_{1}=\{v\_Br\}$, $P_{2}=\{v\_Pe,\ v\_Da\}$
>	- $h^{P_{1}}(I)=1$, $h^{P_{2}}(I)=4+3.5=7.5$
>- $\mathcal{C}=\{P_{1},P_{2}\}$: $P_{1}$ and $P_{2}$ are orthogonal
>	- $\Rightarrow$ $h^{\mathcal{C}}(I)=h^{P_{1}}(I)+h^{P_{2}}(I)=1+7.5=8.5$
>- $P'_{1}=\{v\_Br,\ at\}$: $h^{P'_{1}}(I)=2$, because now we have to drive back from $Br$ to satisfy the goal $at=Sy$
>- $\mathcal{C}=\{P'_{1},P_{2}\}$: $P'_{1}$ and $P_{2}$ are **not** orthogonal
>	- $\Rightarrow$ $h^{\mathcal{C}}(I)=\max\{h^{P'_{1}}(I),\ h^{P_{2}}(I)\}=7.5$
>
>>[!tip]
>>Refining a pattern brings its $h$ closer to $h^{*}$, but it may **break orthogonality** with the other patterns of the collection: the gain can be useless, or even detrimental.
>>
>>E.g. $P_{1}\to P'_{1}$: $h$ goes from $1$ to $2$, but $h^{\mathcal{C}}(I)$ drops from $8.5$ to $7.5$, because we can only take the max instead of the sum.

#### On the power of the cliques heuristic

>[!tldr]
>The cliques heuristic $h^{\mathcal{C}}$ is the best possible admissible heuristic we can derive from $\mathcal{C}$ **based on exploiting only orthogonality**.

>[!example] The Attack of the Zombie Tomatoes
>- $I$: $A$ and $B$ both false
>- $G$: $A$ and $B$ both true
>- $A$:
>	- $carA$: effect $A$, cost $1$
>	- $carB$: effect $B$, cost $1$
>	- $fancyCar$: effect $A$ and $B$, cost $1.5$
>- Patterns $P_{1}=\{A\}$ and $P_{2}=\{B\}$
>
>$\Rightarrow$ $fancyCar$ affects both patterns, so $P_{1}$ and $P_{2}$ are **not** orthogonal: $h^{\mathcal{C}}(I)=\max\{1,1\}=1<1.5=h^{*}(I)$
>
>>[!tip]
>>**Cost partitioning** is a more general technique for combining heuristics, which always dominates $h^{\mathcal{C}}$ and yields the perfect lower bound $1.5$ here.
>>E.g. split the cost of $fancyCar$ as $0.75$ in $P_{1}$ and $0.75$ in $P_{2}$ (and give $carA$ cost $0$ in $P_{2}$, $carB$ cost $0$ in $P_{1}$): then $h^{P_{1}}(I)=h^{P_{2}}(I)=0.75$ and the sum $1.5$ is admissible.

---
## Summary
- **Pattern database (PDB) heuristics** are abstraction heuristics based on projection to a subset of variables: the **pattern**. For FDR tasks, they can easily be implemented via **syntactic projection** on the task representation.
- **Pattern databases** are **lookup tables** that store heuristic values, indexed by **perfect hash values** for projected states.
- Two patterns are **orthogonal** if no action affects variables from both. The heuristics are then **additive**, i.e., their sum is admissible.
- Given a **pattern collection**, the **cliques heuristic** sums heuristic values from all orthogonal pattern subsets, and maximizes over these sums.