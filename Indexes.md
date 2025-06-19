
# I/O Complexity
The number of read + writes to/from disk.
A read/write is per block/page in memory. 

# Index Structure
An index structure (such as hash table or BST) allows us to quickly find rows in our data.

# B+ Trees
The B+ tree is a balanced tree (meaning the distance from the root to the leaves is the same) where the leaves are a [doubly-linked list](https://en.wikipedia.org/wiki/Doubly_linked_list).
Every row in the database has a matching leaf value.

![](https://www.youtube.com/watch?v=K1a2Bk8NrYQ)

## Lookup
When looking up a key in the tree, we traverse along the branches where for every pair of adjacent keys in the node there is a branch to the left, one to the right, and one in between. So key values lower or equal to the smaller one go left, value in between go center and value greater or equal to the larger one go right.
The number of values in each node depends on the branching factor $d$ and is equal to $d-1$ naturally.

## Requirements
We know each node can have at most $d$ child nodes. We also require that each node has **at least** $\Large\left\lceil  \frac{d}{2}  \right\rceil$ child nodes.
~={blue}So the tree height is at most =~
$$
\large\log_{\lceil \frac d2 \rceil }n
$$

We need to be able to read each node into a single page in memory.
Suppose
- integer search key size = 4 bytes
- pointer size = 8 bytes
- block size = 1024 bytes

Then we require
$$
8d + 4(d-1) \leq 1024
$$
so $d \leq 85$.

In general, if we have
- search key size = $s$ bytes
- pointer size = $p$ bytes
- block size = $b$ bytes

Then the best branching factor is 
$$\Large\left\lfloor  \frac{b+s}{p+s}  \right\rfloor$$

Now then how many leaves can contain matching values in a search *at most*?
If we have $m$ matching rows and a branching factor of $d$,
then the number of leaves containing matching values is at most
$$
\Large\left\lceil  \frac{m}{\lceil d / 2 \rceil -1}  \right\rceil 
$$
Because each leaf contains **at least** $\lceil d / 2 \rceil -1$ values, so $m$ over that is the lower bar of the result.
## I/O Complexity of Search
Dependent on the tree depth and the needed traversal on the leaves.

# Comparing Queries
There are 3 type of scans to consider.
- INDEX UNIQUE SCAN - only traverse tree
- INDEX RANGE SCAN - traverse tree + follow leaves
- TABLE ACCESS BY ROWID - During an index scan, retrieve row from table.

## Examples
**INDEX UNIQUE SCAN**
```PostgreSQL
SELECT DISTINCT "true"
FROM Sailors
WHERE age > 58;
```
**INDEX RANGE SCAN**
```PostgreSQL
SELECT COUNT(*)
FROM Sailors
WHERE age > 58;
```
**TABLE ACCESS BY ROWID**
```PostgreSQL
SELECT name
FROM Sailors
WHERE age > 58;
```
## Number Of I/O Operations
In total, the number of I/O operations are
the depth of the tree 
\+ number of leaves traversed (always at least 1)
\~={blue}+ number of table blocks accessed=~.

Let's look at some examples.
Assume:
- ~={blue}1,000,000=~ rows in Sailors
- B tree of branching factor ~={green}20=~
- Ages are evenly distributed in range (~={red}20=~, ~={red}60=~]
### Index Unique Scan Example
```PostgreSQL
SELECT DISTINCT "true"
FROM Sailors
WHERE age > 58;
```
Then
- B+ tree depth at most $\lceil \log_{\color{green}10}{\color{blue}1,000,000} \rceil=6$
- Step 1 costs at most ~={purple}6=~
- Step 2 traverses a single leaf = ~={purple}1=~
- **Total:** ~={purple}7=~ I/O operations
### Index Range Scan Example
```PostgreSQL
SELECT COUNT(*)
FROM Sailors
WHERE age > 58;
```
Then
- Step 1 again costs at most ~={purple}6=~
- Approx. (~={red}60=~ - ~={purple}58=~)/(~={red}60=~ - ~={red}20=~) * ~={blue}1,000,000=~ = ~={yellow}50,000=~ matching rows.
- So matching rows will spread across at most ~={yellow}50,000=~/~={green}9=~ = ~={purple}5,556=~
- Step 2 therefore costs at most ~={purple}5,556=~
- **Total:** ~={purple}5,562=~ I/O operations
### Table Access By Row ID Example
```PostgreSQL
SELECT name
FROM Sailors
WHERE age > 58;
```
Then
- Step 1: Still only ~={purple}6=~ I/O operations to traverse tree
- Step 2: Read all relevant leaves, again ~={purple}5,556=~
- Step 3: Access matching rows - ++new++
- So remember we have ~={yellow}50,000=~ rows.
- Each row might be in a different block
- **Total:** ~={purple}6=~ + ~={purple}5,556=~ + ~={yellow}50,000=~ = ~={purple}55,562=~ I/O operations

# Not Always Worth It
Indexes may not always be the best choice. Sometimes it would actually be better to simply scan the whole table.
## Good Case Example
```PostgreSQL
SELECT avg(price)
FROM Orders
WHERE itemId = 'i242';
```
Suppose we have an index on `itemId`.
Also Assume
- `Orders` has ~={blue}10,000=~ rows with ~={yellow}10=~ rows per block
- The branching factor is ~={green}100=~
- Item `i242` appears in ~={red}20=~ orders

A full table scan will cost ~={blue}10,000=~/~={yellow}10=~ = 1,000
Using the index:
- Step 1: $\lceil \log_{\color{green}50}{\color{blue}10,000} \rceil = 3$
- Step 2: There are only ~={red}20=~ values with `i242` so we can say they probably are in the same leaf, so 1
- Step 3: ~={red}20=~
- **Total:** 24
## Bad Case Example
What if instead the item `i242` appeared in ~={red}1000=~ orders?
Then using the index:
- Step 1 would still be 3
- Step 2: ~={red}1000=~/~={green}49=~ = 21
- Step 3: ~={red}1000=~ to access the relevant blocks...
- **Total:** 1024
which is ==more than the full table scan==.

# Creating An Index
```PostgreSQL
CREATE INDEX ON table_name (column_name, ...)
```
For example
```PostgreSQL
CREATE INDEX ON Sailors(age)

CREATE INDEX ON Sailors(age, rating)
```

Optionally we can write
```PostgreSQL
CREATE [UNIQUE] INDEX [index_name] ON table_name ({column_name | expression} [ASC/DESC], ...) [WHERE predicate]
```
For example
```PostgreSQL
CREATE INDEX ON Sailors(UPPER(sname) DESC) WHERE rating > 5
```
Here the index is not on the names but on the uppercase of the names, so "John" and "john" will be the same.
## Note
### Multiple Column Index
An index created on multiple columns, considers the first column as the important one, and other columns are tie-breakers. So searching based on the second column using the index is pointless.

It does help however in common cases like:
```PostgreSQL
CREATE INDEX ON Orders(itemId, price);

SELECT avg(price)
FROM Orders
WHERE itemId = 'i242';
```
In this case, step 3 (reading blocks of rows) is not needed, because the information is in the index.

Another way we can do this is by writing something like
```PostgreSQL
CREATE INDEX ind_age ON Sailors(age) INCLUDE (rating);
```
which will include the rating value in the index values.
### Automatic Indexes
Indexes are created automatically for `PRIMARY KEY` and `UNIQUE`.
### Not Created When Querying
Indexes are not created when querying. The database designer has to decide which indexes to create.
### Costly
Each additional index increases storage and maintenance costs. Create them to speed up common and important queries.
Each additional **index column** also increases costs.

# Viewing a Query Plan
We can see how the database system would evaluate a query by writing
```PostgreSQL
EXPLAIN
SELECT name
FROM Sailors
WHERE sid = 22;
```
would show how the query **would be** evaluated.

We can also compare it with the real deal by writing
```PostgreSQL
EXPLAIN ANALYZE
```
Which will evaluate the query after estimating and show the real stats next to the estimations.

Read the documentation at [postgresql.org](https://www.postgresql.org/docs/12/using-explain.html#USING-EXPLAIN-BASICS), from "Now let's modify the query to add a `WHERE` condition:"  to "because then no extra sorting step is needed to satisfy the ORDER BY."