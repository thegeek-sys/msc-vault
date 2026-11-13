---
Class: "[[Artificial Intelligence]]"
Sem: WiSe 26/27
---
---
## Index
- [[#A regression based characterization of $h^{*}$|A regression based characterization of $h^{*}$]]
- [[#$h^{1}$|$h^{1}$]]
- [[#The general definition ($h^{m}$)|The general definition ($h^{m}$)]]
	- [[#The general definition ($h^{m}$)#Properties|Properties]]
- [[#Dynamic programming|Dynamic programming]]
	- [[#Dynamic programming#Iterative definition of $h^{m}$|Iterative definition of $h^{m}$]]
		- [[#Iterative definition of $h^{m}$#Fixed point formulation (for reference)|Fixed point formulation (for reference)]]
- [[#FDR|FDR]]
---
## A regression based characterization of $h^{*}$

>[!info] **Definition** ($r^{*}$)
>Let $\Pi=(P,A,c,I,G)$ be a STRIPS planning task. The **perfect regression heuristic $r^{*}$** for $\Pi$ is the function $r^{*}(s):=r^{*}(s,G)$ where $r^{*}(s,g)$ is the point-wise greatest function (needed to correctly handle $0$-cost actions, probable final exam exercise on this) that satisfies $r^{*}(s,g)=$
>$$
>r^*(s, g) =
>\begin{cases}
>0 & g \subseteq s \\
>\min_{a \in A,\, \mathit{regr}(g,a) \neq \bot} \left[ c(a) + r^*(s, \mathit{regr}(g,a)) \right] & \text{otherwise}
>\end{cases}
>$$
>
>>[!tldr]
>>The cost of achieving a subgoal $g$ from $s$ is $0$ if $g$ is true in $s$; else, it is the minimum cost of using any action $a$ to achieve $g$.

>[!info] **Proposition**
>Let $\Pi=(P,A,c,I,G)$ be a STRIPS planning task. Then $r^{*}=h^{*}$

>[!tip] Capiamo
>![[bangdesk.gif]]
>Partendo dal goal $G$, $r^*$ procede **all'indietro**: il costo di un sottobiettivo $g$ a partire da $s$ è $0$ se $g$ è già vero in $s$ ($g \subseteq s$); altrimenti è il minimo, su tutte le azioni $a$ che possono rendere vero $g$, di $c(a)$ più il costo del sottobiettivo regredito $regr(g,a)$, cioè ciò che deve valere *prima* di applicare $a$. Lo stato $s$ resta fisso; cambia solo il sottobiettivo, finché non è soddisfatto da $s$.

---
## $h^{1}$
>[!info] **Definition** ($h^{1}$)
>Let $\Pi=(P,A,c,I,G)$ be a STRIPS planning task. The **critical path heuristic $h^1$** for $\Pi$ is the function $h^1(s) := h^1(s,G)$ where $h^1(s,g)$ is the point-wise greatest function that satisfies $h^1(s,g) =$
>$$
>h^1(s, g) = \begin{cases}
>0 & g \subseteq s \\
>\min_{a \in A,\, \mathit{regr}(g,a) \neq \bot} \left[ c(a) + h^1(s, \mathit{regr}(g,a)) \right] & |g| = 1 \\
>\max_{g' \in g} h^1(s, \{g'\}) & |g| > 1
>\end{cases}
>$$
>
>> [!tldr]
>> For singleton subgoals $g$, use regression as in $r^*$. For subgoal sets $g$, use the cost of the most costly singleton subgoal $g' \in g$.
>
>>[!question] "Critical path"?
>>No formal def here; to give a rough intuition:
>>"Path" $= g_1 \xrightarrow{a_1} g_2 \dots g_{n-1} \xrightarrow{a_{n-1}} g_n$ where $g_1 \subseteq s$, $g_n \subseteq G$, $g_i \neq g_j$, and $g_i \subseteq \mathit{regr}(g_{i+1}, a_i)$.
>>
>>"Critical path" $=$ Cheapest path through the most costly $g_i$ ($g_{i}$ are the single facts).

>[!tip] Differenze con $r^{*}$
>$r^{*}$ regredisce il sottobiettivo g **intero**. Se $g = \{p, q\}$, cerca un’azione che renda vero (una parte di) $g$ e regredisce tutto l’insieme. Così tiene conto che _tutti_ i fatti di $g$ vanno ottenuti, e il costo di ciascuno si somma lungo il piano.
>
>$h^{*}$ regredisce solo i **singleton**. Quando incontra un insieme con più fatti lo spezza e prende il costo del fatto **più costoso** (il max). È come dire: “per avere $\{p, q\}$ mi basta pagare il più caro tra $p$ e $q$”. In pratica fa finta che, ottenuto il fatto più difficile, gli altri arrivino gratis.
>
>Il minimo va bene: è ammissibile e non sovrastima mai. Il problema è che è **molto meno informativo**, e un’euristica poco informativa aiuta poco la ricerca.
>
>>[!example]
>>Un esempio: goal g = {p, q}, con p già vero in s e q che costa 100.
>>- **min**: $\min(0, 100) = 0$
>>- **max**: $\max(0, 100) = 100$
>>- costo vero: almeno 100

>[!warning]
>![[bangdesk.gif]]
>
>Se si ha un fatto contenuto sia negli $add$ che nelle $pre$ è l’implementazione dell’algoritmo è eager questo va in loop. Infatti:
>$$
>h^{1}(s, \{p\}) = \min(\dots, c(a) + h^{1}(s, \{p\}), \dots)
>$$
>
>Invece in termini di formula matematica non si hanno problemi infatti $c(a)+ h^{1}(s, \{p\})>h^{1}(s,\{p\})$ (quindi non verrebbe mai selezionato dalla funzione di $\min$) a patto che $c(a)>0$.
>
>Per le azioni con costo $0$, l’equazione diventa $h^{1}(s,\{p\}=0+h^{1}(s,\{p\}))$ che è soddisfatta per qualsiasi valore ma $p$ è irraggiungibile quindi il valore corretto è $\infty$. Partendo dal valore più grande si ottiene $\infty$.
>
>Partire da $\infty$ e scendere fa sì che ci si fermi al **più grande** punto fisso, cioè esattamente quello richiesto dalla definizione. Se partissi da $0$, ti fermeresti subito alla soluzione sbagliata.
>
>>[!example]
>>Ciclo con costo 0:
>>- $h(q) = h(p)$
>>- $h(p) = min(3, h(p))$
>>
>>Qui va bene **qualsiasi** $h(p) \leq 3$, perché $min(3, 1) = 1$, $min(3, 0) = 0$ e così via. L’equazione non distingue più il valore vero (3) da quelli falsi (0, 1, 2, …), perché il ciclo a costo 0 permette a p di “pagarsi da solo”. Partendo da infinito, la definizione si ferma al massimo punto fisso ($3$).

>[!example] $h^{1}$ in "TSP" in Australia
>
>![[tsp.png|400]]
>- $P$: $at\_x$ for $x \in \{Sy, Ad, Br, Pe, Da\}$; $v\_x$ for $x \in \{Sy, Ad, Br, Pe, Da\}$.
>- $A$: $drive\_xy$ where $x, y$ have a road.
>$$
>c(drive\_xy) = \begin{cases}
>1 & \{x,y\} = \{Sy, Br\} \\
>1.5 & \{x,y\} = \{Sy, Ad\} \\
>3.5 & \{x,y\} = \{Ad, Pe\} \\
>4 & \{x,y\} = \{Ad, Da\}
>\end{cases}
>$$
>- $I$: $at\_Sy, v\_Sy$; $G$: $at\_Sy, v\_x$ for all $x$.
>
>$$
>h^1(I) = h^1(I,G) = h^1(I, \{at\_Sy, v\_Sy, v\_Ad, v\_Br, v\_Pe, v\_Da\}) = \max(h^1(I,\{at\_Sy\}), \dots, h^1(I,\{v\_Da\}))
>$$
>- $h^1(I,\{at\_Sy\}) = h^1(I,\{v\_Sy\}) = 0$
>- $h^1(I,\{v\_Da\}) = 4 + h^1(I, \mathit{regr}(\{v\_Da\}, drive\_AdDa)) = 4 + h^1(I,\{at\_Ad\})$
>- $h^1(I,\{at\_Ad\}) = \min(3.5 + h^1(I,\{at\_Pe\}),\ 4 + h^1(I,\{at\_Da\}),\ 1.5 + h^1(I,\{at\_Sy\})) = 1.5$
>
>So $h^1(I,\{v\_Da\}) = 5.5$. Further, $h^1(I,\{v\_Pe\}) = 5$ and $h^1(I,\{v\_Br\}) = 1$, hence $h^1(I) = 5.5$.
>
>The critical path is $at\_Sy \xrightarrow{drive(Sy,Ad)} at\_Ad \xrightarrow{drive(Ad,Da)} at\_Da$.
>
>>[!tip] Capiamo
>>Il fatto più costoso è $v\_Da$ (5.5), quindi $h^1$ guarda solo la catena più economica per visitare Darwin e ignora tutto il resto. Il tour vero (Sy→Br→Sy→Ad→Pe→Ad→Da→Ad→Sy) costa $20$: $h^1$ non vede che bisogna visitare **tutte** le città e poi tornare a Sydney, perché valuta ogni fatto da solo.

---
## The general definition ($h^{m}$)
$h^{1}$ is one member of the following family of heuristic functions, characterized by the size of subgoals above which they approximate.

>[!info] **Definition** ($h^{m}$)
>Let $\Pi=(P,A,c,I,G)$ be a STRIPS planning task, and let $m \in \mathbb{N}$. The **critical path heuristic $h^m$** for $\Pi$ is the function $h^m(s) := h^m(s,G)$ where $h^m(s,g)$ is the point-wise greatest function that satisfies $h^m(s,g) =$
>$$
>h^m(s, g) = \begin{cases}
>0 & g \subseteq s \\
>\min_{a \in A,\, \mathit{regr}(g,a) \neq \bot} \left[ c(a) + h^m(s, \mathit{regr}(g,a)) \right] & 1 \leq |g| \leq m \\
>\max_{g' \subseteq g,\, |g'| = m} h^m(s, g') & |g| > m
>\end{cases}
>$$
>
>> [!tldr]
>> For subgoal sets $1 \leq |g| \leq m$, use regression as in $r^*$. For subgoal sets $|g| > m$, use the cost of the most costly $m$-subset $g'$.

### Properties

>[!info] **Proposition** ($h^{m}$ is Admissible)
>$h^m$ is consistent and goal-aware, and thus also admissible and safe.
>
>>[!done] Proof sketch
>>Goal-awareness is obvious. We need to prove that $h^m(s) \leq h^m(s') + c(a)$ for all transitions $s \xrightarrow{a} s'$. Since $s \supseteq \mathit{regr}(s', a)$, we can add $a$ to a critical path $\vec{p}$ for $h^m(s')$ to obtain an upper bound on $h^m(s)$.
>
>> [!tldr]
>> $h^m$ is admissible because it is always more difficult to achieve larger subgoals (so $m$-subsets can only be cheaper).

>[!info] **Proposition** ($h^{m}$ gets more accurate as $m$ grows)
>$h^{m+1}$ dominates $h^m$.
>
>>[!done] Proof intuition
>>"It is always more difficult to achieve larger subgoals."

>[!info] **Proposition** ($h^{m}$ is perfect in the limit)
>There exists $m$ s.t. $h^m = h^*$.
>
>>[!done] Proof
>>Setting $m := |P|$, the case $|g| > m$ will never be used, so $h^m = r^*$.

---
## Dynamic programming
The basic idea is:

$$
\begin{gathered}
\text{"Consider all size-}\leq m\text{ subgoals } g\text{. Initialize } h^{m}(s,g) \text{ to } 0 \text{ if } g\subseteq s\text{,} \text{and to } \infty \text{ otherwise.} \\[1em]
\text{Then keep updating the value of each } g \text{ based on actions applied} \\
\text{to the values computed so far, until the values converge."}
\end{gathered}
$$

Next up:
- we start with an iterative definition of hm that captures this computation.
- we define a dynamic programming algorithm that corresponds to this iterative definition.
- we look at some examples in detail.

### Iterative definition of $h^{m}$

>[!info] **Definition** (Iterative $h^{m}$)
>Let $\Pi=(P,A,c,I,G)$ be a STRIPS planning task, and let $m \in \mathbb{N}$. The **iterative $h^m$ heuristic $h^m_i$** is defined by $h^m_0(s,g) :=$
>$$
>h^m_0(s, g) = \begin{cases}
>0 & g \subseteq s \\
>\infty & \text{otherwise}
>\end{cases}
>$$
>and $h^m_{i+1}(s,g) :=$
>$$
>h^m_{i+1}(s, g) = \begin{cases}
>0 & g \subseteq s \\
>\min\left[h^m_i(s,g),\ \min_{a \in A,\, \mathit{regr}(g,a) \neq \bot} \left[ c(a) + h^m_i(s, \mathit{regr}(g,a)) \right]\right] & 1 \leq |g| \leq m \\
>\max_{g' \subseteq g,\, |g'| = m} h^m_{i+1}(s, g') & |g| > m
>\end{cases}
>$$
>
>>[!note]
>>$h^{m}_{i+1}$ and $h^{m}_{i}$ are two succeeding rows of the table (i.e., two succeeding iterations of the algorithm). $i$ represents the max number of regression steps you can apply at each iteration.

>[!info] **Proposition**
>Let $\Pi=(P,A,c,I,G)$ be a STRIPS planning task. Then the series $\{h^m_i\}_{i=0,\dots}$ converges to $h^m$.
>
>>[!done] Proof sketch
>>(i) **Convergence**: If $h^m_{i+1}(s,g) \neq h^m_i(s,g)$, then $h^m_{i+1}(s,g) < h^m_i(s,g)$; that can happen only finitely often because each decrease is due to a new path for $g$.
>>
>>(ii) If $h^m_{i+1} = h^m_i$ then $h^m_i$ satisfies the $h^m$ equation (direct from definition).
>>
>>(iii) No function greater than $h^m_i$ at any point can satisfy the $h^m$ equation (easy by induction over $i$).
>
>>[!tip] Capiamo
>>Questa è la versione "algoritmica" di $h^m$: parte da $\infty$ (tranne i sottobiettivi già veri in $s$, che valgono $0$) e a ogni iterazione abbassa i valori usando le azioni. Il $\min$ con $h^m_i(s,g)$ garantisce che i valori non salgano mai. Ci si ferma quando nulla cambia, e il punto fisso così raggiunto è il **più grande**, cioè proprio la point-wise greatest function della definizione.

>[!important] Pseudocode
>$$
>\begin{array}{l}
>\textbf{new } \text{table } T^m_0(g), \text{ for } g \subseteq P \text{ with } 1 \leq |g| \leq m \\
>\textbf{for all } g \subseteq P \text{ with } |g| \leq m: \quad T^m_0(g) := \begin{cases} 0 & g \subseteq s \\ \infty & \text{otherwise} \end{cases} \\
>\\
>\textbf{fn } {\color{yellow}\mathit{Cost}_i}(g) := \begin{cases} 0 & g \subseteq s \\ T^m_i(g) & 1 \leq |g| \leq m \\ \max_{g' \subseteq g,\, |g'| = m} T^m_i(g') & |g| > m \end{cases} \\
>\\
>\textbf{fn } {\color{yellow}\mathit{Next}_i}(g) := \min\left[\mathit{Cost}_i(g),\ \min_{a \in A,\, \mathit{regr}(g,a) \neq \bot} \left[ c(a) + \mathit{Cost}_i(\mathit{regr}(g,a)) \right]\right] \\
>\\
>i := 0 \\
>\textbf{do forever}: \\
>\quad \textbf{new } \text{table } T^m_{i+1}(g), \text{ for } g \subseteq P \text{ with } 1 \leq |g| \leq m \\
>\quad \textbf{for all } g \subseteq P \text{ with } |g| \leq m: \quad T^m_{i+1}(g) := \mathit{Next}_i(g) \\
>\quad \textbf{if } T^m_{i+1} = T^m_i \textbf{ then stop endif} \\
>\quad i := i + 1 \\
>\textbf{enddo}
>\end{array}
>$$

>[!info] **Proposition**
>$h^m_i(s,g) = Cost_i(g)$ for all $i$ and $g$. (Proof is easy.)
>
>>[!done] Proof
>>By induction on $i$, for every $g$ (of any size).
>>
>>**Base ($i = 0$).**
>>- If $|g| \leq m$: $Cost_0(g) = T^m_0(g)$, which by definition is $0$ if $g \subseteq s$ and $\infty$ otherwise, i.e. $h^m_0(s,g)$.
>>- If $|g| > m$: $Cost_0(g) = \max_{g'} T^m_0(g')$ over $g' \subseteq g$, $|g'| = m$. If $g \subseteq s$, every $g'$ has value $0$, so the max is $0$. Otherwise there is a fact $p \in g \setminus s$ and some $g' \ni p$ with value $\infty$, so the max is $\infty$. Either way it equals $h^m_0(s,g)$.
>>
>>**Step ($i \to i+1$).** Assume $h^m_i(s,\cdot) = Cost_i(\cdot)$ on all subgoals.
>>- If $g \subseteq s$: $h^m_{i+1}(s,g) = 0$, and $Cost_{i+1}(g) = 0$ as well (first case of its definition).
>>- If $1 \leq |g| \leq m$ and $g \not\subseteq s$:
>>$$
>>\begin{aligned}
>>h^m_{i+1}(s,g) &= \min\left[h^m_i(s,g),\ \min_{a}\ [c(a) + h^m_i(s, \mathit{regr}(g,a))]\right] \\
>>&= \min\left[Cost_i(g),\ \min_{a}\ [c(a) + Cost_i(\mathit{regr}(g,a))]\right] \\
>>&= Next_i(g) = T^m_{i+1}(g) = Cost_{i+1}(g)
>>\end{aligned}
>>$$
>>where the second line uses the induction hypothesis (also on $\mathit{regr}(g,a)$, which may have size $> m$), and the last two equalities follow from the definitions in the algorithm.
>>- If $|g| > m$ and $g \not\subseteq s$: $h^m_{i+1}(s,g) = \max_{g'} h^m_{i+1}(s,g')$. Each $g'$ has $|g'| = m$, so by the previous case $h^m_{i+1}(s,g') = T^m_{i+1}(g')$, and the max is exactly $Cost_{i+1}(g)$. $\blacksquare$
>
>>[!warning]
>>This is *very* inefficient! (Optimized for readability.) We can instead maintain the frontier of cheapest $m$-tuples reached so far.

>[!example] $h^1$ in "Logistics" ($m=1$)
>![[logistics.png|300]]
>
>- $P$: $tx$ for $x \in \{A,B,C,D\}$; $px$ for $x \in \{A,B,C,D,T\}$.
>- $I = \{tA, pC\}$; $G = \{tA, pD\}$.
>- $A$ (unit costs): $dr_{xy}$, $lo_x$, $ul_x$. E.g. $lo_x$: pre $tx, px$; add $pT$; del $px$.
>
>Tables $T^1_i$ (in pedice l'azione che ha prodotto il valore):
>
>| $i$ | tA | tB | tC | tD | pT | pA | pB | pC | pD |
>|---|---|---|---|---|---|---|---|---|---|
>| 0 | 0 | ∞ | ∞ | ∞ | ∞ | ∞ | ∞ | 0 | ∞ |
>| 1 | 0 | 1<sub>drAB</sub> | ∞ | ∞ | ∞ | ∞ | ∞ | 0 | ∞ |
>| 2 | 0 | 1 | 2<sub>drBC</sub> | ∞ | ∞ | ∞ | ∞ | 0 | ∞ |
>| 3 | 0 | 1 | 2 | 3<sub>drCD</sub> | 3<sub>loC</sub> | ∞ | ∞ | 0 | ∞ |
>| 4 | 0 | 1 | 2 | 3 | 3 | 4<sub>ulA</sub> | 4<sub>ulB</sub> | 0 | 4<sub>ulD</sub> |
>| 5 | 0 | 1 | 2 | 3 | 3 | 4 | 4 | 0 | 4 |
>
>Riga 5 = riga 4 → punto fisso. So $h^1(I) = \max(tA, pD) = \max(0, 4) = 4$.
>
>>[!note]- Azioni per riga
>>- $i=1$: $tB = 1 + tA(0)$ via $drAB$
>>- $i=2$: $tC = 1 + tB(1)$ via $drBC$
>>- $i=3$: $tD = 1 + tC(2)$ via $drCD$; $pT = 1 + \max(tC(2), pC(0))$ via $loC$
>>- $i=4$: $pA = 1 + \max(tA(0), pT(3))$ via $ulA$; $pB = 1 + \max(tB(1), pT(3))$ via $ulB$; $pD = 1 + \max(tD(3), pT(3))$ via $ulD$
>
>>[!tip] Capiamo
>>Con costi unitari ogni valore, una volta finito, non cambia più e coincide con il numero $i$ della riga in cui compare. Il critical path è $tA \to tB \to tC \to pT \to pD$.

>[!example] $h^2$ in "Dompteur" ($m=2$)
>- $P = \{alive, haveTiger, tamedTiger, haveJump\}$, in breve $\{A, hT, tT, J\}$.
>- $I = \{A\}$; $G = \{A, J\}$.
>- $A$:
>	- $getT$: pre $A$; add $hT$
>	- $tameT$: pre $A, hT$; add $tT$
>	- $jumpTT$ (jumpTamedTiger): pre $A, tT$; add $J$
>	- $jumpT$ (jumpTiger): pre $A, hT$; add $J$; del $A$
>
>| $i$ | A | hT | tT | J | A,hT | A,tT | A,J | hT,tT | hT,J | tT,J |
>|---|---|---|---|---|---|---|---|---|---|---|
>| 0 | 0 | ∞ | ∞ | ∞ | ∞ | ∞ | ∞ | ∞ | ∞ | ∞ |
>| 1 | 0 | 1<sub>getT</sub> | ∞ | ∞ | 1<sub>getT</sub> | ∞ | ∞ | ∞ | ∞ | ∞ |
>| 2 | 0 | 1 | 2<sub>tameT</sub> | 2<sub>jumpT</sub> | 1 | 2<sub>tameT</sub> | ∞ | 2<sub>tameT</sub> | 2<sub>jumpT</sub> | ∞ |
>| 3 | 0 | 1 | 2 | 2 | 1 | 2 | 3<sub>jumpTT</sub> | 2 | 2 | 3<sub>jumpTT</sub> |
>
>So $h^2(I) = h^2(I, \{A,J\}) = 3$, in contrast to $h^1(I) = \max(A, J) = 2$.
>
>>[!note]- Azioni per riga
>>- $i=1$: $hT = 1 + A(0)$ via $getT$; $\{A,hT\} = 1 + \{A\}(0)$ via $getT$
>>- $i=2$:
>>	- $tT = 1 + \{A,hT\}(1)$ via $tameT$
>>	- $J = 1 + \{A,hT\}(1)$ via $jumpT$
>>	- $\{A,tT\} = 1 + \{A,hT\}(1)$ via $tameT$
>>	- $\{hT,tT\} = 1 + \{A,hT\}(1)$ via $tameT$
>>	- $\{hT,J\} = 1 + \{A,hT\}(1)$ via $jumpT$
>>- $i=3$: $\{A,J\} = 1 + \{A,tT\}(2)$ via $jumpTT$; $\{tT,J\} = 1 + \{A,tT\}(2)$ via $jumpTT$
>
>>[!tip] Capiamo
>>Alla riga 2, $A$ e $J$ sono raggiunti entrambi ($A=0$, $J=2$), ma **non insieme**: $\{A,J\}$ resta $\infty$.
>>- $regr(\{A,J\}, jumpT) = \bot$, perché $jumpT$ cancella $A$.
>>- $regr(\{A,J\}, jumpTT) = \{A,tT\}$, che alla riga 1 vale $\infty$.
>>
>>È qui che $h^2$ vede un'interazione tra fatti che $h^1$ non vede.

>[!example] $h^1$ in "TSP" in Australia ($m=1$)
>![[tsp.png|400]]
>
>- $P$: $at\_x$ for $x \in \{Sy, Ad, Br, Pe, Da\}$; $v\_x$ for $x \in \{Sy, Ad, Br, Pe, Da\}$.
>- $A$: $drive\_xy$ where $x,y$ have a road; costs $Sy$–$Br$: $1$, $Sy$–$Ad$: $1.5$, $Ad$–$Pe$: $3.5$, $Ad$–$Da$: $4$.
>- $I = \{at\_Sy, v\_Sy\}$; $G = \{at\_Sy, v\_x \text{ for all } x\}$.
>
>| $i$ | at Sy | at Ad | at Br | at Pe | at Da | v Sy | v Ad | v Br | v Pe | v Da |
>|---|---|---|---|---|---|---|---|---|---|---|
>| 0 | 0 | ∞ | ∞ | ∞ | ∞ | 0 | ∞ | ∞ | ∞ | ∞ |
>| 1 | 0 | 1.5<sub>drSyAd</sub> | 1<sub>drSyBr</sub> | ∞ | ∞ | 0 | 1.5<sub>drSyAd</sub> | 1<sub>drSyBr</sub> | ∞ | ∞ |
>| 2 | 0 | 1.5 | 1 | 5<sub>drAdPe</sub> | 5.5<sub>drAdDa</sub> | 0 | 1.5 | 1 | 5<sub>drAdPe</sub> | 5.5<sub>drAdDa</sub> |
>| 3 | 0 | 1.5 | 1 | 5 | 5.5 | 0 | 1.5 | 1 | 5 | 5.5 |
>
>So $h^1(I) = \max(\dots) = 5.5$ (il fatto più costoso è $v\_Da$).
>
>>[!note]- Azioni per riga
>>- $i=1$: $at\_Ad = 1.5 + at\_Sy(0)$ via $drSyAd$; $at\_Br = 1 + at\_Sy(0)$ via $drSyBr$; $v\_Ad$ e $v\_Br$ con le stesse azioni
>>- $i=2$: $at\_Pe = 3.5 + at\_Ad(1.5) = 5$ via $drAdPe$; $at\_Da = 4 + at\_Ad(1.5) = 5.5$ via $drAdDa$; $v\_Pe$ e $v\_Da$ con le stesse azioni
>
>>[!tip] Capiamo
>>Con costi non unitari il valore di una cella dipende dalla somma dei costi, non dal numero della riga. Il critical path è $at\_Sy \to at\_Ad \to at\_Da$.

>[!example] $h^2$ in very simple "TSP" in Australia ($m=2$)
>- $P = \{at\_Sy, at\_Br, v\_Sy, v\_Br\}$; in breve $aS, aB, vS, vB$.
>- $I = \{aS, vS\}$; $G = \{aS, vS, vB\}$.
>- $A$ (entrambe costo $1$):
>	- $drSB$: pre $aS$; add $aB, vB$; del $aS$
>	- $drBS$: pre $aB$; add $aS, vS$; del $aB$
>
>| $i$ | aS | aB | vS | vB | aS,aB | aS,vS | aS,vB | aB,vS | aB,vB | vS,vB |
>|---|---|---|---|---|---|---|---|---|---|---|
>| 0 | 0 | ∞ | 0 | ∞ | ∞ | 0 | ∞ | ∞ | ∞ | ∞ |
>| 1 | 0 | 1<sub>drSB</sub> | 0 | 1<sub>drSB</sub> | ∞ | 0 | ∞ | 1<sub>drSB</sub> | 1<sub>drSB</sub> | 1<sub>drSB</sub> |
>| 2 | 0 | 1 | 0 | 1 | ∞ | 0 | 2<sub>drBS</sub> | 1 | 1 | 1 |
>| 3 | 0 | 1 | 0 | 1 | ∞ | 0 | 2 | 1 | 1 | 1 |
>
>So $h^2(I) = \max(\{aS,vS\}, \{aS,vB\}, \{vS,vB\}) = \max(0, 2, 1) = 2$, in contrast to $h^1(I) = \max(aS, vS, vB) = 1$.
>
>>[!note]- Azioni per riga
>>- $i=1$ (tutte via $drSB$, regredendo su $\{aS\}$ che vale $0$):
>>	- $aB = 1 + \{aS\}(0)$
>>	- $vB = 1 + \{aS\}(0)$
>>	- $\{aB,vS\} = 1 + \{aS,vS\}(0)$
>>	- $\{aB,vB\} = 1 + \{aS\}(0)$
>>	- $\{vS,vB\} = 1 + \{aS,vS\}(0)$
>>- $i=2$: $\{aS,vB\} = 1 + \{aB,vB\}(1)$ via $drBS$
>
>>[!tip] Capiamo
>>Alla riga 1, $aS$ e $vB$ sono raggiunti entrambi, ma **non insieme**.
>>- $regr(\{aS,vB\}, drSB) = \bot$, perché $drSB$ cancella $aS$.
>>- $regr(\{aS,vB\}, drBS) = \{aB,vB\}$, che alla riga 0 vale $\infty$.
>>
>>Solo alla riga 2, quando $\{aB,vB\}$ vale $1$, si ottiene $2$. $\{aS,aB\}$ resta invece $\infty$ per sempre, perché nessuna azione può regredirlo (ognuna cancella uno dei due fatti).

#### Fixed point formulation (for reference)



---
## FDR
All definitions, results, and proofs apply, exactly as stated, also to FDR planning tasks. (See the single exception below.)

We refer to pairs $(v,d)$ of variable and value as **facts**. We identify partial variable assignments with sets of facts.

>[!info] **Proposition** ($h^{m}$ is perfect in the limit)
>There exists $m$ s.t. $h^m = h^*$.
>
>>[!done] Proof
>>For an FDR task with variables $V$, $h^m = h^*$ for $m = |V|$. Given condition (iii) in the definition of $\mathit{regr}(g,a)$ for FDR (Chapter 7), every subgoal $g$ contains at most one fact for each variable $v \in V$. Thus with $m = |V|$ the case $|g| > m$ will never be used, so $h^m = r^*$.
>
>>[!warning]
>>In FDR, it suffices to set $m$ to the number of **variables**, as opposed to the number of variable values, i.e., STRIPS facts.
>
>>[!tip] Capiamo
>>In STRIPS un sottobiettivo può essere un insieme qualsiasi di fatti, quindi può avere fino a $|P|$ elementi e serve $m = |P|$. In FDR un sottobiettivo è un assegnamento parziale di variabili, quindi ogni variabile compare al più una volta (con un solo valore). Ci sono $|V|$ variabili, quindi $|g| \leq |V|$ sempre.
>>
>>La condizione (iii) di $\mathit{regr}$ per FDR serve proprio a questo: se il nuovo sottobiettivo $(g \setminus \mathit{eff}_a) \cup \mathit{pre}_a$ avrebbe due valori diversi per la stessa variabile, è autocontraddittorio e la regressione è $\bot$. Non possono quindi nascere sottobiettivi con più di $|V|$ fatti.
>>
>>Esempio: nel Logistics con $4$ posizioni del camion e $5$ del pacco, le variabili sono solo $2$ (posizione del camion e del pacco), mentre i fatti STRIPS sono $9$. Questo è il motivo per cui nel quiz $h^2(I) = h^*(I)$.