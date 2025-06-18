4th note in *Design Theory*
Continuing the 3rd note - [[Decompositions]]

---
# Dependency Preservation
Checking that dependencies are satisfied without having to join tables together.
## Informal Definition
A decomposition is **dependency preserving** if it is sufficient to check that each *functional dependency* holds on each part of the decomposition, in order to know that the dependencies hold in the original relation.
## Projection Of Dependencies
Exactly what you'd imagine. We just formalize it mathematically.
We define the **projection** of $F$ onto $R_{i}$
as all the dependencies that follow from $F$, that have both attributes from $R_{i}$.

For example if $R=(A,B,C),R_{1}=(A,B)$ and $F=\{ A\to B,B\to C,C\to A \}$
then $F_{R_{1}}=\{ A\to B,B\to A \}$ because both of them follow from $F$.
(**Note** that we also have the trivial dependencies in $F_{R_{1}}$ and the dependencies that stem from those two, but I omitted them)
## Formal Definition
Now we can define it formally:
The decomposition is dependency preserving if:
$X\to Y$ follows from $F$ iff $X\to Y$ follows from $F_{R_{1}}\cup\dots \cup F_{R_{n}}$

==DO NOTE:== While the definition talks about everything that follows from $F$, we should remember that it suffices to check only $F$ itself.
We take every element in $F$ and see if it follows from that union.
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
## Algorithm Explained By ChatGPT
This algorithm checks **whether a decomposition of a relational schema preserves dependencies**. In other words, it verifies if **the functional dependencies (FDs) from the original relation `R` are still enforceable using only the decomposed relations `R₁, R₂, ..., Rₙ`**.

---

### 🔍 **Intuitive Explanation**

When we decompose a relation `R` into smaller relations (e.g., to achieve normalization), we want to make sure we haven’t lost important functional dependencies.

This algorithm checks each functional dependency `X → Y` in the original set `F` and verifies whether we can still **infer** it using **only the FDs that apply to each piece** of the decomposition.

It simulates how attributes might "spread" through the pieces (`R₁, R₂, ..., Rₙ`) to see if the right-hand side `Y` of each dependency can still be logically derived from its left-hand side `X`, **using only FDs within each individual piece**.

---

### 🪜 Step-by-Step Natural Language Instructions (Performing by Hand)

Suppose you're given:

- A relation `R`
    
- A set of decomposed sub-relations `R₁, ..., Rₙ`
    
- A set of functional dependencies `F` on `R`
    

#### For every functional dependency `X → Y` in `F`, do the following:

1. **Start with `Z := X`** – this is your current known set of attributes.
    
2. **Repeat until nothing changes**:
    
    - For each `Rᵢ` in the decomposition:
        
        1. Find the intersection `Z ∩ Rᵢ` (attributes of `Z` that appear in `Rᵢ`).
            
        2. Compute the **closure** of this set _within `Rᵢ`_, using **only the FDs that apply to attributes of `Rᵢ`**.
            
        3. Add any newly discovered attributes **from `Rᵢ`** to `Z`.
            
    - Stop when no new attributes are added to `Z`.
        
3. **Check if all of `Y` is now in `Z`**.
    
    - If **not all attributes of `Y` are in `Z`**, the decomposition does **not** preserve this dependency. So, you can stop and return `"No"`.
        
4. If you finish this for all dependencies and they are all preserved, then return `"Yes"`.
    

---

### 💡 Key Concepts

- **Closure of a set of attributes** `A⁺` is the set of attributes functionally implied by `A`.
    
- You are using the **FDs from each `Rᵢ`**, meaning you can only apply FDs that are relevant **within that specific sub-relation**.
    
- The repeated loop is similar to computing a fixpoint – you keep expanding your known attributes until nothing changes.
    

---

### ✅ Simple Example

Let’s say:

- `R = {A, B, C, D}`
    
- Decomposition: `R₁ = {A, B}`, `R₂ = {B, C}`, `R₃ = {C, D}`
    
- FDs: `F = {A → D}`
    

To check if `A → D` is preserved:

1. Start: `Z := {A}`
    
2. Loop:
    
    - In `R₁`: `Z ∩ R₁ = {A}`, closure in `R₁` gives `{A}` (no FD in `R₁` uses `A`)
        
    - In `R₂`: `Z ∩ R₂ = {}`, nothing
        
    - In `R₃`: same, nothing  
        → No change in `Z`, so exit loop
        
3. Is `D` in `Z`? ❌ No → Return `"No"`
    

So, the decomposition does **not** preserve dependencies.

---
# BCNF or 3NF
Our end goal was, after all, to normalize the relations. So let's see if a decomposition is successful in doing that.
## Algorithm For $R_{i}$
Determining the normal form $R_{i}$ requires computing $F_{R_{i}}$ which takes exponential time.
But we have no choice...
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
So we know how to examine whether a decomposition is good. Now let's see ==how to actually find a good one==.

We're not quite there yet when it comes to finding a good decomposition. We first need to define **minimal cover**.
## Minimal Cover
A minimal cover contains only the functional dependencies that are necessary and are not implied from the others. Any redundant attributes in dependencies are also dropped. ==So no redundant FDs, and smaller FDs==.
For example:
$F=\{ AB\to C,A\to B,AC\to A \}$
will lead to $\{ A\to C,A\to B \}$.
### Definition
Given a set of FDs $F$, we define the minimal cover of $F$ as the set $G$ such that:
1. **Small Right-hand Side:** Every dependency in $G$ has a single attribute on the right-hand side
2. **Equivalence to $F$:** Every dependency following from $F$ also follows from $G$, and vice-versa.
3. **Cannot be made smaller by deletions:** If we obtain $H$ from $G$ by deleting one or more dependencies and/or deleting one or more attributes from a dependency, then $H$ is not equivalent to $F$.
### Algorithm
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
## Decomposition Into 3NF
### Algorithm
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
### Why It Works
#### Dependency Preserving
Each dependency is contained in a schema
#### Lossless
Can go back to [[Decompositions#Lossless Join Decomposition|here]] to remember how we check.
Intuitively, it's because every dependency exists in a schema, and we have a schema which is a superkey. So when using the table algorithm to check lossless, the row with the superkey will have be all **a**'s.
#### Every subschema is in 3NF
$XA$ is necessarily 3NF because $X$ is a key there (by way of construction of $X$).
(Proof omitted)
## Decomposition Into BCNF
We use a non-polynomial algorithm, even though there is a polynomial one, because it is a bit easier to run and understand.
### Algorithm
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
So the result is BCNF obviously.
Also it is lossless because $R_{1}\cap R_{2}=X$ and therefore $R_{1}\subseteq(R_{1}\cap R_{2})^+=X^+$
==It is important to note:== We **can't** ensure Dependency Preservation.
