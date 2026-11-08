---
Class: "[[Artificial Intelligence]]"
Sem: WiSe 26/27
---
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
>The idea is that $a$ can make the conjunctive subgoal $g$ true if (1) it achieves part of $g$; (2) it contradicts none of $g$; and (3) the new subgoal we would have to solve is not self-contradictory.

>[!info] **Definition** (FDR regression)
>Let $(V,A,c,I,G)$ be an FDR planning task, $g$ be a partial variable assignment, and $a\in A$. We say that $g$ is **regressable** by $a$ if:
>1. $e\hspace{-0.1em}f\hspace{-0.1em}f_{a}\cap g\neq \varnothing$
>2. there is no $v\in V$ s.t. $v\in Vars[e\hspace{-0.1em}f\hspace{-0.1em}f_{a}]\cup Vars[g]$ and $e\hspace{-0.1em}f\hspace{-0.1em}f_{a}(v)\neq g(v)$
>