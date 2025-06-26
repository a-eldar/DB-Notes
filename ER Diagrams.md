# Components
**Entities** are beings that are distinguishable from one another.
**Relationships** are subsets of cartesian products of entity-sets, with descriptions.
**Properties** of entity sets and relationship sets must have a single non-null value for each entity or relationship.

# Multiplicities
Without any special lines connecting entities in relationships, there is no limit beyond the obvious.
This is a Many-to-Many relationship.
## One-to-Many
One to many relationship have an arrow pointing at the "many".
Meaning the "one" can appear once in the relationship set with one of the "many".
For example
![[Pasted image 20250323185335.png]]
![[Pasted image 20250323185354.png]]
## One-to-One
Either entities can appear at most once.
![[Pasted image 20250323185450.png]]

## In Non-binary Relationships
![[Pasted image 20250323185600.png]]
Means
$$
\Large
\begin{align}
(e_{1},\dots,e_{n},f) & \in R \\
(e_{1},\dots,e_{n},f') & \in R
\end{align}
\implies
f = f'
$$
And with many "many"
![[Pasted image 20250323185849.png]]
Then for all $1\leq i \leq m$
$$
\Large
\begin{align}
(e_{1},\dots,e_{n},f_{1},\dots,f_{i},\dots,f_{m}) & \in R \\
(e_{1},\dots,e_{n},f_{1},\dots,f_{i}',\dots,f_{m}) & \in R
\end{align}
\implies
f_{i} = f_{i}'
$$

When analyzing such diagrams, we talk about what each **arrow** ($\to$) tells us separately.

# Referential Integrity
We use a **rounded arrow** to say there must be exactly one relationship for each entity in the set.
For example here for each film there must be **exactly** one director.
![[Pasted image 20250323191901.png]]

## In Non-binary Relationships
This also works with non-binary relationships.
For example here **for each pair** of teacher and student we have exactly one course which the teacher teaches the student in.
![[Pasted image 20250323192424.png]]

## Complex Combination
![[Pasted image 20250323192745.png]]
What if we have many "normal" line, many arrows and many round arrows?
We separate exactly as before.
It means:
1. 
For all $1\leq i\leq m$
$$
\Large
\begin{align}
(e_{1},\dots,e_{n},f_{1},\dots,f_{i},\dots,f_{m},g_{1},\dots,g_{k}) & \in R \\
(e_{1},\dots,e_{n},f_{1},\dots,f_{i}',\dots,f_{m},g_{1},\dots,g_{k}) & \in R
\end{align}
\implies
f_{i} = f_{i}'
$$
2. 
For all $1\leq i\leq k$
$$
\Large
\begin{align}
(e_{1},\dots,e_{n},f_{1},\dots,f_{m},g_{1},\dots,g_{i},\dots,g_{k}) & \in R \\
(e_{1},\dots,e_{n},f_{1},\dots,f_{m},g_{1},\dots,g_{i}',\dots,g_{k}) & \in R
\end{align}
\implies
g_{i} = g_{i}'
$$
3. 
For all $1\leq i\leq k$
And for all
$$
\Large \begin{align}
(e_{1},\dots,e_{n}) & \in E_{1}\times\dots\times E_{n} \\
(f_{1},\dots,f_{m}) & \in F_{1}\times\dots\times F_{m} \\
(g_{1},\dots,g_{i-1},g_{i+1},\dots,g_{k})  & \in G_{1}\times\dots \times G_{i-1}\times G_{i+1}\times\dots\times G_{k}
\end{align}
$$

There exists $g_{i}\in G_{i}$ such that
$$
\Large(e_{1},\dots,g_{k})\in R
$$

# Inheritance
![[Pasted image 20250323221216.png]]
B and C inherit the properties of A, and in addition $B,C\subseteq A$

# Weak Entity Sets
![[Pasted image 20250323222340.png]]
Some entity sets may be uniquely identified not just using their key.
As shown in the above example, it is not enough to identify a מחלקה without its פלוגה and without its גדוד.
The identifying relationship is of type [[ER Diagrams#Referential Integrity |exactly one]].

## Identifying Using Many Other Entities
We may identify an entity using many others rather than just one.
![[Pasted image 20250323222905.png]]
So we can now also use weak entity sets to "capture" combinations of entities
(such as (actor $\times$ film) for actors playing in films)
and give that weak entity a relationship of the actor winning a prize for that film.
The weak entity would, of course, be identified using Actor and Film.
# Translating Diagrams To Relations
Translating an entity is done using its name and its attributes (with ++underline++ for the key).

For a relationship, we take its name, and its attributes with the ++keys++ of the entities in the relationship.
If the relationship has a triangular arrow, the key of the relationship would not include **one** of the keys of the triangular arrows.
If the relationship is between two entities and has a round arrow, we just include the key of the round arrow entity in the schema of the other entity.
If the relationship is between many entities and has a round arrow, we do it like in the case of the triangular arrow.

For a weak entity relationship, we add the key of the strong entity in the key of the weak entity schema.
## Translating ISA
In the case of [[#Inheritance]] we have multiple approaches.
### ER Style Conversion
We create a relation for each entity set like so:
![[Pasted image 20250626131027.png]]
Meaning an entity may appear in several relations.
### Object-Oriented Approach
We create a relation for each possible combination of entities:
![[Pasted image 20250626131417.png]]
Meaning `MoviePerson` would be someone who is not an actor nor a director, etc.
If we know, for example, that there can't be movie persons who are both actors and directors, we can drop that one.
### Null Value Approach
Create a single relation for all entities, and put `NULL` for values not applicable.
