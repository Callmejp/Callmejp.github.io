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

## Background

### Maude
1. Rewrite: `op` and `rl`.
2. Search: `search` and `LTL`.


### CCSL
1. Clock: tick idle tick idle ...(infinite).
2. Constraint: Clock A must `tick` after Clock B `tick`.
3. Scheduling: decide each clock when to tick.
   
## But

When we search the key words(`Maude` and `CCSL`). To our surprise, there is one paper which have implemented it.

![Authors of the paper](/img/2019-12-3/authors.JPG)

`Soooo! Creation => Reproduction`

## Implemention

### Comprehension To CCSL

1. Formal Definition:
   * definition:
   $$
   Clock: (c_n)_{n \in N^+}.\\
   Clock Sets: C = \{c_1, c_2, ..., c_n\}.\\
   Schedule: \delta: N^+ \rightarrow 2^C.\\
   History: \chi: C \times N^+ \rightarrow N.
   $$
   * example:
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

### Generate the expected CCSL sequence

1. Key Code Segment:
   ```
   crl [next] : < (F, F') ; PHI ; X ; K > => < (F, F'); PHI ; update(X, F) ; K + 1 >
     if F =/= empty /\ satisfy(F, X, PHI) /\ .
   
   ***(
     F is the subset of Clocks that will tick at the time step (K + 1) .
     And the precondition is the schedule will satisfy all constraints(i.e. PHI).
   )
   ```
   
According to our understanding, we think the 
1. $$
   \subseteq(a, b, cnt) = \subseteq(a, b, cnt+1) \\
   if get(a, cnt) == 0 || (get(a, cnt) == 1 \And\And get(b, cnt) == 1)
   $$
2. $$
   +(d, a, b, cnt) = +(d, a, b, cnt+1) \\
   if get(d, cnt) == 0 || (get(d, cnt) == 1 \And\And (get(a, cnt) == 1 || get(b, cnt) == 1))
   $$

## Example

### Sequence

1. a: 0, 1, 0, 1, 0, 1, 0, 0, 1.
2. b: 1, 1, 0, 1, 0, 1, 1, 1, 1.
3. c: 1, 0, 1, 0, 1, 0, 1, 1, 0.
4. d: 1, 1, 0, 1, 0, 1, 1, 1, 1.
   
### CCSL

1. $\sigma \vDash a \subseteq b$
2. $\sigma \vDash d \triangleq a+b$
3. $\sigma \vDash c \sharp a$ 

