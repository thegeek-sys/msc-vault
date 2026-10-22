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

>[!info] Definition (heuristic function)
>Let $\Theta=(S,A,c,T,I,S^{G})$ be a state space. A **heuristic function** for $\Theta$ is a function $h:S\to \mathbb{R}^{+}_{0}\cup \{\infty\}$ (we use $\infty$ for the cases where the goal states are not reachable from the current state).
>
>Its value $h(s)$ for a state $s$ is referred to as the state’s *heuristic value* or *$h$ value*.

>[!info] Definition (remaning cost, $h^{*}$)
>Let $\Theta=(S,A,c,T,I,S^{G})$ be a state space. The **remaining cost** of $s \in S$ is the cost of an optimal plan for $s$, or $\infty$ if there exist no plan for $s$. The *perfect heuristic* for $\Theta$, written $h^{*}$, assigns every $s\in S$ its remaining cost as the heuristic value.

Heuristic functions $h$ estimate remaining cost $h^{*}$.

>[!note]
>The value of $h$ depends only on the state $s$, not on the search node (i.e., the path we took to reach $s$). I’ll sometimes abuse notation writing “$h(n)$” instead of “$h(n.State)$”.

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

