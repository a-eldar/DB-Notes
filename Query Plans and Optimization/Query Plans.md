2nd note of *Query Plans and Optimization*

---
A query plan specifies how to evaluate a query.
To create a query plan for an SQL query:
1. Translate the query to algebra
2. Choose evaluation methods for each operator
3. Push projections and selections where possible (but see [[Query Plans#Push Selections and Projections#**Not** Always Pushing|this important note]])

We have already seen number 3 in [[Join Algorithms]], so we'll focus on 1 and 2.
# Translate To Algebra
This one is quite intuitive so I'll sum things up briefly with an example:
```PostgreSQL
SELECT S.sid, S.sname, R.bid
FROM Sailors S, Reserves R
WHERE S.sid = R.sid AND
	rating = 10 AND
	date < 1/1/11;
```
translates to
$$
\large\pi_{\text{sid,sname,bid}}(\sigma_{\text{rating = 10 and date < 1/1/11}}(\text{Sailors} \bowtie \text{Reserves}))
$$

After that we simply translate this expression to a tree as follows:
![[Pasted image 20250518200848.png]]
# Evaluation Methods
Now that we have the algebra tree, we need to choose how we evaluate (meaning perform) each operation.
Recall the different [[Join Algorithms]] we saw. Then we can, for example, write "BNL" next to the join operation like so
![[Pasted image 20250518201349.png]]
to indicate we are using the [[Join Algorithms#Block Nested Loops Join|BNL Join]] algorithm.
==Important to note== that we consider the **outer relation** of the algorithm to be the one on the left, so in this case Sailors.

A tree with the specification of which algorithms are used at each step ==**is a query plan**.==
## Pipelining
Instead of evaluating the entire join, then selecting the rows who follow the condition, then picking the projection columns, we do it row by row.
This is pipelining. Pipelining is the way it is done by default, if not specified otherwise.
# Push Selections and Projections
Since `rating` is a selection unique to Sailors, we can push it to get:
![[Pasted image 20250518202517.png]]
(Note that ~={red}**SM**=~ stands for Sort-Merge join which is omitted this year)
using an Index Scan for the condition on Sailors.

We can also push the other condition and the projection to get:
![[Pasted image 20250518202805.png]]

## **Not** Always Pushing
Consider the following:
![[Pasted image 20250518204122.png]]
Should we push the `date` condition into `Reserves`? NO
because since we have an index on `R.sid` we can perform [[Join Algorithms#Index Nested Loops Join|an Index Nested Loops join]] with `R` being the inner relation.
However, if we push the condition, since we don't have an index on `(sid, date)` we cannot.
# Choosing The Best Plan
Typically:
1. Best way to apply each selection ($\sigma$)
2. Best Join method
3. Full push of selection and projection where possible

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
