# Column Generation for the Cutting Stock Problem

## Problem Description

The Cutting Stock Problem deals with the problem of cutting stock material with the same, fixed width into smaller pieces, according to a set of orders specifying both the widths and the demand (weight) requirements, so as to minimize the amount of wasted material.

-----

## Motivation

Despite the great advancements that we've seen in mathematical optimization solver technology, some problems are still challenging to solve due to their large number of variables or constraints. Decomposition techniques — such as delayed column generation or Benders decomposition — apply the principle of "divide and conquer" to tackle these challenging problems.

Delayed column generation is typically used with large problems, which have so many variables that it's not possible to consider all of them explicitly. Experience with large problems indicates that, usually, most of these variables never enter the basis. Therefore, only a subset of these variables need to ever be considered when solving the problem. Delayed column generation exploits this fact to only generate variables (i.e., columns) that have the potential to improve the objective function. It's interesting to note that generating a column in the primal formulation amounts to adding a constraint to its dual. Thus, you can think of this technique as a cutting plane method in the dual space.

-----

## How to run this project

- Step 1: Prepare Finished Good and Mother Coil Data in ./data folder
            Task list will be saved in ./jobs_by_day folder
- Step 2: Run `24_process_tasks.py` to convert raw excel data to job list as dict-type 
- Step 3: Run `deflow_flow_multistocks_json.py` to execute Optima Model for all task 
            or Run test for a specific job/task with `test_dual_ver2.py`  
- Step 4: Check ./results folder

-----

## Solution Approach

Mathematical programming is a declarative approach where the modeler formulates a mathematical optimization model that captures the key aspects of a complex decision problem.

A mathematical optimization model has five components, namely:

- Sets and indices.
- Parameters.
- Decision variables.
- Objective function(s).
- Constraints.

#### Sets and Indices

$i \in \text{Stock}=\{1,2,\dots, N\}$: Set of available stock material units.

$j \in \text{Pieces}$: Set of final pieces.

#### Parameters

- $N \in \mathbb{N}$: Upper bound for the number of stock material units that can be used.
- $\text{Stock\_length} \in \mathbb{R}^+$: Length of any stock material unit.
- $\text{Piece\_length}_j \in \mathbb{R}^+$: Length of final piece $j$.
- $\text{Requirement}_j \in \mathbb{N}$: Demand requirement of final piece $j$.

#### Decision Variables

$\text{use}_i \in \{0,1\}$: 1 if stock material unit $i$ is used; 0 otherwise.

$\text{cut}_{i,j} \in \mathbb{N}$: Number of pieces of type $j$ cut out from stock material unit $i$.

#### Objective Function

- **Material Consumption:** Minimize the number of stock material units used.

$$
\begin{equation}
\text{Min} \sum_{i \in \text{Stock}}{\text{use}_i}
\tag{1.0}
\end{equation}
$$

One approach to solving this problem is to create a list of all finished parts, a list of stocks for each length, and then use a set of binary decision variables to assign each finished product to a particular piece of stock. This approach will work well for small problems, but the computational complexity scales much too rapidly with the size of the problem to be practical for business applications.

Given a list of patterns, the optimization problem is to compute how many copies of each pattern should be cut to fit the width of the Mother Coil and under the weight demand of the Finished Goods.

**A pattern $p$** is specified by the stock $s_p$ assigned to the pattern and integers $ap_{f}$ that specify the maximum of the finished parts of type $f$ are cut from stock $s_p$ and $\text{margin}^S_{s}$ is the allowed minimum margin of the according steel coil type; $wu^S_{s}$ the weight unit per mm weight of the coil stock, which is the weight divided by the MC width; $d^F_f$ the weight demand of the Finished Goods. A pattern $p \in P$ is feasible if:

$$
\begin{align}
& ap_{f}w^F_f  \leq   w^S_{s} - \text{margin}^S_{s} && \forall f\in F\\
& wu_{s} \times ( ap_{f} w^F_f) \leq d^F_f && \forall f\in F
\end{align}
$$
