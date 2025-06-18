
Relational algebra operations receive tables and return a table.
A table is a set of rows (tuples), so not order or duplicates.

# Unary Operators
## Projection
$$
\Large\pi_{B_{1},\dots,B_{m}}
$$
Where $\{B_{1},\dots,B_{m}\} \subseteq\{ A_{1},\dots A_{n} \}$
extracts the **columns** of a table - meaning the table without some of the columns.
The number of rows may now be smaller due to duplicates.

## Selection
$$
\Large\sigma_{C}
$$
Where $C$ is a condition
extracts the **rows** of a table for which the condition is met.

# Binary Operators
## Union
$$
R\cup S
$$
Given $R(A_{1},\dots,A_{n}), S(A_{1},\dots,A_{n})$
returns $R\cup S$.

## Set Difference
$$
R-S
$$
Given $R(A_{1},\dots,A_{n}), S(A_{1},\dots,A_{n})$
returns $R\setminus S$.

## Cartesian Product
$$
R\times S
$$
Given $R(A_{1},\dots,A_{n}), S(B_{1},\dots,B_{m})$
returns $R\times S$, which is a table with columns $(A_{1},\dots,B_{m})$.
If $R,S$ share a column name $A$, it is written as $R.A, S.A$

# Additional Operator
## Auxiliary (renaming)
$$
\rho_{R(B_{1},\dots,B_{n})}(S(A_{1},\dots A_{n}))
$$

# Syntactic Sugar
## Intersection
$$
R\cap S
$$
$R(A_{1},\dots,A_{n}),S(A_{1},\dots,A_{n})$
Equivalent to $R-(R-S)$.

## Conditional Join
$$
R\bowtie_{C}S
$$
$R(A_{1},\dots,A_{n}),S(B_{1},\dots,B_{m})$
Equivalent to $\sigma_{C}(R\times S)$.

## Natural Join
$$
R\bowtie S
$$
$R(A_{1},\dots,A_{n}),S(B_{1},\dots,B_{m})$
Equivalent to performing:
1) cartesian product of $R$ and $S$
2) selection of tuples with the same values in $R$ and $S$ on all common attributes
3) projection to remain with one copy of each attribute

It is associative and commutative.

## Division
$$
R\div S
$$
$R(A_{1},\dots,A_{n},B_{1},\dots,B_{m}),S(B_{1},\dots,B_{m})$
Returns all tuples $(a_{1},\dots,a_{n})$ such that **for all** $(b_{1},\dots,b_{m})$ in $S$,
$(a_{1},\dots,a_{n},b_{1},\dots,b_{m})$ is in $R$.
Sometimes denoted $R/S$.
Again, $S$ has to be completely "contained" by that tuple.
It is like the operation yields $(a_{1},\dots,a_{n})$ such that
$$
(a_{1},\dots,a_{n}, S)\subseteq R
$$
For example, if you have a Teacher table and a Teaches table,
where Teaches contains triplets of (tid, sid, course),
then
$$
\Large\pi_{_{tid,sid}}(Teaches)\div \pi_{tid}(Teacher)
$$
yields all the students that are taught by every teacher.