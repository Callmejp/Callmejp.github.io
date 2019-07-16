---
layout:     post
title:      "关于强化学习的形式化验证"
subtitle:   "论文解读"
date:       2019-07-16
author:     "JohnReese"
header-img: "img/post-7-16.jpg"
mathjax: True
tags:
    - RL
---

## 引言
主要内容来自：[https://www.aaai.org/ocs/index.php/AAAI/AAAI18/paper/download/17376/16225](https://www.aaai.org/ocs/index.php/AAAI/AAAI18/paper/download/17376/16225) 
讲的是如何使用形式化方法(`Formal Methods`)来确保`RL`(`Reinforcement Learning`, 即强化学习)训练时的安全性。
起初看到内容中有很多延伸阅读的论文和一些专业名词，以为是比较艰深的一篇paper，不过静下心来后发现难度尚可。
依旧是按照论文的行文顺序进行一定的解读。

## Introduction
在引言部分就有一些值得关注的名词。`Cyber-physical systems`(简称`CPSs`)，维基译为网络实体系统，详细介绍摘入如下：
> 实体（Physical）：自然界中的或由人类制造的，遵循物理定理在连续的时间内运行的系统。</br>
> 网络（Cyber）：利用计算、通信、及控制系统进行的离散及逻辑化管理。</br>
> 网络实体系统：（Cyber-Physical System）：将实体与网络的各个组成部分在所有层面和维度上紧密结合的系统，对实体及网络进行对称性的深入管理。</br>
> CPS实质上是一种多维度的智能技术体系，以大数据、网络与海量计算为依托，通过核心的智能感知、分析、挖掘、评估、预测、优化、协同等技术手段，将计算、通信、控制有机融合与深度协作，做到涉及对象机理、环境、群体的网络空间与实体空间的深度融合。CPS能够从实体空间、环境、活动大数据的采集、存储、建模、分析、挖掘、评估、预测、优化、协同，并与对象的设计、测试和运行性能表征相结合，使网络空间与实体空间深度融合、实时交互、互相耦合、互相更新的网络空间（包括机理空间、环境空间与群体空间的结合）；进而，通过自感知、自记忆、自认知、自决策、自重构和智能支持促进工业资产的全面智能化 [13]。

最近强化学习开始被应用于关键领域中的`CPSs`，所以保证其安全性是迫在眉睫的。并且作者认为将经过验证的模型纳入基于强化学习的控制器的安全案例中非常重要，因为对于在开放环境(如自动驾驶汽车)中运行的系统，单独测试是一种难以处理的系统验证和验证方法。

作者提出了`Justified Speculative Control`(简称为`JSC`)的安全保障技术，主要内容是将可验证的实时监控与强化学习的训练相结合。详细来说就是当`JSC`没有检测到模型不准确时，系统的动作被限制为一组经过验证的控制动作。当检测到模型不准确时，系统就可以采取未经验证的、可能不安全的(不准确的)模型操作。

不过，作者也提到`JSC`也结合了前人的许多方法，比如`ModelPlex`等。

## Background
一般论文的背景阅读可能可有可无，但这篇论文几乎没有废话。

### Differential Dynamic Logic
我译为`微分动态逻辑`。作者主要通过一个行驶在直线上的汽车的一维模型作为例子给出了其简单的混合程序(`hybrid program`)。

$$
Example.1 \\
{ ( \underbrace{(a := A \cup a := 0)}_{ctrl}; \underbrace{( p^{'} := v \cup v^{'} := a)}_{plant} ) }^{*}
$$

例1描述了一辆汽车，它非确定性地选择以最大加速度`a`加速或不加速，然后遵循一个微分方程。这个过程可能会重复任意多次(由星号表示)。Because there is no evolution domain constraint on *plant*, each continuous evolution has any arbitrary non-negative duration r ∈ R。（这句话的细节不是很懂，应该是想表达没有施加多余的约束条件。但*ctrl*和*plant*的准确含义我不清楚）

#### Hybrid Program Semantics. 
混合程序的语义。这里我打不出来两个紧连的方括号所以就截图表示。表1是正规的一些语法含义。这些都会在建模的时候用到。
![img](/img/2019-7-16/image1.JPG)

![img](/img/2019-7-16/image2.JPG)

#### Differential Dynamic Logic.
每一个混合程序$\alpha$都和操作符$[\alpha]$和$<\alpha>$联系在一起，表达了程序的状态可达性。公式$[\alpha]\phi$表达的是由程序$\alpha$可到达的任何状态，公式$\phi$都是正确的。而$<\alpha>\phi$表达的是在程序$\alpha$执行了几次后，公式$\phi$是正确的。然后一般情况下微分动态逻辑的公式形式如下
1. ${\theta}_1 ~ {\theta}_2$
2. $\neg \phi | \phi \land \psi | \phi \lor \psi | \phi \rightarrow \psi$
3. $\forall x \phi | \exists x \phi$
4. $[\alpha]\phi | <\alpha>\phi$
其中${\theta}_{i}$是关于实数的数学表达式，$\phi and \psi$是公式，$\alpha$是程序，$~$意味着大于，小于等关系符号。$s \models P$表达在状态$s$下$P$为真。

这些都是基础的语法，它有助于我们了解之后实际的建模规则。

$$
Example.2(对于直线行驶汽车模型的安全性描述) \\
\underbrace{ v \geq 0 \land A > 0}_{initial condition} \rightarrow { [( \underbrace{(a := A \cup a := 0)}_{ctrl}; \underbrace{( p^{'} := v \cup v^{'} := a)}_{plant} )] }^{*} \underbrace{ v \geq 0 }_{post cond.}
$$
上述公式表达了如何汽车以非负的速度出发，它仍旧会在一段时间的状态转移后拥有非负的速度。

### ModelPlex: Verified Runtime Validation



