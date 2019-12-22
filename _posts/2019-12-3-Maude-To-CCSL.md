---
layout:     post
title:      "基于Maude验证CCSL约束的可行性"
subtitle:   "论文的复现与拓展"
date:       2019-12-21
author:     "JohnReese"
header-img: "img/2019-12-3/grass.jpg"
mathjax: True
tags:
    - 形式化
---

# Background

## Maude
1. Rewrite: `op` and `rl`.
2. Search: `search` and `LTL`.


## Clock Constraint Specification Language(CCSL)
1. Clock: tick idle tick idle ...(infinite).
2. Constraint: Clock A must `tick` after Clock B `tick`.
3. Scheduling: decide each clock when to tick.
   
# But

When we search the key words(`Maude` and `CCSL`) in Google. To our surprise, there is one paper which have implemented it.

![Authors of the paper](/img/2019-12-3/authors.JPG)

`Soooo! Creation => Reproduction`

# Implemention

## Final Results

Github: [https://github.com/Callmejp/CCSL2Maude](https://github.com/Callmejp/CCSL2Maude)

Some Statistics:

$$
111 / 233 \thickapprox 50.3\% \\
fmod: CLOCK , TICK-LIST, MAIN\\
Constraints: Subclock, Supremum, Causality, Exclusion, Union, Intersection \\
Other Function: verify.maude
$$

## Comprehension To CCSL

1. Formal Definition:
   * `definition`:
  
   $$
   Clock: (c_n)_{n \in N^+}.\\
   Clock Sets: C = \{c_1, c_2, ..., c_n\}.\\
   Schedule: \delta: N^+ \rightarrow 2^C.\\
   History: \chi: C \times N^+ \rightarrow N.
   $$

   * `example`:

   $$
   C = \{c_1, c_2\}.\\
   c_1 = tick, tick.\\
   c_2 = idle, tick.\\
   \delta(1) = \{c_1\}, \delta(2) = \{c_1, c_2\}.\\ 
   \chi(c_1, 1) = 1, \chi(c_1, 2) = 2.\\
   \chi(c_2, 1) = 0, \chi(c_2, 2) = 1.\\
   $$

2. Constraints:
   
   ![Constraints](/img/2019-12-3/definitions.JPG)

## Generate the expected CCSL sequence

1. Key Code Segment:
   ```m
   crl [next] : < (F, F') ; PHI ; X ; K > => < (F, F'); PHI ; update(X, F) ; K + 1 >
     if F =/= empty /\ satisfy(F, X, PHI) /\ .
   
   ***(
     F is the subset of Clocks that will tick at the time step (K + 1) .
     And the precondition is the schedule will satisfy all constraints(i.e. PHI).
   )
   ```
   
According to our understanding, we think the `formalization of a clock system` doesn't express the `if and only if`(i.e. $\Leftrightarrow$).

For example, as the figure below shows: if there is a constraint $c_1 \triangleq c_2 + c_3$, then condition $c1 \notin F \lor c_2 \in F \lor c_3 \subset F$ can satisfy it.

![Union](/img/2019-12-3/union.jpg)

But, it's clear that $c_1 \in \chi(n) \Leftrightarrow c_2 \in \chi(n) \land c_3 \in \chi(n)$ express the ` bidirectional relationship`.

So, state $c_1 \notin \chi(n) \land c_2 \in \chi(n) \land c_3 \notin \chi(n)$ apparently satisfy the `formalization of a clock system`, but it can't satisfy the original defination of `Union Constraint`.

As a result, we abstract the constraints based on the original definitions and our understanding.

You can check the `TestCaseForConstraints.md` in the repository.

## Periodic scheduling

There is a `paradox` between the `finite` sequence we can generated and the formal verication to the `infinite` sequence.

So, as the figure below shows, periodic scheduling can solve the problem to some extent.

![Periodic scheduling](/img/2019-12-3/period.jpg)

`In short, all clocks have the same cycle.`

```m
eq isPeriodic(([C1, TL1, N1],[C2, TL2, N2], X), (C1 < C2) PHI, K, K’) = 
  tick(TL1,K) == tick(TL1,K’) and
  tick(TL2,K) == tick(TL2,K’) and
  sd(N1,num(TL1,K)) >= sd(N2,num(TL2,K)) and
  isPeriodic(X,PHI,K,K’) .

***(
    C*: Clock's name
    TL*: Tick-List(tick, idle, ...)
    N*: Nat
    PHI: Constraints
    K: start of the cycle
    K': end of the cycle
)
```
We thick the code above written in the paper has a logical error. As you can see, `isPeriodic(X,PHI,K,K’)` will remove the `C1 & C2` in the `X`. But if there're other constraints that include 'C1 & C2', it will apprently cause errors when running programs.

You can also check our implemention and test in our repository.

## Formal verification by (bounded) model checking

The content is mainly about the detection of `Deadlock` via `search` in `Maude` and `LTL verification` via `Model-checker` in `Maude`.

We have to admire the authors' profound understanding and application of Maude. As you can see in the code segment, `bounded` and `periodic` schedule can both happen at the same time.

```m
crl [next] : < (F, F') ; PHI ; X ; K > => < (F, F'); PHI ; X' ; K + 1 >
    if F =/= empty /\
	   satisfy(F, X, PHI) /\
	   X' := update(X, F) /\
	   getPeriod(X', PHI, K, K + 1) == 0 .

crl [periodic] : < (F, F') ; PHI ; X ; K > => < (F, F') ; PHI ; rollback(X', P) ; sd(K + 1, P) >
    if F =/= empty /\
	   satisfy(F, X, PHI) /\
	   X' := update(X, F) /\
	   P := getPeriod(X', PHI, K, K + 1) .
```

By the way, `rollback` is done by ourselves. 

```m
  --- Code Segment in fmod ConFIGURATION
  var X : Conf .
  vars K N1 : Nat .
  var Y : ConfElt .
  var C1 : Clock .
  var TL1 : TickList .
  
  op rb : ConfElt Nat -> ConfElt .
  eq rb([C1, TL1, N1], 0) = [C1, TL1, N1] .
  eq rb([C1, (TL1, 1), N1], K) = rb([C1, TL1, N1 - 1], K - 1) .
  eq rb([C1, (TL1, 0), N1], K) = rb([C1, TL1, N1], K - 1) .

  op rollback : Conf Nat -> Conf .
  eq rollback(empty, K) = empty .
  eq rollback((Y, X), K) = rb(Y, K), rollback(X, K) . 
```

## Generate specific sequences

balabala

# Concluding remarks

1. `Maude`: FrameWork for Formal Verification.
2. `CCSL`: Like the `SAT/SMT` problem to some extent.
