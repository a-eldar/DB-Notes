1st note of *Design Theory*

---
# Database Normalization
Refers to a schema design that satisfies certain principles.
There are questions for when we should use normalization.
The common "principle" is:
"Normalize 'till it hurts, de-normalize 'till it works."
## Main Advantages
- Updates run quickly
- Inserts run quickly
- Tables are smaller
- Data integrity and consistency is preserved
## Main Disadvantages
More joins are required for querying.
## First Normal Form
The normalization process is split into stages. Let's take a quick look at stage 1.
Suppose we have the following table to use as an example.
**SalesStaff**
![[Pasted image 20250527173803.png]]
### Definition
A relation is in **first normal form** if every attribute contains atomic values, and there are no repeated attributes.
So the stage 1 normalization is about splitting the table into an additional table for the sake of having a **many to many** relationship.
Very intuitive and is probably what you would do anyway.
In our example:
![[Pasted image 20250527173836.png]]
### Duplication Problem
There is still a problem with that.
In our example, we can see the the `OffPhone` data is duplicated. This, generally speaking is a waste of storage.

As a general rule, we don't duplicate information, because it's a waste of storage and it makes it harder to update the value (in our case, update the office phone number).

Another problem is with **insertion**. Suppose the company has opened another office with no employees yet. Then we can't add its data to the tables because we don't have an `EmpId`. Same goes for **deleting** the last employee of the office.

We know that each `Office` has a single `OffPhone`. We know this because we have **domain knowledge** about how things work in the company.
# Functional Dependencies (FD)
Given $R(A_{1},\dots,A_{n})\quad X,Y\subseteq A_{1},\dots,A_{n}$
and $r$ is an instance of $R$ (meaning a specific table following the schema of $R$).
Then the functional dependency $X\to Y$ holds in $r$
if for every two tuples $s,t\in r$
$$
s[X]=t[X]\implies s[Y]=t[Y]
$$

Looking at the example from [[#Database Normalization#First Normal Form|Database Normalization]], there is a functional dependency $\text{Office}\to\text{OffPhone}$.

In general, the real world is what is interesting to us, so we get the dependencies from a schema designer.
## Implication
We can infer implied dependencies. For example:
Suppose we have a table with columns `EmpID, EmpName, DeptID, DeptName`
and the schema designer tells us
- `EmpID` $\to$ `EmpName`
- `EmpID` $\to$ `DeptID`
- `DeptID` $\to$ `DeptName`

Then we can infer `EmpID` $\to$ `DeptName`.
### Formal Definition
A set of functional dependencies $F$ over $R$ **implies** $X\to Y$
if for all $r$ instance of $R$:
$F$ holds in $r$ $\large\implies$ $X\to Y$ holds in $r$.
We can also say $X\to Y$ follows from $F$.
## Closure (סגור)
Define $X_{F}^+$ to be the set of attributes $A$ such that $X\to A$ follows from $F$.
If $F$ is obvious, we may write $X^+$.
Note that $A\in A_{F}^+$ always because $F \models A\to A$.
### Closure Algorithm
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

It's easy to see `Closure(X,F)` $\subseteq X_{F}^+$ by loop preservation (שמורת הלולאה).
Showing $X_{F}^+$ is more tricky. We do this by showing that if $A\not\in$ `Closure(X,F)` then $X\to A$ does **not** follow $F$.
### Proof
Assume $A\not\in$ `Closure(X,F)`.
Want to construct an instance that will satisfy $F$ but not $X\to A$.

We'll define the instance $r$ to have two rows $s,t$.
Suppose $V$ is the set of attributes (columns) returned by the algorithm.
For $B\in V$ we define this column of $r$ to have the same value between the two rows.
For $B\not\in V$ we define this column of $r$ to have different values between the two rows.

For example $R(A,B,C,D,E), V=\{ D,E \}$ then $r$ will be

| A   | B   | C   | D   | E   |
| --- | --- | --- | --- | --- |
| 0   | 0   | 0   | 0   | 0   |
| 1   | 1   | 1   | 0   | 0   |

Now show that $F$ is satisfied in $r$. Let $Y\to Z\in F$.
Separate to 2 cases:
**Case 1:** $Y\subseteq V$
Then by the way the algorithm works - $Z\subseteq V$.
Thus both rows are equal on both $Y$ and $Z$, and the dependency holds.

**Case 2:** $Y\not\subseteq V$
Then at least some of $Y$ is not in $V$, and therefore by the way we constructed $r$ - $s[Y]\neq t[Y]$, hence the dependency holds trivially.

So we've shown $r$ satisfies $F$. Left to show $r$ doesn't satisfy $X\to A$.

By the way of the algorithm (first step), $X\subseteq V$.
Hence $s[X]=t[X]$.
We assumed $A\not\in V$.
Hence $s[A]\neq t[A]$.
Q.E.D.
### Extended to Subsets
$X\to Y$ follows from $F$ iff $Y\subseteq X^+$.
Easy to show by considering each attribute in $Y$ separately.
Therefore we can conclude:
$X\to Y$ follows from $F$ iff $Y\subseteq$ `Closure(X,F)`.
# Keys and Superkeys
~={cyan}Why tf does key imply superkey lmao who thought of this?=~
$X$ is a **superkey** in $R$ if $X^+=R$.
$X$ is a **key** in $R$ if $X^+=R$
	and for all $Y\subset X$, $Y^+\subset R$
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
Note that there can be multiple keys while the algorithm only returns one of them.
Finding other keys is done next week - but I included it in this note for flow.
# Finding All Keys
In continuation to [[Semester B/DB/Design Theory/Framework#Keys and Superkeys#Finding a Key|Finding a Key]], we will now show a way to find all of them.
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
## Algorithm Analysis
### Initialization
We start by finding a single key $K$.
A Key Queue which we use to find new keys from existing ones - initialized as $\{ K \}$.
A Key set which is our end result, containing all found keys - initialized as $\{ K \}$.
### While Loop
We iteratively extract a key from our Key Queue. Call it $K$.
We then find all subsets of $K$ (namely $Y\cap K$) which we can replace with another subset (namely, $X$).
We use $X \to Y\in F$ for that, replacing $K$ with the new $K\setminus(Y\cap K)\cup X$

Therefore $S$ is a superkey.
Now if we don't already have a minimized version of $S$, we add it to our sets.
### Runtime
There could be an exponential number of keys.
The runtime is polynomial in the input and output together.
## Examples
### In one image
In one image - an example of the algorithm
![[Pasted image 20250603160848.png]]
### Another example - Step by step
Suppose
$$
R(A,B,C,D)
$$
$$
F = \{ AB\to C,\ C\to DA,\ BD\to C,\ AD\to B \}
$$
The algorithm would work something like:
Take $K=AB$ as initial key.
Pull $AB$ from queue
- Take $C\to DA\in F$ - $AB\cap DA=A$
  $S=BC,S'=C$
- Take $AD\to B\in F$ - $AB\cap B=B$
  $S=S'=AD$

Queue = $\{ C,AD \}$
Pull $C$ from queue
- $AB$ already seen
- Take $BD\to C\in F$
  $S=S'=BD$

Queue = $\{ AD,BD \}$
Pull $AD$ from queue
- Nothing new

Pull $BD$ from queue
- Nothing new

Result: $\{ AB, C, AD, BD \}$
