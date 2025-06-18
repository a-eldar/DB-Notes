


A note for SQL queries knowledge.

**SQL is case insensitive!**
# Types
```SQL
integer
float
double
year
varchar (string)
varchar(N) - string up to N characters
char(N) - exactly N characters
```

# Constraints
## Column Constraints
```PostgreSQL
NOT NULL
CHECK(condition{for example gender='M'})
UNIQUE
PRIMARY KEY
DEFAULT(value)
```
## Table Constraints
```PostgreSQL
CHECK(condition)
UNIQUE(columnName1, columnName2,...)
PRIMARY KEY(columnName1, columnName2,...)
FOREIGN KEY(columnName) REFERENCES tableName(columnName) [ON DELETE CASCADE]
```

# Create Table
```SQL
CREATE TABLE TableName {
	columnName columnType columnConstraints,
	columnName columnType columnConstraints,
	...
	tableConstraint,
	tableConstraint,
	...
};
```

# Drop Table
```PostgreSQL
DROP TABLE tableName [CASCADE];
```
`CASCADE` here removes dependencies from other tables.

# Queries
## Optional and Mandatory
```SQL
SELECT
FROM
```
are mandatory, while
```SQL
WHERE
GROUP BY
HAVING
ORDER BY
```
are optional.
All queries end with a `;`
## Basic Format of a Query
```SQL
SELECT [Distinct] Attributes
FROM relations
WHERE condition;
```
Is equivalent to (conceptually)
$$
\pi_{Attributes}(\sigma_{condition}(\times_{relations}))
$$
`Distinct` is makes it so rows don't appear twice.
## Calculations in Queries
We can do something like
```SQL
SELECT sid, age/rating
FROM Sailors
WHERE rating=10;
```
and get for the second column the value of `age` divided by `rating`.
The yielded column name will not be `age/rating` but something like `?column?`.
If we want to name the column, we can so by writing
```SQL
SELECT sid, age/rating AS ar
FROM Sailors
WHERE rating=10;
```
## Conditions
```SQL
SELECT sid FROM Sailors
WHERE (rating != 10 AND age IS NOT NULL)
	OR sname LIKE 'A%_e';
```
The rating and age conditions are self explanatory.
The `LIKE` operation is for text pattern matching.
`'_'` represents a single arbitrary character, and `'%'` represents 0 or more characters.
So in the above condition we choose the sailors with names that have at least three characters, that start with 'A' and end with 'e'.

## Order
If we don't specify an order, the query result will not be ordered.
`ORDER BY` lets us specify a characteristic by which to order the query result.
For example
```SQL
SELECT sid FROM Sailors
ORDER BY age;
```
will yield a list of the sailor IDs, by order of the sailors' age (increasing by default).
We can also have it ordered by descending order, and also have multiple criteria in order of their importance. For example
```SQL
SELECT sid FROM Sailors
ORDER BY rating ASC, age DESC;
```
Here `age` is the "tie breaker".

## From
We said that `FROM Sailors, Reserves, Boats` means the query goes over the cartesian product of the three.
We can also rename the tables in the query like we can rename columns.
Consider this example:
```SQL
SELECT boatname
FROM Sailors, Reserves, Boats
WHERE Sailors.sid = Reserves.sid
	AND Reserves.bid = Boats.bid
	AND sname = "Alice";
```
We can shorten it to
```SQL
SELECT boatname
FROM Sailors S, Reserves R, Boats B
WHERE S.sid = R.sid
	AND R.bid = B.bid
	AND sname = "Alice";
```
Note that `sname` need not a prefix because it is unambiguous in this case.

## Join
$WHERE \to FROM$
### Inner Join
The following are equivalent:
```PostgreSQL
SELECT S1.sname, S2.sname
FROM Sailors S1, Sailors S2
WHERE S1.sid != S2.sid AND
	S1.sname - 'Rusty';
```

```PostgreSQL
SELECT S1.sname, S2.sname
FROM Sailors S1 INNER JOIN Sailors S2
	ON (S1.sid != S2.sid)
WHERE S1.sname - 'Rusty';
```
So a shorthand, though not so naturally used in this case.
This is the conditional join from relational algebra $\bowtie_{C}$.
### Natural Join
A much more *natural* use is
```PostgreSQL
SELECT S.sname
FROM Sailors S, Reserves R
WHERE S.sid = R.sid AND S.age > 20;
```
is the same as
```PostgreSQL
SELECT S.sname
FROM Sailors S NATURAL JOIN Reserves R
WHERE S.age > 20;
```
when the two tables share `sid`.
### Outer Join
Suppose we have the following tables:
Sailors

| sid | sname   | rating | age |
| --- | ------- | ------ | --- |
| 22  | Alice   | 7      | 45  |
| 31  | Barbara | 8      | 55  |
| 58  | Carol   | 9      | 35  |
| 70  | Alice   | 10     | 25  |
Reserves

| sid | bid | day |
| --- | --- | --- |
| 22  | 101 | 1/1 |
| 58  | 103 | 2/1 |
Then naturally joining the two together will drop sid's 31 and 70, since they don't appear in Reserves.
So instead we can do
```PostgreSQL
SELECT sid, bid
FROM Sailors NATURAL LEFT OUTER JOIN Reserves;
```
which will keep all values from Sailors, and result in

| sid | bid  |
| --- | ---- |
| 22  | 101  |
| 31  | NULL |
| 58  | 103  |
| 70  | NULL |

In addition there is `RIGHT OUTER JOIN` which the same but the other way around, and we also have `FULL OUTER JOIN` which will include all the rows from left **and** right,

An example for a popular use-case:
```PostgreSQL
SELECT S.sid, COUNT(bid)
FROM Sailors S NATURAL LEFT OUTER JOIN Reserves R
GROUP BY S.sid;
```
This will show for any sailor, how many boats he reserved, ~={blue}and in addition=~ thanks to the `LEFT OUTER JOIN` we see the sailors who have not reserved any boats, with the number 0.

## DISTINCT
Notice that in the case of
```PostgreSQL
SELECT sid FROM Sailors;
```
it is the same as
```PostgreSQL
SELECT DISTINCT sid FROM Sailors;
```
because either way we will get `sid` once.
However
```PostgreSQL
SELECT sid FROM Reserves;
```
is not the same as
```PostgreSQL
SELECT DISTINCT sid FROM Reserves;
```
because in the first one we will get each `sid` for every reserve that sailor made, while in the second one we will get the list of `sid` of sailors that made a reserve.

Important note:
```PostgreSQL
SELECT [DISTINCT] sid
FROM Sailors NATURAL JOIN Reserves
```
works the same way as the previous example `SELECT [DISTINCT] sid FROM Reserves`.

## Three SET Operators
**UNION, EXCEPT, INTERSECT**
Operating on query results, these are legal if the queries return the same number of columns and have compatible types (same type or similar - like int and float).

The resulting column name are the ones of the top query.

Since these are set operators, by default there will be no duplicates.
If we wish to keep duplicates, we have to add ALL - e.g. `UNION ALL`.
However, virtually no real-life systems support `EXCEPT ALL` and `INTERSECT ALL`.

When ordering the result of a query with these operators, the `ORDER BY` should appear only once, at the end of the bottom query.
### Intersect
```PostgreSQL
SELECT sid
FROM Reserves R, Boats B
WHERE R.bid = B.bid AND color = 'red'
INTERSECT
SELECT sid
FROM Reserves R, Boats B
WHERE R.bid = B.bid AND color = 'green';
```
This will return all `sid` of sailors that reserved both red boats and green boats.
Equivalently (but much less naturally) we can write
```PostgreSQL
SELECT R1.sid
FROM Reserves R1, Reserves R2
	Boats B1, Boats B2
WHERE R1.sid = R2.sid AND
	R1.bid = B1.bid AND
	B1.color = 'red' AND
	R2.bid = B2.bid AND
	B2.color = 'green';
```
— wow what a mess
### Except
```PostgreSQL
SELECT sid
FROM Sailors
EXCEPT
SELECT sid
FROM Reserves
WHERE bid = 103;
```
removes those that have reserved boat 103.
Note that this is not the same as
```PostgreSQL
SELECT sid
FROM Reserves
WHERE bid != 103;
```
which returns those that have ordered a boat which is not 103.
### Union
```PostgreSQL
SELECT DISTINCT sname
FROM Sailors
UNION ALL
SELECT DISTINCT sname
FROM Sailors
```
The `DISTINCT` will ensure each of the `SELECT` returns one copy of each `sname`, and then `UNION ALL` stacks them together (remember `ALL` allows duplicates) and so we simply get two copies of each `sname` in `Sailors`.

## Nested Queries
WHERE clause is evaluated for each tuple in the cartesian product formed by the FROM clause.
The subquery is used by the WHERE to define the Boolean answer for each tuple.
```PostgreSQL
SELECT ...
FROM ...
WHERE ... (Query)...;
```
### Nesting Queries with IN
We can do something like
```PostgreSQL
WHERE ... C [NOT] IN (Query)...;
```
for a query that returns a single column, and C being a value or attribute we look for in that column.
For example
```PostgreSQL
SELECT S.name
FROM Sailors S
WHERE S.sid NOT IN (
	SELECT R.sid
	FROM Reserves R
	WHERE R.bid = 103
);
```
Returns all the names of sailors who did not reserve boat 103.

### Nesting Queries with ANY/ALL
```PostgreSQL
SELECT ...
FROM ...
WHERE ... C {Arithmetic Comparison} [ANY/ALL] (Query)...;
```
where again the query returns a single column, C is a value or attribute.
For example
```PostgreSQL
SELECT *
FROM Sailors S1
WHERE S1.age > ANY (SELECT S2.age
					FROM Sailors S2);
```
returns the sailors who are not the youngest, while
```PostgreSQL
SELECT *
FROM Sailors S1
WHERE S1.age >= ALL (SELECT S2.age
					FROM Sailors S2);
```
returns the sailors who are the oldest.
### Nesting Queries with EXISTS
```PostgreSQL
SELECT ...
FROM ...
WHERE ... [NOT] EXISTS (Query) ...;
```
`EXISTS` returns TRUE if the query result is not empty
`NOT EXISTS` returns TRUE if the query result is empty
For example
```PostgreSQL
SELECT S.sname
FROM Sailors S
WHERE EXISTS (
	SELECT * FROM Reserves R
	WHERE R.bid = 103
		AND S.sid = R.sid
)
```
returns all sailor names that have reserved boat 103.
— Note that the `S` in the subquery refers to the sailor from the **outer** query.
### Nesting Queries in the FROM
We use the query to create a temporary table, which **must be give an alias**.
For example
```PostgreSQL
SELECT sname
FROM Sailors S,
	(SELECT sid
	FROM Sailors
	EXCEPT
	SELECT sid
	FROM Reserves
	WHERE bid=103) T
WHERE S.sid = T.sid;
```
this query returns the names of all sailors that didn't reserve boat 103.

## Division
We don't have a shorthand for [[Relational Algebra#Division|division]], so we memorize an equivalent scheme.
Let's say we want to get all sailors who reserved all boats.
So $\pi_{sid,bid}(R)\div\pi_{bid}(B)$.
```PostgreSQL
SELECT S.sid
FROM Sailors S
WHERE NOT EXISTS (
	(SELECT B.bid FROM Boats B)
	EXCEPT
	(SELECT R.bid FROM Reserves R
	WHERE R.sid = S.sid)
);
```
Step by step:
1) the first part of the EXCEPT give us all boats
2) we then subtract all boats the current sailor has reserved
3) if this list is now empty, the sailor has reserved all boats, so include him


# Aggregate Functions
## Types
### COUNT
`COUNT(*)` counts the number of lines.
`COUNT(A)` for some attribute `A`, counts how many values are in `A`.
`COUNT(DISTINCT A)` counts how many distinct values are in `A`.
==The difference between `COUNT(*)` and `COUNT(A)` is that `COUNT(A)` ignores `null` values.==
### SUM
`SUM(A)` sum of all values of attribute `A`.
`SUM(DISTINCT A)` is the sum of all distinct values of `A`.
### AVG, MIN, MAX
Likewise. (No `[DISTINCT]` in `MIN` and `MAX` of course)
`MIN` and `MAX` can work not only over numbers.
### Examples
```PostgreSQL
SELECT AVG(S.age)
FROM Sailors S
WHERE S.rating=10;
```
Returns the average age of sailors with rating 10.
```PostgreSQL
SELECT COUNT(DISTINCT color)
FROM Boats;
```
Returns the number of colors in `Boats`.

A common mistake might be
```PostgreSQL
SELECT sname, MAX(age)
FROM Sailors;
```
This might seem like a way to acquire the name and age of the oldest sailor, but it is actually illegal. `MAX` returns an aggregate value over all rows, while `sname` is just an attribute which is not necessarily common to all rows.
In order to fix that, we can do the following:
```PostgreSQL
SELECT S.sname, S.age
FROM Sailors S
WHERE S.age = (SELECT MAX(S2.age) FROM Sailors S2);
```
## GROUP BY
An interesting key word in SQL is `GROUP BY`.
It allows dividing rows by groups, and then applying aggregate functions on said groups.
For example:

| <u>sid</u> | sname   | rating | age  |
| ---------- | ------- | ------ | ---- |
| 22         | Alice   | 7      | 45   |
| 31         | Barbara | 8      | 55.5 |
| 58         | Carol   | 10     | 35   |
| 70         | Alice   | 10     | 25   |
```PostgreSQL
SELECT rating, AVG(age)
FROM Sailors
GROUP BY rating;
```
will return

| rating | avg  |
| ------ | ---- |
| 7      | 45   |
| 8      | 55.5 |
| 10     | 30   |

Note that `NULL` values are considered a separate group.

It must be the case that all attributes in `SELECT`, that are not within an aggregate function, must appear in `GROUP BY`.
For example, it is illegal to do the following:
```PostgreSQL
SELECT sid, AVG(age)
FROM Sailors
GROUP BY rating;
```
as `sid` is not necessarily a common attribute in the group. ^groupby-error
## HAVING
`WHERE` was enforcing Boolean conditions on rows. `HAVING` is for groups.
Suppose we have the table `Sailors`:

| <u>sid</u> | sname   | rating | age |
| ---------- | ------- | ------ | --- |
| 22         | Alice   | 7      | 45  |
| 31         | Barbara | 8      | 37  |
| 58         | Carol   | 10     | 35  |
| 70         | Alice   | 10     | 25  |
| 54         | Danna   | 7      | 55  |
| 99         | Erica   | 10     | 15  |
```PostgreSQL
SELECT rating, AVG(age)
FROM Sailors
WHERE age < 50
GROUP BY rating
HAVING COUNT(*) > 1;
```
Will calculate the average age of groups of sailors, aged less than 50, with the same rating; and it will only consider groups with more than 1 rows.
Resulting in:

| rating | avg |
| ------ | --- |
| 10     | 25  |

Similar to the problem in [[#^groupby-error|GROUP BY]], the condition in `HAVING` cannot include any attributes that are not aggregates or in the `GROUP BY`. For example:
```PostgreSQL
SELECT rating, AVG(age)
FROM Sailors
GROUP BY rating
HAVING sname <> 'Alice';
```
is illegal because `sname` is not necessarily a common attribute of the group.
If instead we had `MIN(sname) <> 'Alice'` that would be fine.

## Some More Examples
```PostgreSQL
SELECT B.bid, COUNT(*)
FROM Boats B, Reserves R
WHERE R.bid=B.bid AND B.color='red'
GROUP BY B.bid;
```
will return the number of reservations for each red boat.

```PostgreSQL
SELECT bname
FROM Boats B, Reserves R
WHERE R.bid=B.bid
GROUP BY bid, bname
HAVING COUNT(DISTINCT day) <= 5;
```
will return the names of the boats that have been reserved for at most 5 days.

Notice this failed attempt to get the color which has the largest number of boats colored in it:
```PostgreSQL
SELECT color
FROM Boats
GROUP BY color
HAVING MAX(COUNT(bid));
```
So a couple of problems:
1. `MAX` is an aggregate function expecting a plethora of values from which to return a single value. But so is `COUNT`. So `MAX` has no meaning being applied on a single value. **This is the reason aggregate functions cannot be applied on aggregate functions.**
2. `HAVING` is supposed to be a Boolean value determining whether a group should be included or not, not a number...

The fix:
```PostgreSQL
SELECT color
FROM Boats B
GROUP BY color
HAVING COUNT(bid) >= ALL(
	SELECT COUNT(bid)
	FROM Boats
	GROUP BY color
);
```

Another example with a =={lightblue}sub-query in `SELECT`==:
```PostgreSQL
SELECT S.sid, S.age, (SELECT MAX(S2.age)
					FROM Sailors S2
					WHERE S2.age < S.age)
FROM Sailors S;
```
will return for each sailor, the `sid`, `age` and the max age which is less than that sailor's age. Note that the returned table from the sub-query must be a single value.

## Division Using Aggregation
Suppose we want to find the names of sailors who reserved all boats.
```PostgreSQL
SELECT sname
FROM Sailors NATURAL JOIN Reserves
GROUP BY sname, sid
HAVING COUNT(DISTINCT bid) = (SELECT COUNT(*) FROM Boats);
```
Note that this works for division as long as our tables are **not empty**, otherwise we can get slightly different results.

# Modification Of Tables
## Insert Into
Suppose we have a table
~={blue}Sailors(<u>sid</u>, sname, rating, age)=~
```PostgreSQL
INSERT INTO Sailors (sid, rating)
VALUES (123, 9);
```
will insert a new row to Sailors with NULL values in the missing columns.
Note that the <u>key</u> must always be included.

### Inserting Using Select
We can also write an insert statement adding all `trackids` from tracks with `genreid` 16, for `playlistid` 20 into the table `playlists_tracks` [Quiz: Modifications - Week 4](https://moodle.huji.ac.il/2024-25/mod/quiz/view.php?id=38841)
```PostgreSQL
INSERT INTO playlist_track
SELECT 20, trackid FROM tracks WHERE genreid = 16;
```

## Delete From
Suppose again
~={blue}Sailors(<u>sid</u>, sname, rating, age)=~
```PostgreSQL
DELETE FROM Sailors
WHERE sid NOT IN (SELECT sid FROM Reserves);
```
will do what you think it does. The condition in the `WHERE` can be as complex as we are used to having it.
In addition, `DELETE FROM Sailors;` with no condition will delete all rows.

## Update
~={blue}Sailors(<u>sid</u>, sname, rating, age)=~
```PostgreSQL
UPDATE Sailors
SET rating = rating - 1
WHERE sid NOT IN (SELECT sid FROM Reserves);
```
will reduce one rating point from sailors who have not reserved boats.

# Views
A SQL macro
```PostgreSQL
CREATE VIEW <view-name>
AS <query>;
```
Creates a shorthand for queries.
This is useful for creating readable SQL code, as well securing information when giving access to data. 

## Updating Views
We can also **update** views if they are *updatable*.
The update cascades down to the underlying table, so only the real table is updated.
"Updates" meaning `INSERT INTO`, `UPDATE`, `DELETE FROM` of course.
### Conditions For Updating:
**Condition 1:**
The defining query must have a single table in the `FROM`.

**Condition 2:**
The defining query must not contain any of the following clauses at its **top level**:
1. `GROUP BY`
2. `HAVING`
3. `DISTINCT`
4. `UNION`
5. `INTERSECT`
6. `EXCEPT`

**Condition 3:**
The selection list must not contain any aggregate functions.
### What Is Updated?
The underlying **real** table. For example
```PostgreSQL
CREATE VIEW GreatSailors AS
SELECT sid, sname FROM Sailors
WHERE rating >= 9;
```
Then
```PostgreSQL
DELETE FROM GreatSailors
WHERE sname = 'Alice';
```
is translated to
```PostgreSQL
DELETE FROM Sailors
WHERE sname = 'Alice' AND rating >= 9;
```

When ~={pink}inserting=~ into the view, ==the missing columns are given a `NULL` value.==

# Temporary Tables
We can create a one-use table. A table which we create and calculate instantly, and use in the following query and that's it.

## Creation
We can do this with `WITH`:
```PostgreSQL
WITH ColorCnt(color, num) AS
(SELECT color, count(bid)
FROM Boats
GROUP BY color)
SELECT color FROM ColorCnt
WHERE num = (SELECT max(num) FROM ColorCnt);
```
This is equivalent to
```PostgreSQL
SELECT color FROM Boats B
GROUP BY color
HAVING count(bid) >= ALL (SELECT count(bid)
						FROM Boats
						GROUP BY color);
```
which returns the most popular colors.

## Recursion
We can have temporary tables which refer to themselves in their definition.
The format of that is as such:
```PostgreSQL
WITH RECURSIVE TableName AS
(SELECT ... - non-recursive term
UNION [ALL]
SELECT ... - recursive term)
... - query;
```

`UNION ALL` allows for duplicates, and `UNION` does not.
In fact, the difference is more significant than it seems at first, as we will see.

For example:
```PostgreSQL
WITH RECURSIVE t(n) AS (
	VALUES(1)
	UNION ALL
	SELECT n+1 FROM t WHERE n < 5
)
SELECT sum(n) FROM t;
```
This will result in $1+2+3+4+5=15$.
Look at "How Recursion Works" and come back here to see if you understand ^recursion-example

### How Recursion Works
We will add the notes for the case of `UNION`, i.e. ++without++ duplicates, in ~={blue}**blue**=~.

The recursion has two tables in the process of making
1. working table
2. result table

The **result table** will contain all calculated rows ~={blue}without duplicates=~, beginning with the calculated non-recursive term.
The **working table** will contain the calculated result of ++the recursive term applied on itself++~={blue}, minus **result table** duplicands=~, beginning with the result of the non-recursive term.

A step by step example can be found in the relevant lesson at timestamp 11:45 [here](https://huji.cloud.panopto.eu/Panopto/Pages/Viewer.aspx?id=ddb40c27-a2db-43e0-ae8c-ac46004fb069). ^step-by-step-recursion
#### The Recursive Steps
The recursive query is evaluated on the **working table**. 
(Which in the first round contains the result of the non-recursive term)

Then the <u>results</u> are appended to the **result table**~={blue}, removing from the <u>results</u> any duplicands in **result table**=~.

Then the **working table** will contain those same results~={blue}, minus those **same** duplicands=~, instead of what it now contains.

The cycle is repeated until the **working table** is empty.

Now go see if you understand [[#^recursion-example]].

### Infinite Loop
Naturally with recursion, we should watch out for infinite loops, that is especially the case with SQL, as an infinite loop could occur depending on the table in the query.

For example we saw the following recursive query working just fine [[#^step-by-step-recursion|here]], however we'll see it can be problematic depending on the table:
```PostgreSQL
WITH RECURSIVE Reach(U,V) AS (
SELECT * FROM Edge
UNION ALL
SELECT Reach.U, Edge.V
FROM Reach, Edge
WHERE Reach.V = Edge.U
)
SELECT * FROM Reach;
```
If we have the table `Edge` like so:

| U   | V   |
| --- | --- |
| 1   | 2   |
| 2   | 1   |
You can check and see that this leads to an infinite loop, because the working table will contain (1,2,2,1) again.

However, remember that if we use `UNION` instead of `UNION ALL`, we drop from the result of the recursive term, any duplicands of the **result table**, from both the **result table and working table.**

So if we do that, when reaching the (1,2,2,1) again, since they already exist in the **result table**, we will drop them and the working table will be empty which ends the loop.