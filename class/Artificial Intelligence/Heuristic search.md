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
>- **safe** → if $\forall s\in S$, $h(s)=\infty$ implies $h^{*}(s)=\infty$ (it’s possible to safely prune useless states).
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
>>[!done] Proof
>>We need to show that $h(s)\leq h^{*}(s)$ for all $s$. For states $s$ where $h^{*}(s)=\infty$, this is trivial. For all other states, we show the claim by induction over the length of the cheapest path to a goal state.
>>
>>- base case ($s$ is a goal state) → $h(s)=0$ by goal awareness, so $h(s)\leq h^{*}=0$ as desired
>>- step case → assume the claim holds for all states $s'$ with a cheapest goal path of length $n$. Say $s$ has a cheapest goal path of length $n+1$, the first transition of which is $\overset{a}{s\to s'}$. By consistency, we have *(a)* $h(s)\leq h(s')+c(a)$. By construction, $s'$ has a cheapest goal path of length $n$ and thus, by induction hypothesis, *(b)* $h(s')\leq h^{*}(s')$. As $\overset{a}{s\to s'}$ starts a cheapest goal path, we furthermore have *(c)* $h^{*}(s)=h^{*}(s')+c(a)$. Inserting (b) into (a), we get $h(s)\leq h^{*}(s')+c(a)$. Inserting (c) into the latter, we get $h(s)\leq h^{*}(s)$ as desired. $\blacksquare$

>[!example]
>>[!question] Is $h=\text{straight line distance}$ safe/goal-aware/admissible/consistent?
>>All yes. Easy for goal-aware. Consistency: if you drive $100$km, then straight line distance can’t decrease by more than $100$km. Rest follows.
>
>>[!question] Is $h=\text{Manhattan distance in the 15-puzzle}$ safe/goal-aware/admissible/consistent?
>>All yes. Easy for goal-aware. Consistency: moving a tile can’t decrease $h$ by more than $1$. Rest follows.
>
>In practice, most heuristics are safe and goal-aware, and admissible heuristics are typically consistent.

#### Additional properties

>[!info] **Definition** (Domination)
>Let $\Theta=(S,A,c,T,I,S^{G})$ be a state space, and let $h$ and $h'$ be admissible heuristics for $\Theta$. We say that $h'$ **dominates** $h$ if $h\leq h'$ (i.e., $\forall s\in S$, $h(s)\leq h'(s)$).
>
>$h^{*}$ dominates every other admissible heuristic.
>
>>[!note] $h'$ dominates $h$ = “$h'$ provides a lower bound at least as good as $h$”
>
>>[!example] 15-Puzzle
>>- $h'$ = Manhattan Distance
>>- $h$ = misplaced tiles
>>
>>Each misplaced tiles accounts for at lest $1$ (typically more) in $h'$.

>[!info] **Definition** (Additivity)
>Let $\Theta=(S,A,c,T,I,S^{G})$ be a state space, and let $h_{1},\dots,h_{n}$ are **additive** if $h_{1}+\dots+h_{n}$ is admissible (i.e., $\forall s\in S$ we have $h_{1}(s)+\dots+h_{n}(s)\leq h^{*}(s)$).
>
>>[!note] An ensemble of heuristics is additive if its sum is admissible.
>
>>[!example] 15-Puzzle
>>$h_{1}$ considers only tiles $1\dots_{7}$, and $h_{2}$ considers only tiles $8\dots 15$. The two estimates are then “independent” (so additive).

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

---
## Greedy Best-First Search (GBFS)

>[!important] Pseudocode
>$$
>\begin{array}{l}
>\textbf{function } \textit{GreedyBestFirstSearch}(\textit{problem}) \textbf{ returns } \text{a plan, or unsolvable} \\
>\quad \textit{node} \leftarrow \text{a node } n \text{ with } n.\textit{state} = \textit{problem.InitialState} \\
>\quad \textit{open} \leftarrow \text{a priority queue ordered by } \text{ascending } h\text{, only element } n \\
>\quad \textit{closed} \leftarrow \text{empty set of states} \\
>\quad \textbf{loop do} \\
>\qquad \textbf{if } \textit{Empty?(open)} \textbf{ then return } \text{unsolvable} \\
>\qquad n \leftarrow \textit{Pop(open)} \\
>\qquad \textbf{if } \textit{problem.GoalTest}(n.\textit{State}) \textbf{ then return } \textit{Plan}(n) \\
>\qquad \textit{closed} \leftarrow \textit{closed} \cup n.\text{State} \\
>\qquad \textbf{for each } \textit{action } a \textbf{ in } \textit{problem.Actions}(n.\textit{State}) \textbf{ do} \\
>\qquad\quad n' \leftarrow \textit{ChildNode}(\textit{problem}, n, a) \\
>\qquad\quad \textbf{if } n'.\text{State} \notin \textit{closed} \cup \text{States}(\textit{open}) \textbf{ then } \textit{Insert}(n', h(n'), \textit{open})
>\end{array}
>$$

In this case we have an open list ordered by ascending $h$, and the duplicates are checked at successor generation, against both the open list and the closed list.

Furthermore by construction in this algorithm, once a **goal state is added** to the open list it gets **immediately returned** because it’s the one with lower $h$ value ($=0$ if goal-aware), and if a state is already in `open` and is reached again with a lower $h$ value, the new node is discarded instead of replacing the old one, even though `open` is ordered by ascending $h$ and that state should gain priority.

>[!example] Route from Arad to Bucharest
>![[Pasted image 20261028152912.png]]
>
>In this map there are represented the cities with the actual distances between them. The heuristic in the following picture is the straight line distance between the city and Bucharest.
>
>![[Pasted image 20261028153809.png]]
>
>So the found path is:
>$$\text{Arad}\to \text{Sibiu}\to \text{Fagaras} \to \text{Bucharest}$$
>even if the optimal plan would be:
>$$\text{Arad}\to \text{Sibiu}\to \text{Rimnicu Vilcea}\to \text{Pitesti}\to \text{Bucharest}$$

>[!info] Guarantees
>- **completeness** → yes, thanks to duplicate elimination and our assumption that the state space is finite.
>- **optimality** → no ($h$ might lead us to Bucharest via Paris)
>
>>[!question] Would it be optimal if $h=h^{*}$?
>>GBFS ranks nodes by remaining cost only and ignores g (the cost already paid). A state can have a very low h\* but be reached through a very expensive action.
>>
>>Example (clean the pavement, the car is too expensive to pay off):
>>
>>| Plan | Buy | Clean | Total |
>>|---|---|---|---|
>>| Broom | 5 | 10 | **15** (optimal) |
>>| Sweeper car | 100 | 1 | 101 |
>>
>>- $h^*(\text{after broom}) = 10$, $h^*(\text{after car}) = 1$
>>- `open` is ordered by ascending $h$, so GBFS picks the car state and returns a plan of cost 101 instead of 15.

---
## A*

>[!important]
>$$
>\begin{array}{l}
>\textbf{function } \text{A}^*(\textit{problem}) \textbf{ returns } \text{a plan, or unsolvable} \\
>\quad \textit{node} \leftarrow \text{a node } n \text{ with } n.\textit{State} = \textit{problem.InitialState} \\
>\quad \textit{open} \leftarrow \text{a priority queue ordered by } \textcolor{red}{\text{ascending } g + h}\text{, only element } n \\
>\quad \textit{closed} \leftarrow \text{empty set of states} \\
>\quad \textbf{loop do} \\
>\qquad \textbf{if } \textit{Empty?(open)} \textbf{ then return } \text{unsolvable} \\
>\qquad n \leftarrow \textit{Pop(open)} \\
>\qquad \textbf{if } \textit{problem.GoalTest}(n.\textit{State}) \textbf{ then return } \textit{Plan}(n) \\
>\qquad \textit{closed} \leftarrow \textit{closed} \cup n.\text{State} \\
>\qquad \textbf{for each } \textit{action } a \textbf{ in } \textit{problem.Actions}(n.\textit{State}) \textbf{ do} \\
>\qquad\quad n' \leftarrow \textit{ChildNode}(\textit{problem}, n, a) \\
>\qquad\quad \textbf{if } \textcolor{red}{n'.\text{State} \notin \textit{closed} \cup \text{States}(\textit{open})} \textbf{ then} \\
>\qquad\qquad \textit{Insert}(n', \textcolor{red}{g(n') + h(n')}, \textit{open}) \\
>\qquad\quad \textbf{else if } \text{ex. } \textcolor{red}{n'' \in \textit{open} \text{ s.t. } n''.\text{State} = n'.\text{State} \text{ and } g(n') < g(n'')} \textbf{ then} \\
>\qquad\qquad \textcolor{red}{\text{replace } n'' \text{ in } \textit{open} \text{ with } n'}
>\end{array}
>$$

In this case we have an open list ordered by ascending $g+h$, and the duplicates are handled exactly as in uniform-cost search (nodes are replaced if a new cheaper path is found).

>[!example] Route from Arad to Bucharest
>![[Pasted image 20261028152912.png]]
>
>In this map there are represented the cities with the actual distances between them. The heuristic in the following picture is the straight line distance between the city and Bucharest. Compared to the example for GBFS we also count $g$
>
>![[Pasted image 20261028155946.png]]
>
>So the found path is:
>$$\text{Arad}\to \text{Sibiu}\to \text{Rimnicu}\to \text{Pitesti}\to \text{Bucharest}$$
>which is the optimal plan.

>[!into] **Theorem**
>$A^{*}$ is optimal
>
>>[!done] Proof
>>The idea to prove that $A^{*}$ is optimal is via a correspondence (reduction) to uniform-cost search.
>>
>>**Step 1 → Capture the heuristic function in terms of action costs**
>>
>>>[!info] **Definition** ($h$-weighted state space)
>>>Let $\Theta=(S,A,c,T,I,S^{G})$ be a state space, and let $h$ be a consistent and goal-aware heuristic function for $\Theta$. We define **$h$-weighted state space** as $\Theta^{h}=(S,A^{h},c^{h},T^{h},I,S^{G})$ where:
>>>- $A^{h}:=\{a[s,s']\mid a\in A,s\in S,s'\in S, (s,a,s')\in T\}$
>>>- $c^{h}:A^{h}\to \mathbb{R}^{+}_{0}$ is defined by $c^{h}(a[s,s']):=c(a)-[h(s)-h(s')]$
>>>- $T^{h}=\{(s,a[s,s'],s')\mid(s,a,s')\in T\}$

