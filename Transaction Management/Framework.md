1st note of *Transaction Management*

---
We ask ourselves what could go wrong with applications working with a database.
A few things are
- ~={purple}**partial execution**=~: Failure in the middle of the program
- **bug in the program**: Which can introduce errors
- ~={purple}**concurrently run programs**=~: They can interfere with one another and produce unexpected results
- **system failure**: Can lead to loss of changes

We'll discuss the ones in purple.
# Motivation
Suppose we have
```PostgreSQL
CREATE TABLE Accounts(
	pid int,
	amount int CHECK (amount >= 0)
);
INSERT INTO Accounts VALUES(1, 200), (2, 50);
```
So we have accounts ~={purple}1=~ and ~={purple}2=~ with ~={green}200=~ and ~={green}50=~ NIS.
Now say we want to transfer 100 NIS from account ~={purple}2=~ to account ~={purple}1=~.
If we do the following:
```PostgreSQL
UPDATE Accounts
SET amount = amount + 100
WHERE pid = 1;

UPDATE Accounts
SET amount = amount - 100
WHERE pid = 2;
```
We get an error for the second one only and end up with

| pid           | amount         |
| ------------- | -------------- |
| ~={purple}1=~ | ~={green}300=~ |
| ~={purple}2=~ | ~={green}50=~  |
We want the two operations =={nigga}to not be isolated==, but such that if one fails the other does too.
# Transactions
A **transaction** is a single unit of logic or work, sometimes made up of multiple operations.
We want to provide reliable units of work that allow recovery from failures, and we want to provide isolation between different programs.
## Defining A Transaction
Either
```PostgreSQL
BEGIN TRANSACTION;
... - operations
COMMIT/END TRANSACTION;
```
which will perform the operations and save them =={lightblue}unless there is failure==.
or
```PostgreSQL
BEGIN TRANSACTION;
... - operations
ROLLBACK;
```
which will rollback changes in case of failure or =={lightblue}something that is unexpected to us==.
So for example
```PostgreSQL
BEGIN TRANSACTION;
UPDATE Accounts
SET amount = amount + 100
WHERE pid = 1;

UPDATE Accounts
SET amount = amount - 100
WHERE pid = 2;
END TRANSACTION;
```
This will prevent the =={nigga}previous issue==.
# Concurrent Transactions
Problems that may occur.
## Dirty Write
When a transaction writes to data that was written by an uncommitted transaction.
For example
- Alice wrote to $x$
- ~={blue}Bob wrote to $x$=~
- Alice committed her transaction
## Dirty Read
When a transaction reads data that was written by an uncommitted transaction.
For example
- Alice wrote to $x$
- ~={blue}Bob read $x$=~
- Alice read $x$
## Nonrepeatable Read
When a transaction rereads data that it has previously read, and discovers that it has been modified by another transaction.
For example
- Alice read $x$
- Bob wrote to $x$
- ~={blue}Alice read $x$=~
## Phantom Read
When a transaction re-executes a query, and gets a different result (e.g. extra rows).
## Serialization Anomaly
When transactions perform something concurrently, and the result of that is inconsistent with at least one variation of running them one at a time.

For example suppose a shift needs at least one doctor and both Alice and Bob are registered.
- Alice sees Bob is also registered
- Bob sees Alice is also registered
- Alice unregisters
- Bob unregisters
- =={red}No doctors==
# Isolation
Use a system to make it such that multiple transactions can work simultaneously without interfering with one another.
## Isolation Levels
There are 4 isolation levels (from least to most isolated) which define how isolated it is.
- Read uncommitted
- Read committed
- Repeatable read
- Serializable

![[Pasted image 20250618183029.png]]
The isolation level is defined in the definition of the transaction. Can also define default isolation level for all transaction.