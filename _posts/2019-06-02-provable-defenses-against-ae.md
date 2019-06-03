---
layout:     post
title:      "对于训练数据可验证的鲁棒性的训练方法"
subtitle:   "论文解读"
date:       2019-06-02
author:     "JohnReese"
header-img: "img/post-6-2.jpg"
mathjax: True
tags:
    - ML
---

## 引言
论文链接为：[https://arxiv.org/abs/1711.00851v3](https://arxiv.org/abs/1711.00851v3)  
论文主要提出了一种方法。我们还是从头开始梳理这篇论文吧。

## Introduction

作者简单地介绍了近几年的抵御对抗攻击的训练的方法被`纷纷打脸`。
>  Somewhat memorably, many of the adversarial defense papers at the most recent ICLR conference were broken prior to the review period completing (Athalye et al., 2018)

这句话的翻译如下：
> 令人印象深刻的是，最近ICLR会议上的许多对抗防御论文在审查期结束之前就被破坏了。

那么作者就引出了自己的贡献，我也觉得很厉害，所以专门写个Blog详细记录下：
1. a method for training `provably` robust deep ReLU classifiers, classifiers that are guaranteed
to be robust against `any` norm-bounded adversarial perturbations on the `training set`.
2. a provable method for detecting `any` previously unseen adversarial example, with `zero false negatives`. 当然作者也说能达到这样的结果也会导致将一些非对抗样本列为对抗样本。


## Training Provably Robust Classifiers
1. A $k$ layer feedforward ReLU-based neural network, $f_{\theta}: \mathbb{R} ^ {|x|} \rightarrow \mathbb{R} ^ {|y|} $ given by the equations:
    $$
    \hat{z}_{i+1} = W_{i}z_{i} + b_{i}, for\ i = 1,...,k-1 \\
    z_{i} = max(\hat{z}_{i}, 0) \\
    其中\ z_{1} \equiv x\  且\ f_{\theta}(x) \equiv \hat{z}_{k} \\
    使用\theta = \{W_{i}, b_{i}\}_{i=1,...,k}来表示网络中的所有参数
    $$

    这一段是基本的关于DNN的描述。
2. 他们使用集合
    $$
    Z_{\epsilon}(x) = \{f_{\theta}(x + \Delta):{||\Delta||}_{\infty} \leq \epsilon \}
    $$

    来表示对抗多边形(`adversarial polytope`)的点的集合,即图1中间那个坐标轴的图形，是非凸的。而他们方法的基础是使用外凸边界(`convex outer bound`)来包裹对抗多边形。那么只要能保证外凸边界中的点都改变不了最终DNN的输出就能够保证原来的对抗多边形的点也是如此，那么鲁棒性就得到了满足。这是比较好理解的。
    ![img](/img/2019-6-2/image1.JPG)

3. 构造外凸边界的第一步是对于ReLU函数的线性表示。
    ![img](/img/2019-6-2/image2.JPG)

    如图2所示，我们可以用$z \geq 0, z \geq \hat{z}, -u\hat{z} + (u - l)z \leq -ul.$来代替$z = max(0, \hat{z})$。其中如果$l,u$都为正数或负数，那么缩放（`relaxation`）是确定的。

4. 那么构造外凸边界有什么用呢？作者想找到其中最`坏`的点。也就是给定已知$y^{*}$标签的样本$x$，找到$Z_{\epsilon}(x)$其中最小化正确标签输出值且最大化其它标签$y^{tag}$的输出值。这可以通过解决如下的优化问题来解决：

    $$
    \underbrace{minimize}_{\hat{z}_{k}} {(\hat{z}_{k})}_{y^{*}} - {(\hat{z}_{k})}_{y^{tag}} \equiv c^{T} \hat{z}_{k} \\
    subject\ to\ \hat{z}_{k} \in \tilde{Z}_{\epsilon}(x) \\
    where\ c \equiv e_{y^{*}} - e_{y^{tag}}
    $$

    其中的$e_{y^{*}}$我觉得可以看成是一个`One-Hot`向量。作者说因为目标函数是线性的，故这是一个线性规划问题。所以

    > If we solve this LP for all target classes $y^{tag} \neq y^{*}$ and find that the objective value in all cases is positive (i.e., we cannot make the true class activation lower than the target even in the outer polytope), then we know that no norm-bounded adversarial perturbation of the input could misclassify the example.

    作者认为对于测试集也能进行上述的步骤，只是将正确的标签换成了其预测的标签。但这可能会导致非对抗样本被判断成对抗样本，不过这样可以保证所有的对抗样本都被识别了出来。大致的过程讲完以后，作者认为有两个比较严峻的问题：其一是上述的目标线性规划问题通过传统的地方不是很好处理，因为要处理输出层的所有神经元的个数`次`；其二是ReLU边界缩放的问题。`看到这里我突然发现这好像没谈处理CNN啊。。。`

## Efficient Optimization via the Dual Network
1. 作者认为为了解决第一个问题，我们应该拿出对偶问题。为什么呢？
    >  any feasible dual solution provides a guaranteed lower bound on the solution of the primal.

    翻译一下就是任何对偶解法都提供了对原问题有保障的下界。特别重要的是，作者说对偶问题的可行集合可以被表达为一个深度网络，一个和标准BP网络非常像的网络。这就意味着相关的证明可以通过对原网络进行改动后的一次反向传播实现。
