# Question 1
## A
All the keys of $R$ are
$AB,AC,AD$ as the algorithm tells us
## B
3NF - easy to check
not BCNF - $C\to B$ is not trivial nor is $C$ a super key.
## C
The decomposition is lossless because $R_{1}\cap R_{2}=AB$ is a key.
## D
$$
F_{R_{1}}=\{ AB\to C, C\to B \}
$$
$$
F_{R_{2}}= \{ AD\to ABDE, AB\to ABDE \}
$$
$$
F_{R_{1}}\cup F_{R_{2}}=\{ AB\to ABCD \}
$$
![[Decompositions Part 2#Algorithm Testing For Preservation]]
### Checking if $AD\to CE$ is preserved ✅
In $R_{2}$ $Z\cap R_{2}=AD$ is a key so after first loop $Z=ABDE$
In $R_{1}$ $Z\cap R_{1}=AB$ is a key so after second loop $Z=ABCDE$
Preserved
### Checking if $C\to B$ is preserved ✅
Obviously because in $R_{1}$
### Checking if $B\to D$ is preserved ✅
Obviously because in $R_{2}$
### Checking if $ABC\to E$ is preserved ✅
In $R_{2}$ $Z\cap R_{2}=AB$ is a key so after the first loop $Z=ABCDE$
Preserved
## E
![[Decompositions Part 2#Finding A Decomposition#Minimal Cover#Algorithm]]

The result is the original
$$
F_{R_{1}}=\{ AB\to C, C\to B \}
$$
## F
3NF because of $C\to B$
## G
$$
F_{R_{2}}= \{ AD\to ABDE, AB\to ABDE \}
$$
$$
\implies\{ AD\to B,AD\to E,AB\to D, AB\to E \}
$$
$$
\implies\{ AD\to B,AD\to E,B\to D \}
$$
## H
3NF because of $B\to D$
## I
![[Decompositions Part 2#Finding A Decomposition#Decomposition Into BCNF#Algorithm]]

$$
R=ABCDE,\quad F_{R}=\{ AD\to CE, C\to B, B\to D, ABC\to E \}
$$
Recall the BCNF conditions on a dependency $X\to Y$
1. $X$ is a superkey
2. $Y\subseteq X$
### $C\to B$ is our first violation
$R_{1}=X^+=BCD$
$F_{R_{1}} = \{ C\to B, B\to D \}$
$R_{2}=ACE$
$F_{R_{2}} = \{ AC\to E \}$
**Notice $R_{2}$ is BCNF**
### $B\to D$ is our second violation
Break $R_{1}$ into $R_{1}$ and $R_{3}$
$R_{1}=BD$
$F_{R_{1}}=\{ B\to D \}$
$R_{3}=BC$
$F_{R_{3}}= \{ C\to B \}$
As we know, ==every relation with two attributes is **BCNF**==
### In Total
$R_{1}=BD,\quad F_{R_{1}}=\{ B\to D \}$
$R_{2}=ACE,\quad F_{R_{2}}=\{ AC\to E \}$
$R_{3}=BC,\quad F_{R_{3}}=\{ C\to B \}$
