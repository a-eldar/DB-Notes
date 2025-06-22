
# Sorting In Queries
## Order By
Sometimes when we use `ORDER BY` we don't actually need to sort anything.
For example: if we have both
```PostgreSQL
CREATE INDEX ON Sailors(name);
------------------------------
SELECT name
FROM Sailors
ORDER BY name;
```
then going over the index leaves will already grant the desired result.

However if instead
```PostgreSQL
SELECT name
FROM Sailors
ORDER BY name, age;
```
then the database system will opt for sorting.
## Other Cases
When we have `SELECT DISTINCT`, `UNION`, `EXCEPT`, one way the database system removes duplicates is by sorting.
Also when we use `GROUP BY`, the database might sort according to the `GROUP BY` parameters.

# 2-Phase External Merge Sort
Why not use quicksort? Because there is too much data, and even if we had enough **virtual memory**, we would make a lot of expensive I/O operations during the sort.

Suppose $B(R)$ is the number of blocks in relation $R$ and suppose we have $M$ blocks available in memory.
In our course we will only deal with a simplified case where
$$
\left\lceil  \frac{B(R)}{M}  \right\rceil <M
$$
## The Algorithm
Broken down into two steps:
1) Create sorted sequences
2) Merge sorted sequences

Let's say the blocks or $R$ are $b_{0},\dots ,b_{B(R)-1}$.
### Step 1
Step 1 algorithm is
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
Sorting $b_{left},\dots,b_{right}$ means sorting all of the rows across the $M$ blocks.
For sorting the blocks in place we can simply use quick sort, since now the entire data sorted is in memory.
~={blue}In this part we read $R$ once and we wrote it once so it took $2B(R)$ I/O operations.=~
### Step 2
In this step we merge the sorted sequences from step 1.
We do this in the most natural way:
We simply begin with having the first block of each sequence in memory.
Each block is sorted so we look at the first value of each block, and from all of those values, we choose the smallest one.
![[Pasted image 20250513182207.png]]
We put that value into a result block.
We then move the corresponding pointer one step forward in the block.
![[Pasted image 20250513182304.png]]
Every time we finish a block in a sequence, we swap it for the next block in the sequence.
We do this until the result block is filled, in which case we write it into disk, and begin filling a new block instead of it.
![[Pasted image 20250513182908.png]]
This goes on until all the sorted sequences have been merged and inserted into the disk.

We assumed here that we have enough memory blocks to place one block from each sequence into memory and still have an extra block for output.
This is because we assumed
$$
\left\lceil  \frac{B(R)}{M}  \right\rceil <M
$$
~={blue}How much did this cost us?
Remember that we don't count writing the final output (because we assume we just display it on the screen or something) so all we did is simply read every block of the table once, so $B(R)$ I/O operations.=~

In total, the number of I/O operation it took is $3B(R)$.