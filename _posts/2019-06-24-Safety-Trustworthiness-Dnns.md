---
layout:     post
title:      "关于DNN可信方面的综述"
subtitle:   "论文解读"
date:       2019-06-24
author:     "JohnReese"
header-img: "img/post-6-24.jpg"
mathjax: True
tags:
    - ML
---

## 引言
主要内容来自：[https://arxiv.org/abs/1812.08342](https://arxiv.org/abs/1812.08342)  
可以将本文看成是论文的横向缩略版和纵向微拓展版。因为这是一篇去除参考文献还有长达70页的综述，所以在作者展示的几十篇论文中我重点讲些我所关注的，以及一些补充与思考。
主要内容是关于DNN的可信化，即包括验证（`certification`）与解释（`explanation`）两个方面。原文的作者给出了如下的观点。

$$
Trustworthiness = Certification + Explanation
$$

## Certification
验证主要是针对DNN中出现的[对抗样本](https://arxiv.org/abs/1312.6199)的情况。目前的验证主要包括证明（`verification`）和测试（`testing`）。
其中`证明`一般是针对某个给定的输入$X$来验证某个鲁棒属性是否能满足。然后证明又分为准确的和有误差的，但是基本上都受制于问题的规模，毕竟在[这篇文章](https://arxiv.org/abs/1804.09699)里（原文如下），作者证明了计算最近扰动距离是一个NP问题。

> In addition, we show that there is no polynomial time algorithm that can approximately find the minimum $l_{1}$ adversarial distortion of a ReLU network with a $0.99\log_e n$ approximation ratio unless $NP=P$, where $n$ is the number of neurons in the network.

而`测试`就像目前使用测试样例来保证常规软件的需求一样来使用大量或者特殊的`test cases`来测试神经网络找到其中的bug。优缺点都很明显，因为只是执行样例所以可以适应目前主流网络的规模，然而就像目前商业软件交付一样没有严格的形式化方式证明的话依旧有可能产生bug，且在DNN领域还没有样例的`覆盖性度量`指标。

## Explanation
可解释性，顾名思义，就是DNN的开发与设计人员是否知道DNN对于某个输入为什么会输出这样的结果。特别是包含DNN的软件，在交付之时就无法对用户进行说明。

## Verification
作者将所列举的论文根据其底层所使用的方法大致分为图6所示的样子：

![img](/img/2019-6-24/image1.JPG)

同时也可以依据他们方法所具有的保证来进行分类，详细如下：

#### 1. Approaches with Deterministic Guarantees
精确的保证。
1. SMT/SAT: 通过将一个DNN抽象成一组线性算术约束的布尔组合。且无论何时抽象的模型表明是安全的，那么对于原来的模型也是如此。
2. MILP: 混合整数线性规划。如果都是基于MILP，那么方法的不同性一般在于建模的过程或是如何与其它方式相结合。因为底层的实现大多会采用成熟的求解器。

#### 2. Approaches to Compute a Lower Bound
提供接近最小扰动值的下界。
1. Abstract Interpretation: 抽象解释。将问题转换到自行建模的抽象域中求解，并且在抽象域中所能满足的属性原问题也一定能满足。
2. Convex Optimisation: 凸优化。这里有一篇想法非常新颖的论文，其详细内容可以在[这里](https://callmejp.github.io/2019/06/02/provable-defenses-against-ae/)看到。其特点类似于作者提到的`证明`这个大方向。大多数的证明是证明训练好的DNN具有鲁棒性，而这篇论文证明的是训练方法所能带来的鲁棒性。
3. Interval Analysis: 区间分析。放宽每一层的激活函数值的边界来进行递推，和使用`Interval`抽象域的方法，比如[Gehr et al., 2018](https://www.cs.rice.edu/~sc40/pubs/ai2.pdf)，有相似之处。
4. Output Reachable Set Estimation: 这里阅读了[Xiang et al., 2018](https://arxiv.org/pdf/1708.03322.pdf)这篇论文，其中的想法是计算输入中的一个单位的指定干扰对输出造成的最大波动$\delta$，即`maximum sensitivity`。那么对于给定的输入，枚举有限的在$\delta$扰动内的输入集合。对于每个输入集合，根据`maximum sensitivity`来计算输出的可能范围$\tilde{y}$，其因为`maximum sensitivity`的性质，精确的输出范围$y \subset \tilde{y}$。所以可以快速地得出$\tilde{y}$，并与`safety verification`（$S$）的区域进行比较。但是$S$的区域一般性的方法如何得到论文好像没有提及，好像有改进的余地。同样地，扩展范围的通病是在不满足安全性的时候无法确定真实的范围也是如此。
5. Linear Approximation of ReLU Networks: 对于ReLU网络的线性近似。

#### 3. Approaches with Converging Upper and Lower Bounds
提供最小扰动值的上下界。这些方法相较于之前的方法能适应更加大型的网络。
1. Reduction to A Two-Player Turn-based Game: 这也是比较有意思的一篇[论文](https://arxiv.org/pdf/1807.03571.pdf)，将问题归结为类似于博弈的问题，并使用不同的搜索算法来分别接近其上下界。但是比较复杂，看的不是很懂，有好几十页和很多引理与定义。但是实验结果在我看来还没有达到特别理想，因为其上界和下界往往差了一个数量级，比如上界接近5的时候（从上往下），而下界还在接近0.5（从下往上）。也就是说其中一个边界的指导意义不是很大。

## Testing

### Coverage Criteria for DNNs
对于DNN的测试样例覆盖标准。
1. Neuron Coverage: 神经元覆盖。即如果一个神经元是激活的，就称此样例覆盖了该神经元。
2. Safety Coverage: 安全性覆盖。
3. 。。。
4. 剩下还有好多覆盖的标准就不一一介绍了。

### Test Case Generation
生成测试样例
1. Input Mutation: 修改输入。
2. Fuzzing: 模糊测试。通过程序随机地生成大量的测试样本。
3. Symbolic Execution and Testing: 符号执行与检测。作者提到它使用了新奇（`novel`）的方法，所以我详细看了下，没想到这个形容词就是在原文的摘要中，等于只是复制过来。主要是对比于其它的方法，整体来看并不是很新奇。主要亮点应该是传统方法在`DNN`上的又一次尝试。其第一部分内容是找出一个样本中起比较重要作用的像素点。就是代入化简为系数为参数的式子来决定像素点的重要性。然后其检测也只是从重要的像素点中挑选1/2个像素点（肯定是因为直接选的话可能性太多才有第一步，hh）作为未知变量进行符号执行，然后加上另外的一些约束进行求解。而且肯定是因为时间问题，作者只训练了对于`MNIST`的准确率只有92%的小型网络来进行评估。
4. Testing using Generative Adversarial Networks: 使用GAN来评估网络。这我也看了下，就是使用当下火热的GAN生成背景内容丰富又逼真的驾驶场景来对一些网络进行进一步的鲁棒性评估。其中这些`DNNs`的主要功能是根据车载摄像头拍摄的照片来决定车辆当前的驾驶角度。而当作者根据原先的车道生成一些逼真到人眼已经可以识别的天气场景，比如下雨或下雪，会使得原先准确率较高的`DNNs`产生较大的角度误差。
5. Differential Analysis: 翻译成差别分析就差不多符合它的意思了。给定一个输入，绘制两张热力图分别展示正确分类与错误分类时神经元的重要性，那么两张图所表示出具有差异的神经元就会通过GAN生成特定的例子来消除。

###  Model-Level Mutation Testing
模型层面的改动测试。基本上就是变异测试。主要是用于评估测试集的质量。从常规软件的角度来看，就是改动程序的一小部分，然后使用测试集去测试两个程序，如果所有测试集的结果相同就意味着测试集质量不够好，因为它不能够区分已经经过修改的程序。然后联系到DNN的检测就比如：
1. 删除隐藏层的一个或多个神经元。
2. 改变一个或多个激活函数。
3. 改变偏置的参数值等。。。

## Adversarial Attack and Defence

### Attack
对抗攻击这部分就不详细叙述了。我觉得找到没对抗样本只是一个必要条件。就和测试集对DNN进行测试一样，如果对抗攻击的方法可以被证明在找不到对抗样本的情况下就不存在对抗样本，这样的方法就是真正地上了一个台阶。

### Defence
1. Adversarial Training: 对抗训练。将对抗样本添加进训练集并调整目标函数。
2. Defensive Distillation: 蒸馏。主要的思想是来自Hinton的论文。就是设计相同的两个DNN。使用经过原始网络得到的标签来更新数据集（$X$还是原来的$X$，$Y$不是原来的$Y$），并以此来训练新的网络。
3. Dimensionality Reduction: 维度缩减。比如将784维的MNIST样本缩减为20维的输入进行训练。
4. Input Transformations: 转换输入。比如提前判断测试的样本是否是对抗样本或是对测试的样本进行一些操作，比如说缩放、裁剪等。这个照我的理解就是比方说有针对几个单独像素点进行改动的对抗样本，那么进行高斯滤波消除噪声，应该可以在不影响判断结果的前提下缓解这些突出像素点所造成的影响。
5. Combining Input Discretisation with Adversarial Training: ...
6. Activation Transformations: 针对`激活`的操作。在前馈操作时，随机去掉一些节点，保留节点的概率与其激活程度成正比。然后再扩展幸存的节点。
7. Characterisation of Adversarial Region: 寻找对抗区域。即对抗样本聚集的区域。
8. Defence against Data Poisoning Attack: 这个就比较少见了，攻击者能够对训练数据做手脚。
9. 上述1~8点对于防御后的结果有效性是无法做出保证的。而要达到有保证的程度一般是改进训练目标，也就是损失函数或者是正则化项。这和`Approaches to Compute a Lower Bound`中第二点介绍的论文就有相同之处了。

## Interpretability
可解释性太抽象了，就大致讲讲几个大方向。
1. 可视化的解释。
2. 关于图片中存在特征的`Rank`。
3. 和第二点差不多，就是绘制特征图（` Saliency Maps`）。
4. 用影响函数（` Influence Functions`）理解`DNN`。
5. 使用简化的模型来替代`DNN`。
6. 使用信息理论的信息流方法。

## Future Challenges
未来的一些挑战。下面的点都是作者的总结的，我觉得这些相对有价值些。
1. 
2. 