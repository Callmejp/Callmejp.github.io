---
layout:     post
title:      "关于近期对抗样本Paper的综述"
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
1. 特征函数：We define a feature to be a function mapping from the input space $X$ to real numbers, with the
set of all features thus being $F = \{f : X \rightarrow R\}$.
2. $\rho -useful$ features: For a given distribution $D$, we call a feature $f \rho -useful (ρ > 0)$ if it is correlated with the true label in expectation, that is if
  $$
  E_{(x, y) \sim D}[y \cdot f(x)] \ge \rho.
  $$
  * 鲁棒特征：
  $$
  E_{(x, y) \sim D}[\mathop{}_{\delta \in \triangle(x)}^{inf} y \cdot f(x + \delta)] \ge \gamma.
  $$
  * 非鲁棒特征： A useful, non-robust feature is a feature which is $\rho -useful$ for some $\rho$ bounded away from zero, but is not a $\gamma-robust$ feature for any $\gamma \ge 0$.
3. A classifier $C = (F, w, b$) is comprised of a set of features
$F$, a weight vector $w$, and a scalar bias $b$:
    $$
    C(x) = sgn(b + \mathop{}_{f \in F}^{\sum} w_f \cdot f(x)).
    $$

至此，为了证明鲁棒性特征与非鲁棒性特征的存在，作者设计了四类不同的训练方式：
1. 鲁棒数据集，标准训练方式
2. 鲁棒数据集，对抗训练方式
3. 非鲁棒数据集，标准训练方式
4. 非鲁棒数据集，对抗训练方式

那么只剩下一个问题，如何抽出鲁棒特征与非鲁棒特征从而形成两个数据集。[这篇解读](https://baijiahao.baidu.com/s?id=1633163994368960188&wfr=spider&for=pc)加上自己理解总结如下：两个数据集都是由同一个原数据集产生的。为了总结出鲁棒特征，作者首先需要准备一个鲁棒的模型，然后以类似`GAN`的方式让准备生成的鲁棒图片和原图片在倒数第二层的输出尽可能相近（因为最后一层是线性的）。我感觉这里不理解也没啥关系，因为关键就是如下的训练结果了。

![四类训练结果](/img/2020-6-8/result.jpg)

这样的效果还是很有说服力的。作者为了证明这绝非偶然，准备了只有非鲁棒特征的数据集（好像是改标签的方式：标签一改肯定习得不出来人类所认为的鲁棒特征）来训练`DNN`，发现在测试集的表现上足够好，这就证明说，像素层面的`feature`同样是有一定的泛化性的。

另外，作者从这个论点出发，尝试解释了对抗样本的可转移性：如果一个网络架构的准确率越高，就越有可能习得非鲁棒的`feature`（见下图）。

![四类训练结果](/img/2020-6-8/transfer.JPG)

***

那么，从这个本质出发，我觉得提升鲁棒性无非就是在传统的训练过程中的每个环节花一点额外的心思：

### 数据集

[3]也是最近的一篇文章，提出了高斯补丁的方法。

![四类训练结果](/img/2020-6-8/patch.JPG)

从上图中可以很直观地理解实现方式（$W$控制补丁大小，$\sigma$控制扰动大小）。其实加噪声的方法通过以上的论述可以很好地解释为在不经意间“毁坏”了像素层面的`feature`致使`DNN`去寻找其它`feature`从而提升了鲁棒性。

### 训练方法

[4]是发表在`NIPS 2019`上的文章。虽然只有两位作者，但我觉得质量很高。

作者在其中提出了鲁棒性训练的问题：当前的鲁棒性训练方法往往只对一种攻击方式有效（这里解释下攻击方式有哪些：1范数，2范数，无穷范数，旋转以及缩放等）。并且作者枚举且“证明”了一些互相排斥的扰动（证明看不懂）：
1. Small $l_{\infty}$ 和 $l_1$ Perturbations are Mutually Exclusive
2. Small $l_{\infty}$ and Spatial Perturbations are (nearly) Mutually Exclusive

因此，作者提出一种新的训练方式(其实有些显然>-<):
$$
\hat{R}_{adv}(f,S) = \sum_{1}^{m} \mathop{}_{r \in S}^{max}L(f(x^{(i)} + r), y^{(i)})
$$

其中对于$S$的解释：

>We assume $n$ perturbation types, each characterized by a set $S$ of allowed perturbations for an input
$x$. The set $S$ can be an $l_p$-ball  or capture other perceptually small transforms such as image rotations and translations. For a perturbation $r \in S$, an adversarial example is $\hat{x} = x + r$(this is pixel-wise addition for $l_p$ perturbations, but can be a more complex operation, e.g., for rotations).

实验结果是很棒的：虽然每一项的抗扰动值都低于最优的一种训练方法，但其它方面都是遥遥领先，做到了“全面鲁棒”。

![多扰动训练结果](/img/2020-6-8/multi.JPG)

那么，对于以上的多扰动问题我也尝试`YY`一下：可能在像素层面上有不少的非鲁棒`feature`，而只针对单范数进行训练并不能保证`DNN`会迁移到鲁棒特征上，可能还有其它范数层面上的非鲁棒`feature`。

***

[5]尝试将可解释性与鲁棒性联系在一起，其训练目标如下：

![可解释训练方法](/img/2020-6-8/link.JPG)


其中的$I(x)$是预先计算的关于雅可比矩阵的函数。没看懂，我蒙一下：$I(x)$映射到输入层对应的是鲁棒特征所在的像素部分，比如耳朵，尾巴等；而最右边的雅可比矩阵是当前训练所得到的，它应该和$I(x)$越近越好，这样子就表明当前的导数更新是由可解释的像素部分所决定的。但是好像实验结果并不是很惊艳。

***

以上都是2019年左右的论文，比较新。但[4]和[5]都引用了一篇MIT在2017的Paper，引用数有点咂舌，同时也有干货，所以也放到一起讲讲[6]。


大叔多

1. Ilyas A, Santurkar S, Tsipras D, et al. Adversarial examples are not bugs, they are features[C]//Advances in Neural Information Processing Systems. 2019: 125-136.
2. Szegedy C, Zaremba W, Sutskever I, et al. Intriguing properties of neural networks[J]. arXiv preprint arXiv:1312.6199, 2013.
3. Lopes R G, Yin D, Poole B, et al. Improving robustness without sacrificing accuracy with patch gaussian augmentation[J]. arXiv preprint arXiv:1906.02611, 2019.
4. Tramèr F, Boneh D. Adversarial training and robustness for multiple perturbations[C]//Advances in Neural Information Processing Systems. 2019: 5858-5868.
5. Noack A, Ahern I, Dou D, et al. Does Interpretability of Neural Networks Imply Adversarial Robustness?[J]. arXiv preprint arXiv:1912.03430, 2019.
6. Madry A, Makelov A, Schmidt L, et al. Towards deep learning models resistant to adversarial attacks[J]. arXiv preprint arXiv:1706.06083, 2017.