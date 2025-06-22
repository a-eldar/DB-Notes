# Indexes
[[Indexes]]
## B+ Trees
[[Indexes#B+ Trees|B+ Trees]]
If $d$ is the branching factor (number of child nodes)
then we require each node has at least $\left\lceil  \frac{d}{2}  \right\rceil$ child nodes.
and so the **depth** is at most $\large\log_{\left\lceil  \frac{d}{2}  \right\rceil}n$.
### Best branching factor
$b$ - block size
$p$ - pointer size (for pointing to child node)
$s$ - key size
$$
\large\left\lceil  \frac{b+s}{p+s}  \right\rceil
$$
### Number of Leaves With Matching Results
Suppose there are $m$ matching rows. Then
$$
\large\left\lceil  \frac{m}{\lceil d / 2 \rceil -1}  \right\rceil 
$$
Because each leaf contains **at least** $\lceil d / 2 \rceil -1$ values, so $m$ over that is the lower bar of the result.
## Number Of I/O Operations
There are 3 type of scans to consider.
- INDEX UNIQUE SCAN - only traverse tree
- INDEX RANGE SCAN - traverse tree + follow leaves
- TABLE ACCESS BY ROWID - During an index scan, retrieve row from table.

Best to look at the colored examples [[Indexes#Comparing Queries#Number Of I/O Operations|here]] and understand the logic.
# Nested Loops Join Complexity
## Block Nested Loops Join
If we have tables $S,R$ where ==$S$ is the smaller one== and we have $M$ blocks in memory.
$$
\large B(S)+B(R)\left\lceil  \frac{B(S)}{M-2}  \right\rceil 
$$
## Index Nested Loops Join
Let's assume $R(D,E),S(E,F)$ are the tables and we have an index on $S.E$.
Outer relation $R$ is read once, $B(R)$.
For each tuple in $R$, apply selection condition on join attribute: $T(R)\cdot\text{cost\_of\_select}$
We saw in [[Indexes]] how to calculate the cost of select.
So in total
$$
\large B(R) + T(R)\cdot\text{cost\_of\_select}
$$
# 2-Phase Merge Sort
$$
\left\lceil  \frac{B(R)}{M}  \right\rceil <M
$$
# Estimating Selectivity
The whole note is full of mathematical estimations so I recommend just going over it.
[[Estimating Selectivity]]
# Query Plans
## Calculating Runtime Of Join
Denote with $E_{R},E_{S}$ the results of operations on $R,S$ prior to performing the join operation according to the query plan.
### Block Nested Loops Join
$$
\large\text{Read}(E_{R})+ \text{Read}(E_{S})\cdot \left\lceil  \frac{B(E_{R})}{M-2}  \right\rceil
$$
### Index Nested Loops Join
$$
\large\text{Read}(E_{R}) + T(E_{R})\cdot \text{cost\_of\_select}
$$
