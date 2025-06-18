[[DB/Transaction Management/Framework#Transactions|Transactions]]
# Definition
A **schedule** of a set of transactions $T_{1},\dots,T_{n}$ is an ordering of the actions in all of $T_{1},\dots,T_{n}$ that is consistent with each transaction (meaning the actions of $T_{i}$ are in the correct order).

If all transactions in the schedule have commit or abort actions at the end, the schedule is **complete**.

If the transactions are not interleaved (meaning overlapping in execution), the schedule is **serial**.
# Serializable Schedules
A schedule of transactions is **serializable** if its effect is guaranteed to be like running some serial schedule of these transactions.
# Recoverable Schedules
Meaning transactions can abort safely and truly undoing the changes.
Not always the case, for example

| T1    | T2     |
| ----- | ------ |
| R(A)  |        |
| W(A)  |        |
|       | R(A)   |
|       | W(A)   |
|       | R(B)   |
|       | W(B)   |
|       | Commit |
| Abort |        |
The W(A) of T1 has influenced T2's actions which has been committed.
## Condition
A schedule $S$ is **recoverable** if the transactions commit ==only after all transaction, whose changes they read, commit==.
## Cascading Aborts
Now if one "dependency" aborts, the transaction can still abort its operation and so on in a cascade of aborts.
## Avoiding Cascading Aborts
It's better if the transaction doesn't have to unexpectedly abort halfway.

A schedule $S$ **avoids cascading aborts** if the transactions ==only read changes of committed transactions==.
# Recognizing Serializable Schedules
[See here](https://huji.cloud.panopto.eu/Panopto/Pages/Viewer.aspx?id=4f1e832c-0287-4047-939b-ae9700c7414a) the lecture about **conflict serializability** and **how we can recognize a serializable schedule**.

We can know a schedule is serializable if it is **conflict equivalent** to a serial schedule.
Meaning if we can ==draw a graph== where the transactions $T_{1},\dots,T_{n}$ are the nodes,
and there is an edge $T_{i}\to T_{j}$ if there is a conflict (RW, WW, WR) between them, and $T_{i}$ comes first.
This graph is called =={green}the precedence graph==.