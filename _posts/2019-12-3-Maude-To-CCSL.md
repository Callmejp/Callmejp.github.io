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
1. sth
   
## But

When we search the key words(`Maude` and `CCSL`). To our surprise, there is one paper which have implemented it.

![Authors of the paper](img/2019-12-3/grass.jpg)

## Implemention

### Abstract

1. sth
2. sth

#### Rewrite the rules in CCSL

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

