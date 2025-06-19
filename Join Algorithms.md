
# Intro
Suppose we two tables `Sailors` and `Reserves` for sailors and their reservations of boats. Now say we want to compute their [[SQL#Queries#Join#Natural Join|natural join]].
If we have enough space in memory for all the blocks of `Sailors` and `Reserves` **and** the output of the join, then we're good, but we might not always have that.

Instead, at the worst case scenario, if we just have 3 pages, one for each, we can:
Hold one block from `Sailors`, one block from `Reserves` and one block for the output, and then swap a block for the next when we are done with it.

So we need some external algorithm to perform that join operation with all the swapping, and we want it to be efficient. However it isn't very clear what that algorithm should look like. How should we swap the blocks in the most efficient way?

We will look at ~={blue}2=~ join algorithms:
- block nested loops join
- index nested loops join

# Block Nested Loops Join
## How the algorithm works
So we want to perform a Join operation on two tables $S,R$ and produce an output $O$.
Suppose we have $M$ pages in our memory.
The algorithm will load the first $M-2$ blocks of $S$, the first block of $R$ and will leave one page for the blocks of $O$.

It will iteratively produce the blocks of $O$ one by one using the present blocks of $S,R$.
Once it is finished with the block of $R$ it will swap it out for the next, and so on.
Once it is finished with all of $R$'s blocks, it will swap the present $M-2$ blocks of $S$ for the next $M-2$ blocks of $S$. Repeat until the Join is done.

So basically, the algorithm behaves like nested loops where $O$ is the most inner loop, then $R$, then $S$; and $S$ gets to have as many present blocks as possible.

## Complexity
Suppose $B(T)$ is the number of blocks in table $T$.
In the algorithm we read all of $S$ blocks, and for every $M-2$ blocks of $S$, we read all of $R$ blocks.
In total we get
$$
\large B(S)+B(R)\left\lceil  \frac{B(S)}{M-2}  \right\rceil 
$$
We notice that when using this algorithm, it is better to choose the ==smaller== table as $S$.
Also note that we ==don't include the output blocks== ($O$) in our calculations. This is because we don't write them to the disk but rather display them on the screen.
### Optimizing By Pushing Selections
Suppose we want to compute
```PostgreSQL
SELECT *
FROM Sailors S, Reserves R
WHERE S.sid = R.sid AND date = '1/1/01';
```
That is we take the natural join with a condition on the `date` of the reservation.
So if we first natural join $S,R$ and then take the condition on the date, we will have the same complexity for the Join.
However, if we include in the Join operation only the rows from $R$ which satisfy the `date` condition, our Join operation will take much less. In total it would take reading $R$ once and then performing the Join on the much smaller result.

This will take much less than the previous way.
In conclusion, we should always push the selection if possible.

# Index Nested Loops Join
We use an [[Indexes|index]] to find matching rows in the second table.

Regardless of the number of pages available in memory, we will need exactly **4**.
One for $R$ blocks, one for $S$ blocks, one for the index search and one for the output.
## Pseudo Code
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
## Complexity
Suppose $B(T')$ is the number of blocks in table $T'$.
Suppose $T(T')$ is the number of tuples in table $T'$.

Outer relation $R$ is read once, $B(R)$.
For each tuple in $R$, apply selection condition on join attribute: $T(R)\cdot\text{cost\_of\_select}$
We saw in [[Indexes]] how to calculate the cost of select.
So in total
$$
\large B(R) + T(R)\cdot\text{cost\_of\_select}
$$
### Optimizing By Pushing Selections
Exactly like [[Join Algorithms#Block Nested Loops Join#Optimizing By Pushing Selections|in Block Nested Loops Join]] except now we can't push the selection if it selects from the table of the index ($S$) and the index doesn't allow us to find based on the condition.