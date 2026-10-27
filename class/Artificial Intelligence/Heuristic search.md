---
Class: "[[Artificial Intelligence]]"
---
---
## Introduction
In the problem of finding a route from Saarbrucken to Bucharest, we can use various approaches. By using BFS would mean “look at all locations 10km distant from SB, look at all locations 20km distant from SB”, by using DFS “just keep choosing arbitrary roads, following through until you hit an ocean, then back up”.

Why don’t we try to use an algorithm that focuses on roads that go to the right direction? An algorithm of this kind is an **heuristic search**.

>[!tldr] Recall
>**Blind search** is a rigid procedure that uses the same expansion order no matter with problem it is applied to (no knowledge of the problem it is solving).

![[Pasted image 20261022193759.png|center|550]]

In the **heuristic search** the knowledge of expanding a state $s$ is given in the forma of a *heuristic function* $h(s)$, which estimates the cost of an optimal path from $s$ to the goal (search prefers to expand states $s$ with small $h(s)$). We could say that is a way of giving the computer knowledge about the problem it is solving, thereby reducing the number of stupid search steps.

>[!warning] Exhaustive search vs. sparse search
>- *exhaustive search* → systematically consider all possibilities; gives guarantees, but often difficult to find deep solutions (long plans)
>- *sparse search strategies* → disconsider some possibilities (at least initially); can go deep quickly, main categories are local search and Monte-Carlo tree search
>
>>[!info] Tree search vs. graph search
>>For the exhaustive search strategies, we consider graph search algorithms exclusively (i.e. we use duplicate elimination)
>>
>>There are also tree search versions of these algorithms. These are easier to understand, but aren’t used in practice (the search is memory-intensive anyway).

---
## Heuristic functions

>[!info] **Definition** (heuristic function)
>Let $\Theta=(S,A,c,T,I,S^{G})$ be a state space. A **heuristic function** for $\Theta$ is a function $h:S\to \mathbb{R}^{+}_{0}\cup \{\infty\}$ (we use $\infty$ for the cases where the goal states are not reachable from the current state).
>
>Its value $h(s)$ for a state $s$ is referred to as the state’s *heuristic value* or *$h$ value*. It’s an estimation of the cost of the path from $s\in S$ to $s'\in S^{G}$.

>[!info] **Definition** (remaning cost, $h^{*}$)
>Let $\Theta=(S,A,c,T,I,S^{G})$ be a state space. The **remaining cost** of $s \in S$ is the cost of an optimal plan for $s$, or $\infty$ if there exist no plan for $s$. The *perfect heuristic* for $\Theta$, written $h^{*}$, assigns every $s\in S$ its remaining cost as the heuristic value.
>
>It’s the exact cost of the path from $s\in S$ to $s'\in S^{G}$.

Heuristic functions $h$ estimate remaining cost $h^{*}$.

>[!note]
>The value of $h$ depends only on the state $s$, not on the search node (i.e., the path we took to reach $s$). I’ll sometimes abuse notation writing “$h(n)$” instead of “$h(n.State)$”.

### Properties

>[!info] **Definition** (Safe/Goal-Aware/Admissible/Consistent)
>Let $\Theta=(S,A,c,T,I,S^{G})$ be a state space and let $h$ be a heuristic for $\Theta$.
>
>The heuristic is called:
>- **safe** → if $\forall s\in S$, $h(s)=\infty$ implies $h^{*}(s)=\infty$
>- **goal-aware** → if $h(s)=0$ for all goal states $s\in S^{G}$
>- **admissible** → if $\forall s \in S,\,\,h(s)\leq h^{*}(s)$
>- **consistent** → if for all transitions $s\overset{a}{\to}s'$, $h(s)\leq h(s')+c(a)$
>
>>[!note] Clarification
>>The intuition for consistency if that no matter which action is performed, the difference between $h(s)$ (original state) and $h(s')$ can’t be more than the actual cost of the action $c(a)$ (upon taking a step with cost $c(a)$, its estimate does not decrease by more than $c(a)$).
>>
>>![[Pasted image 20261023165936.png|350]]

>[!info] **Proposition**
>1. If $h$ is admissible, then $h$ is goal-aware
>2. If $h$ is admissible, then $h$ is safe
>3. If $h$ is consistent and goal-aware, then $h$ is admissible
>
>>[!success] Proof
>>For the first two claim is obvious: if is admissible then for each $s$, $h(s)\leq h^{*}(s)$ then if $s\in S^{G}$ then $h^{*}(s)=0$ then $h(s)=0$. The same applies for safe.
>>
>>For the third proposition let $s\in S$ any state. We have two cases:
>>1. $h^{*}(s)=\infty$ → there is no plan for $s$, so $h(s)\leq \infty=h^{*}(s)$ is trivially verified
>>2. $h^{*}(s)<\infty$ → there is an optimal plan
>>
>>In order to prove point (2) we proceed by induction backwards $i=n,n-1,\dots,0$ on:
>>$$h(s_{i})\leq \sum^{n}_{j=i+1}c(a_{j})$$
>>(the estimated cost by $h$ from $s_{i}$ to goal state is less than the sum of the single actual costs)
>>
>>- base ($i=n$) → the sum is empty ($=0$), because $s_{n}$ is a goal state, for the goal-awareness $h(s_{n})=0\leq 0$
>>- step ($i\to i-1$) → assume the proposition for $i$; the transition $s_{i-1}\overset{a_{i}}{\to}s_{i}$ exists, so by consistency, $h(s_{i-1})\le h(s_i)+c(a_i)\le \sum^{n}_{j=i+1}c(a_{j})+c(a_{i}) =\sum_{j=i}^{n}c(a_j)$. 
>>
>>For $i=0$ this gives $h(s)\le h^*(s)$. $\blacksquare$

### The eternal trade-off
In principle, the “estimation of the remaining cost” is an arbitrary function, in practice we want it to be **accurate** (a constant function $h(x)=0$ is not informative), i.e., close to the actual remaining cost.

We also want it to be **fast** (i.e. a small overhead for computing $h$).

Because this two properties clash (to be accurate we can’t be fast, and to be fast we can’t be accurate), we need to *trade off the accuracy of $h$* against the overhead for computing $h(s)$ on every search state $s$.

To achieve this goal, given a problem $\Pi$, a heuristic function $h$ for $\Pi$ can be obtained as goal distance within a *relaxed* (simplified) problem $\Pi'$.

>[!example]- $\Pi$: find a route from Saarbrucken to Edinburgh
>![[Pasted image 20261022201451.png|400]]
>
>Relaxed problem $\Pi'$ throw away the map
>![[Pasted image 20261022201538.png|400]]
>
>Heuristic function $h$: straight line distance
>![[Pasted image 20261022201611.png|400]]
>

>[!example]- $\Pi$: solve the 15-puzzle (1)
>![[Pasted image 20261022201700.png|500]]
>
>Relaxed problem $\Pi'$: don’t distinguish tiles 8-15
>![[Pasted image 20261022201736.png|500]]
>
>Heuristic function $h$: length of plan for reduced puzzle
>![[Pasted image 20261022201813.png|500]]

>[!example]- $\Pi$: solve the 15-puzzle (2)
>Relaxed problem $\Pi'$: allow to move each tile to any cell in a single move, regardless of the situation
>
> Heuristic function $h$: number of misplaced tiles (here 13)

>[!example]- $\Pi$: solve the 15-puzzle (3. Manhattan distance) 
>Relaxed Problem $\Pi'$: allow to move each tile to any neighbor cell, regardless of the situation.
>Heuristic function $h$: Manhattan distance (here 36)
>
>The **Manhattan distance** between two points is the sum of the absolute differences of their coordinates, i.e. the distance travelled when moving only horizontally and vertically (as along the streets of a grid, without diagonal moves). For the 15-puzzle is the sum, over all tiles (excluding the blank), of the Manhattan distance between the tile’s current position and its goal position. It is the exact solution cost of the relaxed problem in which every tile can move independently to any adjacent cell, and it is therefore admissible.

>[!bug] Heuristic function pitfalls: path planning
>In the problem of path planning the situation is not as easy as in the previous examples.
>
>![[Pasted image 20261023125438.png]]
>
>In this picture we can observe $h^{*}$ (the actual remaining cost from each cell, taking obstacles into account). So that $h^{*}(I)=24$.
>
>Let’s now try to apply the Manhattan Distance to the previous example
>
>![[Pasted image 20261023125814.png]]
>
>In this case the topology is quite easy, so the Manhattan Distance is quite informative (few obstacles so the $h$ values are close to $h^{*}$ values). Even tho, if we reach $(3,8)$ with $h(2,8)=10$ and instead of trying to go down towards the goal, we go right the path is going to be much longer despite the numbers are getting smaller.
>
>![[Pasted image 20261023130256.png]]
>
>In this case, the topology seems more difficult because there are obstacles that forces you to take longer paths. The formula for $h$ is the same, but right now many cells have an $h$ value much lower than $h^{*}$ because the right path is another one.
>
>The Manhattan Distance ignores the obstacles, so its quality depends on the topology of the problem. With appropriately chosen walls, the research is drawn towards cells that seems closer to the goal but are dead eends.

