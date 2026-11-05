---
Class: "[[Artificial Intelligence]]"
Sem: WiSe 26/27
---
---
## Introduction

>[!info] Classical planning
>- **deterministic** vs. stochastic
>- **single-agent** vs. multi-agent

The symbolic specification of $\Pi$ comes in a problem description language

>[!info] Symbolic specification of a problem
>Here’s one way to do it:
>- **$P$** → set of Boolean variables (*prepositions*)
>- **$I\subseteq P$** → indicate which proposition are true in the initial state
>- **$G\subseteq P$** → $s$ is a goal state iff $G\subseteq s$
>- **$A$** → set of actions $a$, each which *precondition $pre_{a}$*, *add list $add_{a}$*, and *delete list $del_{a}$*; $a$ applicable to $s$ iff $pre_{a}\subseteq s$, outcome state is $(s \cup add_{a}\backslash del_{a})$
>- **$c$** → maps each $a\in A$ to its cost $c(a)$

This language is called “STRIPS” and specifying the problem means writing STRIPS. It is strictly more powerful than simulators as it allows to implement the simulator API but also to analyze the problem.

The techniques successful for satisficing vs. optimal planning are almost disjoint for symbolic classical planning, but satisficing planning is much more effective in practice.

---
## STRIPS

>[!info] **Definition** (STRIPS planning task)
>A **STRIPS planning task** is a tuple $\Pi=(P,A,c,I,G)$ where:
>- $P$ is a finite set of *facts* (or *propositions*)
>- $A$ is a finite set of *actions*; each $a\in A$ is a triple *$a=(pre_{a}, add_{a}, del_{a})$* of subsets of $P$ referred to as the action’s precondition, add list, and delete list respectively; we require that $add_{a}\cap del_{a}=\varnothing$
>- $c:A\longrightarrow \mathbb{R}^{+}_{0}$ is the *cost function*
>- $I\subseteq P$ is the *initial state*
>- $G\subseteq P$ is the *goal*
>
>>[!question] Why do we use a function for cost $c$ instead of a relation?
>>It’s a simplification, indeed the cost of an action may vary based on other parameters (e.g. travel time of a road may vary based on the time of the day)

>[!example]
>![[Pasted image 20261105015559.png|400]]
>
>- $P: \{at(x), visited(x)\mid x \in \{Sydney, Adelaide, Brisbane, Perth, Darwin\}\}$
>- $I: \{at(Sydney), visited(Sydney)\}$
>- $G: \{at(Sydney), visited(x) \mid x \in \{Sydney, Adelaide, Brisbane, Perth, Darwin\}\}$
>- $a\in A: drive(x,y)$ where $x$, $y$ have a road
>	- $pre_{a}: \{at(x)\}$
>	- $add_{a}: \{at(y),visited(y)\}$
>	- $del_{a}: \{at(x)\}$
>- $c$:
>$$
>c(drive(x,y)) =
>\begin{cases}
>1   & \{x,y\} = \{Sydney, Brisbane\} \\
>1.5 & \{x,y\} = \{Sydney, Adelaide\} \\
>3.5 & \{x,y\} = \{Adelaide, Perth\} \\
>4   & \{x,y\} = \{Adelaide, Darwin\}
>\end{cases}
>$$

>[!info] **Definition** (STRIPS State Space)
>Let $\Pi=(P,A,c,I,G)$ be a STRIPS planning task. The **state space** of $\Pi$ is the tuple *$\Theta_{\Pi}=(S,A,c,T,I,S^{G})$* where:
>- $S=2^{P}$ (set of parts)
>- $T=\{s\xrightarrow{a}s'\mid a\in A(s),\ s'=s[\![a]\!]\}$, where
>	- $A(s)=\{a\in A\mid pre_{a}\subseteq s\}$, and for $a\in A(s)$:
>	- $s[\![a]\!]=(s\cup add_{a})\setminus del_{a}$ (for $a\notin A(s)$, $s[\![a]\!]$ is undefined, $s[\![a]\!]=\bot$)
>- $S^{G}=\{s\in S\mid G\subseteq s\}$
>
>We define (optimal) plans for $\Pi$ as (optimal) plans for $\Theta_{\Pi}$.
>
>For $\vec{a}=\langle a_{1},\dots,a_{n}\rangle$ (apply a set of actions sequentially):
>$$s[\![\vec{a}]\!]:=\begin{cases} s & n=0 \\ s[\![\langle a_{1},\dots,a_{n-1}\rangle]\!][\![a_{n}]\!] & n>0 \end{cases}$$
>
>>[!question] Is $\Theta_{\Pi}$ deterministic?
>>Yes: the successor state $s'$ in $s\xrightarrow{a}s'$ is uniquely determined by $s$ and $a$, through $s'=s[\![a]\!]$.

>[!example] STRIPS Encoding of Small "TSP"
>![[Pasted image 20261105021552.png|400]]
>
>- $P: \{at(x), visited(x)\mid x \in \{Sydney, Adelaide, Brisbane\}\}$
>- $I: \{at(Sydney), visited(Sydney)\}$
>- $G: \{visited(x) \mid x \in \{Sydney, Adelaide, Brisbane\}\}$ (Note: no $at(Sydney)$)
>- $a\in A: drive(x,y)$ where $x$, $y$ have a road
>	- $pre_{a}: \{at(x)\}$
>	- $add_{a}: \{at(y),visited(y)\}$
>	- $del_{a}: \{at(x)\}$
>- $c$:
>$$
>c(drive(x,y)) =
>\begin{cases}
>1   & \{x,y\} = \{Sydney, Brisbane\} \\
>1.5 & \{x,y\} = \{Sydney, Adelaide\}
>\end{cases}
>$$
>
>![[Pasted image 20261105022345.png]]
>
>Optimal plan (cost $1+1+1.5=3.5$): $\langle drive(Sy,Br),\ drive(Br,Sy),\ drive(Sy,Ad)\rangle$.
>
>>[!question] Is this actually the state space?
>>No, only the reachable part. E.g., $\Theta_{\Pi}$ also includes the states $\{v(Sy)\}$ and $\{at(Sy), at(Br)\}$.

