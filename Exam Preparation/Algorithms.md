# 2-Phase External Merge Sort
## Step 1
```pseudo
\begin{algorithm}
\caption{Phase 1 of Algorithm}
\begin{algorithmic}
\State left = 0
\While {left < B(R)}
	\State right = $\min(\text{left} + M, B(R)) - 1$
	\State read blocks $b_\text{left},...,b_\text{right}$ to main memory
	\State sort $b_\text{left},...,b_\text{right}$ in place
	\State write sorted sequence to disk
	\State left = left $+ M$
\EndWhile
\end{algorithmic}
\end{algorithm}
```
## Step 2
Take one block from each sorted sequence into memory, and then simply merge-sort them.
## Runtime
$3B(R)$ I/O operations
# Join Algorithms
## Block Nested Loops Join
So we want to perform a Join operation on two tables $S,R$ and produce an output $O$.
Suppose we have $M$ pages in our memory.
The algorithm will load the first $M-2$ blocks of $S$, the first block of $R$ and will leave one page for the blocks of $O$.

It will iteratively produce the blocks of $O$ one by one using the present blocks of $S,R$.
Once it is finished with the block of $R$ it will swap it out for the next, and so on.
Once it is finished with all of $R$'s blocks, it will swap the present $M-2$ blocks of $S$ for the next $M-2$ blocks of $S$. Repeat until the Join is done.

**So basically**, the algorithm behaves like nested loops where $O$ is the most inner loop, then $R$, then $S$; and $S$ gets to have as many present blocks as possible.
$$
\large B(S)+B(R)\left\lceil  \frac{B(S)}{M-2}  \right\rceil 
$$
$$
\large\text{Read}(E_{R})+ \text{Read}(E_{S})\cdot \left\lceil  \frac{B(E_{R})}{M-2}  \right\rceil
$$
## Index Nested Loops Join
Let's assume $R(D,E),S(E,F)$ are the tables and we have an index on $S.E$.
```pseudo
\begin{algorithm}
\caption{Index Nested Loops Join}
\begin{algorithmic}
\ForAll {block $b$ of $R$}
	\State read $b$ to memory
	\ForAll{tuple $(d,e) \in b$}
		\State search the index on $S.E$ for $e$
		\ForAll {index entry ($e$, $\texttt{tupleId}$)}
			\State access $S$ at $\texttt{tupleId}$ to retrieve $(e,f)$
			\State add $(d,e,f)$ to the output
		\EndFor
    \EndFor
\EndFor
\end{algorithmic}
\end{algorithm}
```
$$
\large B(R) + T(R)\cdot\text{cost\_of\_select}
$$
$$
\large\text{Read}(E_{R}) + T(E_{R})\cdot \text{cost\_of\_select}
$$
---
Design Theory
==================
# Closure And Keys
## Closure Algorithm
```pseudo
\begin{algorithm}
\caption{Closure Algorithm}
\begin{algorithmic}
\Procedure{Closure}{$X,F$}
	\State V := X
	\While{$\exists\ Y \to Z \in F$ such that
		$Y \subseteq V$
		\& $Z \not\subseteq V$}
		\State add Z to V
    \EndWhile 
	\Return V
\EndProcedure
\end{algorithmic}
\end{algorithm}
```
Basically the algorithm iteratively adds attributes to the result in a chain starting from X.
## Finding a Key
```pseudo
\begin{algorithm}
\caption{Finding A Key}
\begin{algorithmic}
\Procedure{Minimize}{$X,F$}
	\For {$A \in X$}
		\If {$A \in (X\setminus A)^+$}
			\State $X = X\setminus A$
		\EndIf
	\EndFor
	\Return X
\EndProcedure
\Procedure{FindKey}{$R,F$}
	\Return \Call{Minimize}{$R,F$}
\EndProcedure
\end{algorithmic}
\end{algorithm}
```
## Finding All Keys
```pseudo
\begin{algorithm}
\caption{Finding All Keys}
\begin{algorithmic}
\Procedure{AllKeys}{R,F}
	\State K := \Call{FindKey}{R,F}
	\State KeyQueue := \{K\}
	\State Keys := \{K\}
	
	\While{KeyQueue is not empty}
		\State K := KeyQueue.dequeue()
		\ForAll{$X \to Y \in F$
				where $Y \cap K \neq \emptyset$}
			\State S := $(K \setminus Y) \cup X$
			\If {S does not contain any $J \in$ Keys}
				\State S' := \Call{Minimize}{S,F}
				\State Keys.add(S')
				\State KeyQueue.add(S')
			\EndIf
        \EndFor
    \EndWhile
	\Return Keys
\EndProcedure
\end{algorithmic}
\end{algorithm}
```
Let's break it down part by part
### Algorithm Analysis
#### Initialization
We start by finding a single key $K$.
A Key Queue which we use to find new keys from existing ones - initialized as $\{ K \}$.
A Key set which is our end result, containing all found keys - initialized as $\{ K \}$.
#### While Loop
We iteratively extract a key from our Key Queue. Call it $K$.
We then find all subsets of $K$ (namely $Y\cap K$) which we can replace with another subset (namely, $X$).
We use $X \to Y\in F$ for that, replacing $K$ with the new $K\setminus(Y\cap K)\cup X$

Therefore $S$ is a superkey.
Now if we don't already have a minimized version of $S$, we add it to our sets.
#### Runtime
There could be an exponential number of keys.
The runtime is polynomial in the input and output together.
# Lossless Join Decomposition
## Checking Decomposition Into 2 Relations
A decomposition of $R$ into $R_{1},R_{2}$ is a ==lossless join== decomposition with respect to $F$, if at least one of the following holds:
- $R_{1}\subseteq (R_{1}\cap R_{2})^+$
- $R_{2}\subseteq (R_{1}\cap R_{2})^+$
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
# Dependency Preservation And Projection
## Algorithm Testing For Preservation
This algorithm doesn't calculate the entire $F_{R_{i}}$ because those can be way too big.

```pseudo
    \begin{algorithm}
    \caption{Dependency Preserving Test}
    \begin{algorithmic}
    \Procedure{IsDependencyPreserving}{$R,R_1,...,R_n,F$}
	    \ForAll{$X\to Y\in F$}
		\State{$Z:=X$}
		\Repeat
			\State{$Z':=Z$}
			\For{$i$=1 to $n$}
				\State $Z=Z\cup((Z\cap R_i)^+\cap R_i)$
            \EndFor
        \Until{$Z=Z'$}
        \If{$Y\not\subseteq Z$}
	        \Return "No"
        \EndIf
        \EndFor
	    \Return "Yes"
    \EndProcedure
    \end{algorithmic}
    \end{algorithm}
```
### 🔍 **Intuitive Explanation**

When we decompose a relation `R` into smaller relations (e.g., to achieve normalization), we want to make sure we haven’t lost important functional dependencies.

This algorithm checks each functional dependency `X → Y` in the original set `F` and verifies whether we can still **infer** it using **only the FDs that apply to each piece** of the decomposition.

It simulates how attributes might "spread" through the pieces (`R₁, R₂, ..., Rₙ`) to see if the right-hand side `Y` of each dependency can still be logically derived from its left-hand side `X`, **using only FDs within each individual piece**.
## Computing Dependency Projection

```pseudo
\begin{algorithm}
\caption{Computing a set of dependencies equivalent to $F_{R_i}$}
\begin{algorithmic}
\Procedure{ComputeDependenciesInProjection}{$R,R_i,F$}
	\State G := $\emptyset$
	\ForAll{$X\subseteq R_i$}
		\State add the dependency $X\to (X^+ \cap R_i)$ to G
    \EndFor
\EndProcedure
\end{algorithmic}
\end{algorithm}
```
Then we check each dependency whether it satisfies **for** $R_{i}$ BCNF as we've seen [[Normal Forms#BCNF#Easier Definition|here]],
or 3NF as we've seen [[Normal Forms#3NF#Easier Definition|here]].
Do note that we can entirely skip the trivial dependencies and the ones that have a key on the left.
# Finding A Decomposition
## Minimal Cover
Given a set of FDs $F$, we define the minimal cover of $F$ as the set $G$ such that:
1. **Small Right-hand Side:** Every dependency in $G$ has a single attribute on the right-hand side
2. **Equivalence to $F$:** Every dependency following from $F$ also follows from $G$, and vice-versa.
3. **Cannot be made smaller by deletions:** If we obtain $H$ from $G$ by deleting one or more dependencies and/or deleting one or more attributes from a dependency, then $H$ is not equivalent to $F$.

```pseudo
\begin{algorithm}
\caption{Finding a Minimal Cover}
\begin{algorithmic}
\Procedure{ComputeMinimalCover}{F}
	\State G := $\emptyset$
	\ForAll{$X\to Y \in F$}
	\Comment{Small right-hand side}
		\ForAll{$A\in Y$}
			\State add $X\to A$ to G
        \EndFor
    \EndFor
    \ForAll{$X\to A\in$ G}
    \Comment{Drop redundant attributes in left-hand side}
	    \ForAll{$B\in X$}
		    \If{$A\in (X\setminus B)^+$}
			    \State remove $B$ from $X$
            \EndIf
        \EndFor
    \EndFor
    \ForAll{$X\to A\in$ G}
    \Comment{Drop redundant dependencies}
	    \If{$X\to A$ follows from the other dependencies}
		    \State remove $X\to A$ from G
        \EndIf
    \EndFor
    \Return G
\EndProcedure
\end{algorithmic}
\end{algorithm}
```
For example if you run the algorithm on
$F=\{ A\to B, ABCD\to E, EJ\to GH, ACDJ\to EG \}$
After step 1 we'll get
$\{ A\to B, ABCD\to E, EJ\to G, EJ\to H, ACDJ\to E, ACDJ\to G \}$
After step 2 we'll get
$\{ A\to B,ACD\to E,EJ\to G,EJ\to H,ACD\to E, ACDJ\to G \}$
After step 3 we'll get
$\{ A\to B, EJ\to G, EJ\to H, ACD\to E \}$
## Finding 3NF Decomposition
```pseudo
\begin{algorithm}
\caption{Decomposition into 3NF}
\begin{algorithmic}
\Procedure{Find3NFDecomposition}{R,F}
	\State G:= \Call{ComputeMinimalCover}{F}
	\ForAll{$X\to A\in$ G}
		\State add the schema $XA$
    \EndFor
    \If{no schema created contains a key}
	    \State add a key as a schema
    \EndIf
    \State remove schemas contained in other schemas
\EndProcedure
\end{algorithmic}
\end{algorithm}
```
## Finding BCNF Decomposition
```pseudo
\begin{algorithm}
\caption{simple algorithm for a lossless BCNF decomposition}
\begin{algorithmic}
\Procedure{FindBCNFDecomposition}{R,F}
	\If{R is in BCNF}
		\Return R
    \EndIf
    \State let $X\to Y$ be a BCNF violation
    \State $R_1 = X^+$
    \State $R_2 = X\cup (R\setminus X^+)$
    \Return \Call{FindBCNFDecomposition}{$R_1, F_{R_1}$} $\cup$
    \Call{FindBCNFDecomposition}{$R_2, F_{R_2}$}
\EndProcedure
\end{algorithmic}
\end{algorithm}
```
