3rd note in *Design Theory*
Although part of the same week as [[Normal Forms]], I thought it should be its own separate note.

---
How to normalize a relation?
We decompose our relation by extracting attributes according to dependencies.
I recommend watching the examples in the [lecture](https://huji.cloud.panopto.eu/Panopto/Pages/Viewer.aspx?id=3dea8f4d-018a-4a32-85c9-ac8500738b9e).
# Definition
A decomposition of $R$ is a set of relations $R_{1},\dots,R_{n}$ such that the set of attributes in $R_{1},\dots,R_{n}$ is the same as in $R$.
# Necessary Property
**Lossless**
meaning we can reconstruct $R$ by natural joins of $R_{1}\dots R_{n}$.
# Desired Properties
**Dependency Preserving**
We would like to only verify functional dependencies for each $R_{i}$ and therefore being assured that functional dependencies are preserved in $R$.
Put simply, we want to verify functional dependencies locally, without worrying about ruining them after join.

**In BCNF or 3NF**
We obviously want our decomposition to normalize our relation.
# Lossless Join Decomposition
A decomposition of $R$ into $R_{1},\dots,R_{n}$ is a **lossless join decomposition** with respect to $F$, if for every instance $r$ of $R$ that satisfies $F$:
$$
\large r=\pi_{R_{1}}r \bowtie\dots\bowtie \pi_{R_{n}}r
$$
meaning we can restore $R$ from $R_{1},\dots,R_{n}$.

Keep in mind that $r\subseteq \pi_{R_{1}}r \bowtie\dots\bowtie \pi_{R_{n}}r$  always holds regardless.
## Checking Decomposition Into 2 Relations
A decomposition of $R$ into $R_{1},R_{2}$ is a lossless join decomposition with respect to $F$, if at least one of the following holds:
- $R_{1}\subseteq (R_{1}\cap R_{2})^+$
- $R_{2}\subseteq (R_{1}\cap R_{2})^+$

For example if we have $R(A,B,C)$, $F=\{ A\to B,B\to C \}$
then $R_{1}=(A,B),R_{2}=(B,C)$ is a lossless join decomposition
because $B^+=BC=R_{2}$.
### The proof for why it's true
Take a row from the join $s \in\pi_{R_{1}}r \bowtie \pi_{R_{2}}r$
each part of it, $s_{R_{1}},s_{R_{2}}$ originated from a row in $r$.
assume WLOG $R_{1}\subseteq (R_{1}\cap R_{2})^+$

then because the shared attributes of $R_{1},R_{2}$ determine $R_{1}$
and the missing attributes of $R_{2}$ are in $R_{1}$ (as they are a decomposition)

we get that the values in $s_{R_{1}}$ are completely dependent on the shared values
and hence the original row in $r$ from which we got $s_{R_{2}}$ necessarily had the values of $s_{R_{1}}$ in its $R_{1}$ part.
Therefore $s\in r$ Q.E.D
## Checking Decomposition Into Many Relations
Let us start by describing helpful algorithms.
```pseudo
\begin{algorithm}
\caption{Checking Decomposition}
\begin{algorithmic}
\Procedure{CreateTable}{$R=(A_1,...,A_k), R_1, ..., R_n$}
	\State $T$ := new Table[n,k]
	\For{$i$ = 1 to $n$}
		\For{$j$ = 1 to $k$}
			\If{$A_j\in R_i$}
				\State T[i,j] = $a_j$
			\Else
				\State T[i,j] = $b_{ij}$
            \EndIf 
        \EndFor 
    \EndFor
	\Return T
\EndProcedure
\end{algorithmic}
\begin{algorithmic}
\Procedure{ChaseTable}{T,$F$}
	\While {there are rows $t,s$ in T and $X \to Y$ in $F$ such that
		t[X] = s[X] and t[Y] != s[Y]}
		\For{$A_i \in Y$}		
			\If{t[$A_i$] = $a_i$ or s[$A_i$] = $a_i$}
				\State s[$A_i$] = t[$A_i$] = $a_i$
			\Else
				\State t[$A_i$] = s[$A_i$]
			\EndIf
        \EndFor
    \EndWhile
\EndProcedure
\end{algorithmic}
\end{algorithm}
```
Note that in `ChaseTable` when we replace `b_ij` with `a_j`, we replace all instances of `b_ij` with `a_j`.
This means we also replace the table entries which we previously have changed using the `else` clause.
```pseudo
\begin{algorithm}
\begin{algorithmic}
\Procedure{TestDecomposition}{$R,R_1,...,R_n,F$}
	\State T := \Call{CreateTable}{$R,R_1,...,R_n$}
	\State \Call{ChaseTable}{T,F}
	\If {T contains row with only "a" values}
		\Return {"lossless"}
	\EndIf
	\Return {"not lossless"}
\EndProcedure
\end{algorithmic}
\end{algorithm}
```
So to sum up the process:
- we create the matching table
- we change it to satisfy $F$
- we check for a full "a" row
### Dual Purpose
Our table has a dual purpose.
If it has an "a" row, it serves as a proof of lossless join decomposition.
If it doesn't, it serves as a counter example.

