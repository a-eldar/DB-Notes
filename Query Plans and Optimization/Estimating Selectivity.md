1st note of *Query Plans and Optimization*.

---
The database system would like to know the best strategy for querying the database.
In order to do that, it must collect statistics about the database and estimate the cost of each approach.
# Choosing a Join Order
Suppose we have the following two equivalent ways of querying our database:
$$
{\color{blue}(}\sigma_{\text{age}>90}\ \text{Sailors}\bowtie \text{Reserves}{\color{blue})}
\bowtie \sigma_{\text{color}=\text{"pink"}} \text{Boats}
$$
$$
\sigma_{\text{age}>90}\ \text{Sailors}\bowtie {\color{blue}(}\text{Reserves}
\bowtie \sigma_{\text{color}=\text{"pink"}} \text{Boats}{\color{blue})}
$$
We as intelligent humans, can guess that the first method is better since there are probably not many sailors older than 90.

So more generally, if we know the statistics of how much reduction we would get for each condition (each $\sigma$) then we would know the better approach.
# Collecting Statistics
Every time we run `analyze`, the database system collects statistic about the data.
It also happens occasionally when the database is idle.
# Size of Selection Output
Let us denote
$B(R)$ - the number of blocks in table $R$
$T(R)$ - the number of rows
$V(R,A)$ - the number of different values of $R.A$
## Equality
Consider the query type of selecting based on equality to a constant.
```PostgreSQL
SELECT *
FROM R
WHERE A = 'a';
```
If we ==assume uniform distribution==, the size of the query result (#rows) would be
$$
\large\frac{T(R)}{V(R,A)}
$$
## Inequality
Now the query type is
```PostgreSQL
SELECT *
FROM R
WHERE A <= x;
```
Suppose the ==range of $A$ is known to be $[y,z]$ with uniform distribution==.
Then the size of the query result would be
$$
\large T(R) \frac{x-y+1}{z-y+1}
$$
In this case the range was **given**. If the range is **not given** we assume the number of rows in the result to be
$$
\frac{T(R)}{3}
$$
meaning ==one third of rows==. Why? Because yes.
## Multiple Selections
### 2 Equalities
```PostgreSQL
SELECT *
FROM R
WHERE A = 'a' and B = 'b';
```
Assume independence of conditions. Then the number of rows in the result is
$$
\large\frac{T(R)}{V(R,A)\times V(R,B)}
$$
This is because it is equal to the number of rows times the probability of a row being in the result, which is the **expectancy**. Note that since we assumed independence, the probability of a row being in the result is $\frac{1}{V(R,A)\times V(R,B)}$.
### Equality and Inequality
```PostgreSQL
SELECT *
FROM R
WHERE A = 'a' and B < 'b';
```
Same idea, not given the range of $B$, we estimate
$$
\large \frac{T(R)}{V(R,A)\times 3}
$$
### 2 Inequalities
```PostgreSQL
SELECT *
FROM R
WHERE A < 'a' and B < 'b';
```
You get the point by now
$$
\large \frac{T(R)}{3\times 3}
$$

Now if we have `or`, we use the exact same logic as before (🎤) to get
$$
\large T(R)\left( 1-
\left( \underbrace{ 1- \frac{1}{V(R,A)} }_{ Pr[A\neq'a'] } \right)
\times \left( \underbrace{ 1 - \frac{1}{V(R,B)} }_{ Pr[B\neq'b'] } \right)
\right)
$$
where we use the common logic of $or = not(and(not,not))$
also known as d'Morgan's law.

## Size Of Join
```PostgreSQL
SELECT *
FROM R, S
WHERE R.A = S.A;
```
We again assume uniform distribution.
For every row of $R$, there should be $\frac{T(S)}{V(S,A)}$ rows of $S$ that match it, so from $R$'s perspective, the size of the join is
$$
T(R) \times \frac{T(S)}{V(S,A)}
$$
But we can also look from $S$'s perspective to get
$$
T(S)\times \frac{T(R)}{V(R,A)}
$$
So we estimate the real result to be the smaller of the two, meaning
$$
\large\frac{T(R) \times T(S)}{\max\{ V(R,A),V(S,A) \}}
$$
### Foreign Key Constraint
If we add that $A$ is a **key** in $R$ and a **foreign key** in $S$
then for every row of $S$ we expect $R$ to have **exactly one** matching row.
Hence the result is $T(S)$.
==This works with the previous formula==, when you realize:
$$
\max\{ V(R,A),V(S,A) \}=V(R,A)
$$
$$
V(R,A)=T(R)
$$
## Join And Equal
```PostgreSQL
SELECT *
FROM R, S
WHERE R.A = S.A and R.B = 'b';
```
As per usual with the assumptions
$$
\large \frac{T(R)\times T(S)}{\max\{ V(R,A),V(S,A) \}\times V(R,B)}
$$
Note that it doesn't matter if you take the join before or after the equal.
## Join On Two
```PostgreSQL
SELECT *
FROM R, S
WHERE R.A = S.A and R.B = S.B;
```
As per usual with the assumptions
$$
\large \frac{T(R)\times T(S)}{\max\{ V(R,A),V(S,A) \}\times \max\{ V(R,B),V(S,B) \}}
$$
