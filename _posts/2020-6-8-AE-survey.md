---
layout:     post
title:      "近期关于对抗样本Paper的综述"
subtitle:   "重新认识Adversarial Example"
date:       2020-06-08
author:     "JohnReese"
header-img: "img/2020-6-8/stone.jpg"
mathjax: True
tags:
    - ML
---

最近MIT在对抗样本领域发了一篇`Paper`，从主要内容和自己的体会来看，有点“半里程碑”的味道了。

所以以这篇`Paper`为基点，尝试总结一些对抗样本研究的趋势并探究其背后的本质。

论文名是《Adversarial Examples are not Bugs, they are Features》，发表在`NIPS 2019`上[1]。

深度学习在图片分类领域火了好久了，但自Szegedy C等人[2]在2013年发现对抗样本后，这个问题也同样困扰了`Researchers`好久了。

刚开始的时候，我记得Ian Goodfellow认为这是由于DNN在高维空间中表现出的线性性质致使微小扰动经过DNN计算后放大致使输出产生较大的偏差。但是这样的解释总感觉有些不够，而Andrew Ilyas等人[1]就在他们的论文中尝试研究其真正的本质。

从论文的题目中你可以直接体会到作者的观点：对抗样本不是缺陷。

尝试考虑这个过程：我们的目的是训练一个图片分类器；我们选择`DNN`是因为其强大的函数拟合能力；我们选择`Empirical risk minimization`(ERM)作为结果的衡量标准；我们为其提供庞大的数据集；最后，`DNN`反馈给我们不俗的准确率。可我们不满足，我们还要求`DNN`表现出人类所谓的鲁棒性。但问题是，如上的过程中，我们做了哪一步致使我们可以抱有这样的预期呢？是因为我们人类默认了只能从轮胎、耳朵、外形等鲁棒的性质来区分狗和卡车。但说不定，是我们无知了呢？`DNN`发现了可以用于分类的，但我们人类无法发现的，像素层面上的`feature`。

> Adversarial vulnerability is a direct result of sensitivity to well-generalizing features in the data.

如上，是作者对于对抗性质的一个总结。——问题的根源是数据集。虽然我们不停念叨`Big Data`，但数据集到底需要多大才能避免`DNN`习得易于泛化，但其实不鲁棒的性质的问题应该目前是没有一个标准答案的。

接着来看看作者设计了怎样的实验来论证自己的观点吧：
1. We define a feature to be a function mapping from the input space X to real numbers, with the
set of all features thus being $F = {f : X \rightarrow R}$.




1. Ilyas A, Santurkar S, Tsipras D, et al. Adversarial examples are not bugs, they are features[C]//Advances in Neural Information Processing Systems. 2019: 125-136.
2. Szegedy C, Zaremba W, Sutskever I, et al. Intriguing properties of neural networks[J]. arXiv preprint arXiv:1312.6199, 2013.