---
Class: "[[Artificial Intelligence]]"
---
---
## Introduction
Compared to blind and heuristic search, the sparse search renounces to the systematic of the firsts so as to be able to go deep quickly, by just using the guide information in a greedy way and limiting the exploration.

### Hill-climbing
This is the most simple strategy (also known as *gradient descent*). From a node, one chooses the child with the minimum $h$ value (random for tie-breaking), and never does backtracking until a goal state is reached (pure exploitation). Trivially this approach is neither complete or optimal.

>[!important] Pseudocode
>$$
>\begin{array}{l}
>\textbf{function } {\mathit{HillClimbing}}(\mathit{problem}) \\
>\quad n \leftarrow \text{a node } n \text{ with } n.\mathit{state} = \mathit{problem.InitialState} \\
>\quad \textbf{loop do} \\
>\qquad n' \leftarrow \text{among child nodes } n' \text{ of } n \text{ with minimal } h(n'), \\
>\qquad\qquad\quad \text{randomly pick one} \\
>\qquad \textbf{if } \mathit{problem.GoalTest}(n'.\mathit{State}) \textbf{ then return } \mathit{Plan}(n') \\
>\qquad n \leftarrow n'
>\end{array}
>$$

The problem of this approach is that the algorithm will get stuck in the local minima.

>[!example]
>$$
>\begin{align}
>&\textbf{Problem: } \text{Place queens so that they don't attack each other.} \\
>&\textbf{Heuristic: }\text{Number of pairs attacking each other.} \\
>&\textbf{Neighborhood: }\text{Move any queen within its column.}
>\end{align}
>$$
>
>This approach, starting from a random initialization, solves only $14\%$ of cases
>
>![[Pasted image 20261030122030.png|center|280]]
>
>To get $h=0$, we must move either of the 2 queens involved in the conflict. But every such move results in at least 2 new conflict, so this is a **local minimum**.
>
>The workaround to this problem is to use random walks to foster state-space exploration. Lets add it to the problem.
>$$\begin{align}
>\textbf{Random walks: }\;&\text{Allow up to 100 consecutive moves} \\
>&\text{in which the }h\text{ value does not get better.}
>\end{align}$$
>
>With this new approach, starting from a random initialization, it solves $94\%$ if cases

>[!info]
>Successful local search procedures combine exploitation of guidance information with randomized exploration to find new solution options

---
## MCTS (Monte-Carlo Tree Search)
Monte-Carlo Tree Search is one of the most important algorithms in the field of sparse search. Lets analyze how it works.

![[Pasted image 20261030123544.png|290]]

During each iteration proceeds as it follows:
1. **Tree policy** selects a leaf node (a node that has at least one non-expanded action) $n_{L}$ and traverses the current tree from root $n_{0}$ to leaf $n_{L}$
2. expand an arbitrary non-expanded action in $n_{L}$, yielding a new node $n'_{L}$ added to the tree
3. perform a sample run from $n'_{L}$, using a **rollout policy** up to a *terminal node* $n_{T}$
4. update *visit counts* $N(n,a)$ and *average action-rewards* $Q(n,a)$ on the path fro $n'_{L}$ to $n_{0}$, with the *reward* $r$ obtained in $n_{T}$

>[!example]-
>![[Pasted image 20261030125048.png]]
>tree policy selects the only leaf node available and performs an arbitrary action (expand center node)
>
>![[Pasted image 20261030125057.png]]
>rollout policy on the central node
>
>![[Pasted image 20261030125102.png]]
>update central node and root with the reward and increment visit count
>
>![[Pasted image 20261030125108.png]]
>root node still has non-expanded actions. left action is selected arbitrary and rollout policy is performed (to go down from a node that node must be complete, the central node cannot be selected yet)
>
>![[Pasted image 20261030125115.png]]
>all involved nodes are updated with reward and visit count
>
>![[Pasted image 20261030125123.png]]

>[!important] Pseudocode
>$$
>\begin{array}{l}
>\textbf{function } {\color{yellow}\mathit{MCTS}}(\mathit{problem}, s) \textbf{ returns } {\color{red}\text{an action}} \\
>\quad n_0 \leftarrow \text{node with } n_0.\mathit{State} = s \\
>\quad \mathit{MCTStree} \leftarrow \text{tree containing only } n_0 \\
>\quad \textbf{for each } a \in \mathit{problem.Actions}(n_0.\mathit{State}) \textbf{ do} \\
>\qquad Q(n_0, a) \leftarrow 0,\ N(n_0, a) \leftarrow 0 \\
>\quad \textbf{while } \text{time limit not reached } \textbf{do} \\
>\qquad n_L \leftarrow \mathit{TreePolicy}(\mathit{problem}, \mathit{MCTStree}, n_0) \\
>\qquad \textbf{if } \text{not } \mathit{problem.isTerminal}(n_L.\mathit{State}) \textbf{ then} \\
>\qquad\quad n'_L \leftarrow \mathit{ExpandOne}(\mathit{problem}, \mathit{MCTStree}, n_L) \\
>\qquad\quad n_T \leftarrow \mathit{RollOut}(\mathit{problem}, n'_L) \\
>\qquad \textbf{else} \\
>\qquad\quad n'_L \leftarrow \text{parent of } n_L \text{ in } \mathit{MCTStree},\ n_T \leftarrow n_L \\
>\qquad \mathit{UpdatePath}(\mathit{MCTStree}, n'_L, n_T) \\
>\quad \textbf{return } \underset{a \in \mathit{problem.Actions}(n_0.\mathit{State})}{\arg\max}\ Q(n_0, a)
>\end{array}
>$$
>
>where:
>- $Q(n,a)$ is the average action-rewards
>- $N(n,a)$ is the visit counts
>- $UpdatePath$ updates based on $n_{T}$ reward, backwards from $n'_{L}$ to $n_{0}$.
>
>$$
>\begin{array}{l}
>\textbf{function } {\color{yellow}\mathit{TreePolicy}}(\mathit{problem}, \mathit{MCTStree}, n) \textbf{ returns } \text{a node} \\
>\quad \textbf{loop do} \\
>\qquad \textbf{if } \mathit{problem.isTerminal}(n.\mathit{State}) \textbf{ then return } n \\
>\qquad \textbf{if } \text{ex. } a \in \mathit{problem.Actions}(n.\mathit{State}) \\
>\qquad\qquad \text{s.t. not ex. } n' \text{ with } (n, a, n') \in \mathit{MCTStree} \textbf{ then return } n \\
>\qquad a \leftarrow \mathit{Select}(\mathit{MCTStree}, n) \quad {\color{red}\text{// balance exploration and exploitation, see later}} \\
>\qquad n \leftarrow n' \text{ where } (n, a, n') \in \mathit{MCTStree} \\
>\\
>\textbf{function } {\color{yellow}\mathit{ExpandOne}}(\mathit{problem}, \mathit{MCTStree}, n) \textbf{ returns } \text{a node} \\
>\quad \text{arbitrarily choose } a \in \mathit{problem.Actions}(n.\mathit{State}) \\
>\qquad \text{s.t. not ex. } n' \text{ with } (n, a, n') \in \mathit{MCTStree} \\
>\quad n' \leftarrow \mathit{ChildNode}(\mathit{problem}, n, a) \\
>\quad \text{add } (n, a, n') \text{ to } \mathit{MCTStree} \\
>\quad \textbf{for each } \mathit{action}\ a' \in \mathit{problem.Actions}(n'.\mathit{State}) \textbf{ do} \\
>\qquad Q(n', a') \leftarrow 0 \\
>\qquad N(n', a') \leftarrow 0 \\
>\quad \textbf{return } n' \\
> \\
>\textbf{function } {\color{yellow}\mathit{RollOut}}(\mathit{problem}, n) \textbf{ returns } \text{a node} \\
>\quad d \leftarrow 0 \\
>\quad \textbf{loop do} \\
>\qquad \textbf{if } \mathit{problem.IsTerminal}(n.\mathit{State}) \textbf{ then return } n \\
>\qquad \textbf{if } d = \mathit{problem.Horizon}() \textbf{ then return } n \quad {\color{red}\text{// optional}} \\
>\qquad \text{uniformly sample } a \in \mathit{problem.Actions}(n.\mathit{State}) \\
>\qquad n \leftarrow \mathit{ChildNode}(\mathit{problem}, n, a) \\
>\qquad d \leftarrow d + 1 \\
>\\
>\textbf{function } {\color{yellow}\mathit{UpdatePath}}(\mathit{MCTStree}, n, n_T) \\
>\quad \textbf{while } n \text{ is not NULL } \textbf{do} \\
>\qquad a \leftarrow \text{start of path from } n \text{ to } n_T \text{ in } \mathit{MCTStree} \\
>\qquad Q(n, a) \leftarrow Q(n, a) * N(n, a) \quad {\color{red}\text{// recover summed-up reward}} \\
>\qquad Q(n, a) \leftarrow Q(n, a) + \mathit{problem.Reward}(n, \ldots, n_T) \\
>\qquad N(n, a) \leftarrow N(n, a) + 1 \\
>\qquad Q(n, a) \leftarrow Q(n, a) / N(n, a) \quad {\color{red}\text{// new average}} \\
>\qquad n \leftarrow \text{parent of } n \text{ in } \mathit{MCTStree}
>\end{array}
>$$
>
>where:
>- $IsTerminal()$ → define for application/context; e.g. $s$ is a goal state or $A(s)=\varnothing$ (MCTS works best if rollouts are always short, i.e., every path ends a terminal state after small number of steps; intrinsic in most games, e.g. Go, Chess, but not in general)
>- $Horizon()$ → can be used if sensible (e.g. classical planning); may be used to limit rollout policy, to avoid loops, or if the state space if infinite, in general is used to avoid spending too much time doing the rollout
>- $Reward(n,\dots,n_{T})$ → rewards for path $n,\dots,n_{T}$ (in general how good a path is depends not only on the terminal node, not useful if we just need a satisficing plan, e.g. games)
>	- $Reward(n,\dots,n_{T})$ is *satisficing* planning → if $n_{T}$ goal then $1$, else $0$
>	- $Reward(n,\dots,n_{T})$ is *cost-optimal* planning → if $n_{T}$ goal then $-1 \cdot \sum_{a \in \text{path}(n, \ldots, n_T)} c(a)$; else high negative number (e.g. $-1\cdot problem.Horizon() \cdot \text{maximal action cost}$)
>	- for games it translates into $n_{T}$ win $1$, $n_{T}$ draw $0$, $n_{T}$ loss $-1$
>

>[!note] Capiamo
>![[bangdesk.gif]]
>treepolicy viene chiamato su radice → verifica se il nodo ha delle azioni non espanse (se si ritorna così che expandone possa espanderli) → altrimenti select seleziona uno tra i figli del nodo analizzati → viene aggiornato il nodo attualmente preso in considerazione → ricomincia il loop, quindi viene verificato se effettivamente ci sta una azione da aprire per il nuovo nodo etc.

>[!info] Capiamo pt2
>![[bangdesk.gif]]
>Nel contesto di un gioco (e.g. checkers) una volta che io faccio una mossa viene eseguito MSCT (supponiamo come time limit 100 esecuzioni del while) e viene scelta come mossa successiva quella con average reward maggiore. Inoltre per come funziona l’argmax viene scelta come successiva mossa quella con reward maggiore tra quelle legali dalla radice. (Però nel giochi la Select è diversa)
>
>https://vgarciasc.github.io/mcts-viz/
### Properties
- *only requires simulator functionality* (expanding actions, generating outcome states) → can be effective given just this while heuristic search requires $h$ to be effective, which must be designed in the first place (MCTS can be effective even without any guidance information)
- *complete and optimal* → in the sense of returning an action starting a reward-maximizing plan given sufficient time and memory, provided $Select$ is designed so as to guarantee convergence of $Q(n,a)$-values to the actual value of $a$

$Select$ must have “sufficient exploration” → all actions need to be tried sufficiently often to find and exploit the maximal rewards (“explore-only“ would mean select the nodes with lowest $N(n,a)$)
$Select$ must have “sufficient exploitation” → there must be a bias to maximize rewards, strong enough for the average rewards to converge to the optimal rewards in the limit (“exploit-only” algorithm would select only based on the highest $Q(n,a)$, independent from $N(n,a)$ which would mean train a path that has higher lower rewards but is been visited much less times)

However the completeness and optimality are mainly theoretical. Typically MCTS runs out of time long before convergence, but convergence implies **anytime behavior** (MCTS can select good actions quickly, and guarantees to improve if more time is available).

---
## Search control (exploitation vs. exploitation) in MCTS

>[!question] How to take action decisions in $Select$? So as to guarantee convergence and obtain good practical performance?
>There is a lot of literature on this, but **UCT** is still the canonical choice

Other than the $Select$, there are many other possibilities to have search guidance information:
- $RollOut$ → use approximate (e.g. learned) rollout policy instead uniform random action choice
- $Select$ → bias by approximate (e.g. learned) tree policy
- state evaluation → integrate (e.g. learned) heuristic function

The search guidance can eve be learned as part of MCTS, by combining MCTS with RL.

> [!info] Multi-armed bandit
> **Problem:** a slot machine ("bandit") has several levers (*arms*). Each arm pays out a reward from a different distribution, unknown to the agent. With a limited number of pulls, which arms should you pull?
>
> **The dilemma:**
> - *Exploitation*: pull the arm that has paid the most so far, to collect rewards already known.
> - *Exploration*: also try other arms, because the current estimate may be wrong and a rarely tried arm could be better.
>   
>**UCB** (*Upper Confidence Bounds*): a formula that resolves the dilemma by giving each arm a score, namely the observed average plus a bonus that grows for rarely tried arms
>


### UCT algorithm
The UCT algorithm uses the **UCB formula**:
$$\text{SelVal}(n,a) = Q(n,a) + c\sqrt{\frac{2\ln N(n)}{N(n,a)}}$$

where $c$ is an algorithm parameter used to compensate the rewards and the visits (e.g. is usually the rewards are in the order of millions, $c$ will be big to compensate $N(n,a)$) and $N(n)$ is the number of times $n$ was visited by MCTS so far.

So the **UCT algorithm**, is basically a MCTS with UCB plugged into:
$$
\begin{array}{l}
\textbf{function } {\color{yellow}\mathit{Select}}(\mathit{MCTStree}, n) \textbf{ returns } \text{an action} \\
\quad \textbf{return } \underset{a:(n,a,n') \in \mathit{MCTStree}}{\arg\max}\ \mathit{SelVal}(\mathit{MCTStree}, n, a)
\end{array}
$$

>[!info] Notes regarding $c$
>- affects exploration-exploitation trade-off
>- should scale linearly in rewards (mediate lhs vs. rhs of UCB)
>- simple for rewards in $[0,1]$ (e.g. satisficing planning, games)
>- a general method is $c:=max_{a}Q(n,a)$

### Action policies

>[!info] Notation
>Let $\Theta=(S,A,c,T,I,S^{G})$ be a state space. For $s\in S$, $A(s)\subseteq A$ is the set of actions applicable in $s$. Denote by $\mathcal{D}(A)$ the set of probability distributions over $A$.

>[!info] **Definition** (Action policy)
>An **action policy** is a function $\pi:S\to A$ where $\pi(s)\in A(s)$ for all $s\in S$
>
>In probabilistic planning (MDPs) and in games, “plans” take the form of policies, because we need to specify a reaction to all possible events (action outcomes/opponent moves). Here we consider policies as a form of search guidance for MCTS

>[!info] **Definition** (Stocastic action policy)
>A **stochastic action policy** is a function $\hat{\pi}:S\to \mathcal{D}(A)$, where $\sum_{a\in A(S)}\hat{\pi}(s,a)=1$ for $s\in S$ (denoting $\hat{\pi}(s)(a)$ as $\hat{\pi}(s,a)$)
>
>Stochastic policies capture (for our purposes here) imperfect search-guidance knowledge. In what follows, we assume stochastic policies $\hat{\pi}_{roll}$ and $\hat{\pi}_{tree}$ (obtained e.g. by learning, prior to running MCTS)

Having this now we can modify the $RollOut$ by simply plugging-in $\hat{\pi}_{roll}$:
$$
\begin{array}{l}
\textbf{function } {\color{yellow}\mathit{RollOut}}(\mathit{problem}, n) \textbf{ returns } \text{a node} \\
\quad \textbf{loop do} \\
\qquad \ldots \\
\qquad \text{sample } a \in \mathit{problem.Actions}(n.\mathit{State}) \ {\color{red}\text{according to } \hat{\pi}_{\text{roll}}(n.\mathit{State})} \\
\qquad \ldots
\end{array}
$$
Act intelligently in rollouts, instead of uniform randomly (much better information about true value of leaf node).

We can also modify the $Select$ formula using $\hat{\pi}_{tree}$ to bias exploration
$$
\text{SelVal}(n,a) = Q(n,a) + c \frac{\hat{\pi}_{tree}(n.\text{State},a)}{N(n,a)}
$$

Prefers to explore actions that appear promising according to $\hat{\pi}_{tree}$ and enhances $Q(n, a)$ knowledge with knowledge captured by $\hat{\pi}_{tree}$

### Informing $Q$-values and updates with heuristic functions
Plain MCTS knows nothing about the problem: every $Q(n,a)$ starts at zero, and the only information it ever gets comes from random rollouts. In cost-optimal classical planning we usually *do* have extra knowledge, namely a heuristic $h$ that estimates the remaining cost to the goal. This slide shows two places where MCTS can use it.
#### Q-values: start from a guess, not from zero
By default every action starts with $Q(n,a) := 0$. With a heuristic we can start from something much more meaningful:
$$Q(n,a) := -c(a) - h(n')$$
where $n'$ is the successor node reached by applying $a$. The reasoning is simple. $Rewards$ are negative costs, so the value of an action is minus the total cost of the plan it starts. That cost is roughly the price of the action itself, $c(a)$, plus what is still missing from $n'$ to the goal, which is exactly what $h(n')$ estimates. So each action begins with a plausible value instead of an uninformative 0, and the averages need fewer simulations to settle (**faster convergence**).
#### Updates: don't trust a single rollout alone
Normally the update after an iteration uses only the reward $r$ of the terminal node $n_T$ reached by the rollout. One random rollout is a very noisy sample of how good the node really is. It’s useful to mix in the heuristic:
$$\frac{r + \left(-h(n'_L)\right)}{2}$$

where $n'_L$ is the node just added by $ExpandOne$. This is the average of two estimates of the same thing: the rollout's outcome and the heuristic's opinion of the new node. The rollout is concrete but random, while the heuristic is stable but approximate, and combining them enriches the rollout with remaining-cost information.

So the pseudocode would look like this:
$$
\begin{array}{l}
\textbf{function } {\color{yellow}\mathit{UpdatePath}}(\mathit{MCTStree}, n, n_T) \\
\quad \textbf{while } n \text{ is not NULL } \textbf{do} \\
\qquad a \leftarrow \text{start of path from } n \text{ to } n_T \text{ in } \mathit{MCTStree} \\
\qquad Q(n, a) \leftarrow Q(n, a) * N(n, a) \quad {\color{red}\text{// recover summed-up reward}} \\
\qquad Q(n, a) \leftarrow Q(n, a) + \dfrac{\mathit{problem.Reward}(n, \ldots, n_T) + \left(-h(n'_L)\right)}{2} \\
\qquad N(n, a) \leftarrow N(n, a) + 1 \\
\qquad Q(n, a) \leftarrow Q(n, a) / N(n, a) \quad {\color{red}\text{// new average}} \\
\qquad n \leftarrow \text{parent of } n \text{ in } \mathit{MCTStree}
\end{array}
$$

>[!warning] The extreme case: no rollouts at all
>If the heuristic is trusted enough, the rollout can be dropped entirely and the update value is just $-h(n'_L)$. The search tree is then built and evaluated with the heuristic alone. This is the idea used in Keller and Helmert (2013) and in the Go-playing systems of Silver et al. (2016, 2017), where a learned evaluation replaces random simulation.

> [!example] Worked example: MCTS on simplified path planning (slide 27)
> **Setup**
>
> |   | 1 | 2 | 3 |
> |---|---|---|---|
> | A | I (start) | ■ | |
> | B | | | |
> | C | | ■ | G (goal) |
>
> - Actions: only S (south) and E (east). ■ = blocked cell.
> - Applicable actions: A1: S · B1: S, E · B2: E · B3: S · C1 and C3: none (terminal).
> - Satisficing planning: reward $r = 1$ if the rollout ends in the goal, else $0$.
> - ExpandOne: first S if applicable, then E.
> - RollOut with $\hat{\pi}_{\text{roll}}$: S if applicable, else E.
> - Select: $Q(n,a) + c\,\frac{\hat{\pi}_{\text{tree}}(n.\text{State},a)}{N(n,a)}$ with $c = 1$, $\hat{\pi}_{\text{tree}}$: S ↦ 0.75, E ↦ 0.25.
>
> **Iterations** (updates written as N, Q)
>
> | It. | TreePolicy → $n_L$ | ExpandOne | RollOut | $r$ | Updates |
> |---|---|---|---|---|---|
> | I | A1 has a free action → stop at A1 | S → B1 | B1 →S C1 (terminal) | 0 | B1(S): 1, 0 · A1(S): 1, 0 |
> | II | A1 complete → Select S → B1; B1 has free actions → stop | S → C1 | C1 is already terminal | 0 | B1(S): 2, 0 · A1(S): 2, 0 |
> | III | A1 → B1; E still free → stop at B1 | E → B2 | B2 →E B3 →S C3 (goal) | 1 | B2(E): 1, 1 · B1(E): 1, 1 · A1(S): 3, 1/3 |
> | IV | A1 → B1 → Select E → B2; E free → stop | E → B3 | B3 →S C3 (goal) | 1 | B3(S): 1, 1 · B2(E): 2, 1 · B1(E): 2, 1 · A1(S): 4, 1/2 |
> | V | A1 → B1 → Select E → B2 → B3; S free → stop | S → C3 | C3 is already the goal | 1 | B3(S): 2, 1 · B2(E): 3, 1 · B1(E): 3, 1 · A1(S): 5, 3/5 |
> | VI | … → B3 → C3: terminal → $n_L$ = C3 | none (else branch) | none: $n'_L$ = B3, $n_T$ = C3 | 1 | B3(S): 3, 1 · B2(E): 4, 1 · B1(E): 4, 1 · A1(S): 6, 2/3 |
>
> Terminal nodes (C1, C3) have no applicable actions, so they get no N/Q table: updates effectively start from their parent.
>
> **Select at B1** (A1, B2, B3 have only one action, so Select is trivial there)
> - IV: SelVal(B1,S) = $0 + \frac{0.75}{2} = 0.375$ · SelVal(B1,E) = $1 + \frac{0.25}{1} = 1.25$ → **E**
> - V: S = $0.375$ · E = $1 + \frac{0.25}{2} = 1.125$ → **E**
> - VI: S = $0.375$ · E = $1 + \frac{0.25}{3} \approx 1.083$ → **E**
>
> $\hat{\pi}_{\text{tree}}$ prefers S, but $Q(B1,S) = 0$ because S leads to the dead end C1: the observed rewards win over the prior. With this formula the bonus of S never grows (no $\ln N(n)$ term), so S is not chosen again.
>
> **Tree after iteration VI** (edge labels = iterations in which the edge was part of the tree path)
>
> ```mermaid
> flowchart TD
>   A1["A1 (start)<br/>S: N=6, Q=2/3"]
>   B1["B1<br/>S: N=2, Q=0<br/>E: N=4, Q=1"]
>   C1["C1<br/>terminal, no goal"]
>   B2["B2<br/>E: N=4, Q=1"]
>   B3["B3<br/>S: N=3, Q=1"]
>   C3["C3<br/>goal"]
>   A1 -->|"S · I–VI"| B1
>   B1 -->|"S · II"| C1
>   B1 -->|"E · III–VI"| B2
>   B2 -->|"E · IV–VI"| B3
>   B3 -->|"S · V–VI"| C3
> ```
>
> The search has concentrated on the plan **S, E, E, S**, which reaches the goal.
> 
> ![[Pasted image 20261030200156.png]]

---
## MCTS hyperparameters
**Hyperparameters** are choices made by the algorithm designer before execution (i.e., values ​​that the algorithm neither calculates nor modifies on its own).

In MCTS, $Q(n,a)$ and $N(n,a)$ are values ​​that the algorithm computes during the search process. The constant $c$ in the UCT formula, however, is set by you; it is a hyperparameter.

The hyperparameters in MCTS are:
- **terminal states, rollout horizon, reward function** → might be naturally part of the problem; but are often algorithm parameters chosen by designer for efficacy
- **guidance** → many options; even when using UCT only, the $Select()$ function has the parameter $c$
- **updates** → also various options (not discussed here); in particular, Bellman updates are much stronger when applicable
- **learning** → again many options

MCTS really is an algorithm schema, not an algorithm. All successful MCTS applications engineer the algorithm to their specific needs.

