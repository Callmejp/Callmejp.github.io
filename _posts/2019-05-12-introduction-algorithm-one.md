---
layout:     post
title:      "算法导论笔记（上）"
subtitle:   "持续更新。。。"
date:       2019-05-12
author:     "JohnReese"
header-img: "img/home-bg-o.jpg"
mathjax: True
tags:
    - Algorithm
---

笔记对应的课程为：[https://www.bilibili.com/video/av48922404/?p=3](https://www.bilibili.com/video/av48922404/?p=3)
其中本网页包含前10次视频中的8次，第1，2次视频没有包含。有兴趣的同学可以自行观看。

## 目录
1. [分治法](#分治法)
2. [快排&随机化算法](#快排&随机化算法)
3. [线性时间排序](#线性时间排序)
4. [顺序统计&中值](#顺序统计&中值)
5. [哈希表](#哈希表)
6. [全域哈希&完全哈希](#全域哈希&完全哈希)
7. [二叉搜索树](#二叉搜索树)
8. [平衡搜索树](#平衡搜索树)

## 1. 分治法
> 英文：divede and conquer

#### Questions
1. 计算 $x^{n}$
   * 当n为偶数时拆分为$x^{\frac{n}{2}}x^{\frac{n}{2}}$；当n为奇数时拆分为$x^{\frac{n-1}{2}}x^{\frac{n-1}{2}}x$
2. 斐波那契数列
   * $F_{n} = \frac{\varphi^{n}}{\sqrt{5}}$rounded to nearest interger, 其中$\varphi$是黄金比例。但是我们知道计算机内部是用有限位数的二进制来表示无限的实数所以肯定有误差。
   * 用ACM的`术语`应该是转换为矩阵快速幂，即$F_{n}$可以由
     $$
     \left[
       \begin{matrix}
         1 & 1\\
         1 & 0\\
       \end{matrix} 
     \right]^{n}
     $$
     计算得出。令外还介绍了矩阵相乘的$O(n^{\log_2 7})$时间复杂度的做法，叫Strassen算法。
3. VLSI: Very Large Scale Integrated
   * 这个比较神奇，第一次看到将分析时间复杂度的方法应用到集成电路的设计上。`Awesome!`

## 2. 快排&随机化算法
#### Questions
1. quicksort
   * 假设我们当前要为p-q范围的数组进行快排。我们首先确定`Arr[p]`，即第一个`element`为主元（`pivot`)。然后变量`i`指向小于等于`Arr[p]`的最后一个元素的位置，变量`j`指向大于`Arr[p]`的最后一个元素的位置。(`这里的最后一个指的是遍历到的元素而不包括j之后的元素`)
   * 效果图：Arr[p],..., Arr[i], ..., Arr[j], ..., Arr[q]
   * 其中`Arr[p]`是`pivot`，然后第一个省略号包括的是小于等于主元的元素，第二个省略号是大于主元的元素，第三个省略号包括的是未遍历到的元素。
   * 每次j向后移动一位(j = j + 1)，如果Arr[j]大于主元则继续移动，否则i = i + 1, 交换Arr[i]和Arr[j]。
   * 一些初始化和结束条件可以自己思考一下。最后是交换`Arr[p]`和`Arr[i]`，递归排序Arr[p~i-1]和Arr[i+1~q]。
   * ------------------------------我是分割线------------------------------
   * 想一想快排的最坏情况是什么: 恰恰是数组是有序或倒序的时候，因为这样小于等于或大于主元的一个部分是空的，整体会退化成$O(n^2)$的时间复杂度算法。
   * 为了避免最坏的情况，我们引入随机化，即随机选择主元，这样我们就无需关注数组的排列了。
   * 教授花了大把的时间，通过假设概率分布来验证快排在引入随机化后的平均时间复杂度是优秀的$O(n \cdot \log_2 n)$。`（这里底数不一定为2，只是我没找到没底数的表示）`

## 3. 线性时间排序
#### Questions
1. Decision-tree Model: 决策树
   * 个人感觉不是很重要，就像是排序算法过程的可视化。
2. Counting Sort: 计数排序
   * 伪代码带有技巧性方面的复杂，但是本质理解起来并不难。和之后的`基数排序`一样，都是对只对正整数进行排序，且计数排序还需要最大的元素比较小才可以，否则复杂度很高。
   * 假设我们所要排列的数组中元素的大小是`1-k`，那么我们建立数组`C[k]`来保存元素`i`出现的次数。输出的时候只需要遍历`C[k]`并按`C[i]`的大小重复输出`i`。
   * 所以复杂度是$O(n + k)$，其中n是数组的个数，所以也就不难理解假如`k`很大，计数排序就会非常慢而且吃内存。
3. Radix Sort: 基数排序
   * Stable Sort: 稳定排序。这是讲基数排序前需要了解的概念。也就是排序算法会不会打乱数组中相等元素的原来的相对顺序（`relative order`）。比如快排是不稳定排序，归并排序是稳定排序。
   * 假设我们排列的是十进制的正整数，我们按照最后一位（个位）对所有元素进行排序，再按照十位进行排序，以此类推。如果位数不对等，我认为只要在前面补0就行了。如果你写一些数字尝试一下你就会发现必须使用稳定排序，或者明确点就是使用计数排序。
   * ------------------------------我是分割线------------------------------
   * 同时我们也知道数也可以使用二进制进行表示，比如${15_{10}}={1111_2}$。我们可以拆分为{1, 1, 1, 1}（这时候$b=4,r=1$,也就是我们需要遍历4轮），也可以拆分为{11, 11}(即{3, 3}，这时候$b=4,r=2$)或者其他方式。那么我们假设将`r`个bit合在一起，则基数排序的时间复杂度为$O(\frac{b}{r}\cdot(n+k)) = O(\frac{b}{r}\cdot(n+2^r))$。所以你可以在b给出的情况下求导得出最优的r？

## 4. 顺序统计&中值
> 顺序统计: Order Statistic

#### Questions
1. find k-th smallest element:
  * Naive algorithm: 排序并返回第K个元素。但是可以有接近线性时间的算法。
  * 同样也是基于`Randomized divide&conquer`, 很大程度上是类似于快排的做法。
  * 假设我们寻找的是第`i`小的数。按照快排中的步骤，将数组`A[]`的一部分排序成：A[p], ..., A[r], ..., A[q]。
  * 其中第一个省略号是小于等于`A[r]`的元素，第二个省略号是大于`A[r]`的元素。我们称`A[r]`为`partition element`, 并令`index of A[r]`为`k`，接下来我们讨论三种情况（其中`Rand-Select(A, p, q, i)`就是找到`A`中p-q范围内第`i`小的数，步骤就是我们正在叙述的内容）：
    ```c
    if i == k then return A[r]
    if i < k then return Rand-Select(A, p, r-1, i)
    else return Rand-Select(A, r+1, q, i-k) 
    ```
  * 算法很简单，但是分析其预期时间复杂度是比较繁琐的过程，需要建立指示随机变量和替代法等步骤。我这边只记录结论（感觉这就是国内外的差距吧，hh）：
    1. 最坏情况: $O(n^2)$.
    2. 最好情况: $O(n)$.
    3. 整体: $O(n)$.
2. Worst-case linear-time order statistic: 
  - 我们想要排除最坏情况下时间复杂度退化为$O(n^2)$的情况，尽管它出现的概率很小。就要用到这个由众多大牛提出的算法。
  - 我先阐述下这个算法的个人观点，从之前的算法退化成$O(n^2)$的原因中我们可以很直观地体会到可能是由于`partition element`的选择导致了每一步我们只能排除1个元素。所以有没有一个$O(n)$的算法让我们能找到`确定好`的`partition element`。这样就能保证整体的算法是$O(n)$的时间复杂度。这是`Worst-case linear-time order statistic`的关键。
  - 它的第一步非常的特殊，是将需要排序的数组排列成`5 X (n/5)`的序列，当然第5行可能会少些元素。
  - 我们把每一列，也就是5个元素组成一个`group`并找到每个`group`的`median`。这一步你甚至可以排序，因为只有5个元素，你只要保证这一步的时间复杂度是$O(n)$就可以了。
  - 这样我们就得到了`(n/5)`个中值，我们需要选出中值中的中值，假设为`X`,这样我们就保证了至少有：`(1)`$\lfloor{\lfloor{\frac{n}{5}}\rfloor}\div 2\rfloor$个元素是小于等于`X`。`(2)`$\lfloor{\lfloor{\frac{n}{5}}\rfloor}\div 2\rfloor$个元素是大于等于`X`。
  - 这个结论可能不够直观，画个图的话会便于理解。同时简单且直观的结论是大概有$\frac{n}{4}$的元素被`X`分隔在它的一边。
  - 这样$T(n)\leq{T(\frac{n}{5})+T(\frac{3n}{4})+\theta(n))}$，通过一番证明可以确定时间复杂度在$O(n)$以内。

## 哈希表
1. 概念
  * Hash function `h` maps keys "randomly" into slots of  table `T`.
  * The `load factor` of a hast table with `n` keys and `m` slots is $\alpha=\frac{n}{m}$.
2. 解决冲突(collision)
  * When a record to be inserted maps to an already occuped slot, a collision occurs.
  * 链接法：使用邻接表的形式接在已存在单元的位置的后面。
  * 后面有更多更好的方法。
3. 时间复杂度
  * 最坏情况下：当所有key哈希成相同slots就退化成邻接表了，所以一次`operation`的时间复杂度为$O(n)$。
  * 平均情况下：
    1. 预期的失败操作的时间复杂度是：$\theta(1+\alpha)$.
    2. 预期的成功操作的时间复杂度是：$\theta(1)$.
    3. 所以操作并不是常数级别的时间复杂度，它是与$\alpha$有关的。也就是说如果你的哈希表很拥挤那么它的时间复杂度一样也很高。
4. hash function
  * Divison method: h(k) = k mod m，一般我们选择`m`是一个质数。
  * Multiplication method: 
    1. 假设computer has `w`-bit words.
    2. $m = 2^r$
    3. $h(k)=(A \cdot k mod 2^{w}) rsh (w-r)$，其中$2^{w-r} < A < 2^w and A mod 2 = 1$，`rsh`是位右移操作。 
5. 再谈解决冲突
  - open addressing: 开放寻址法。hash之后再hash。`具体内容待补充`。
    1. 线性探查。$h(k, i) = (h(k, 0) + i) mod m$。简单的加一，但可能会造成`clustering`。也就是很多元素聚在了一起。
    2. Double hashing。$h(k, i) = (h_{1}(k) + i \cdot h_{2}(k)) mod m$。
    3. 时间复杂度分析：$\theta(1/(1 - \alpha))$。所以$\alpha < 1$则为$O(1)$。但会随着$\alpha$的增大导致操作次数的增加。

## 全域哈希&完全哈希
1. 全域哈希
  * `Randomness`，即随机地选择哈希函数。
  * $U$: 键值的全域；$H$: 将$U$映射到大小为m的Table中的哈希函数的有限集合。
  * 注意，$H$是全域(`universal`)是需要满足的条件——>$H$ is universal if $\forall x, y \in U$ where $x \neq y.$其中$\mid \{h \in H\}.h(x)=h(y) \mid = \frac{\mid H \mid}{m} $。翻译过来就是使得`h(x)=h(y)`的哈希函数的数目是$\frac{\mid H \mid}{m} $。
  * `那么从中随机选择一个哈希函数使得x, y碰撞的概率就是`$\frac{1}{m}$。
  * `Theorem`: 从$H$随机选择一个函数，并假设我们要将`n`个键值放进`T表`的`m`个槽里。那么对于一个给定的`x`, $E[collisions(x)] < \frac{n}{m}$。也就是期望碰撞次数小于`装载因子`。
  * 一种构造全域Hash的方法：令`m`是质数，将键值`k`分解成`r+1`位的`m`进制数，同时我们随机选择同样是`r+1`位的`m`进制数`a`。最终$h(k)=(\sum_{i=0}^r a_i \cdot b_i) mod m$。所以共有$m^{r+1}$个哈希函数。
2. 完全哈希
  * 主要是解决查询时的最坏情况。目标是`Given n keys, construct a static hash table of size m = O(n)`。可以看到是静态的，没有更新操作，且表不需要很大。且最坏情况下的查询都是`O(1)`。
  * 使用两级结构，其中每一级都使用全域哈希，也就是做两次哈希操作。其中第一级的每个槽都定义了一个供下一级使用的哈希函数。
  * 其中如果有$n_i$个键值映射到第一级的同一个槽，那么第二级就有$m_i$个槽且$m_i = n_{i}^2$。通过第二级的稀疏性来以大概率找到不会碰撞的哈希函数。
  * 经过证明发生0碰撞的概率至少是$\frac{1}{2}$。


## 二叉搜索树
1. 概念
  * 感觉课程面对的是需要二叉树基础的同学。那我在这里简单介绍下`二叉搜索树`。就是普通的二叉树，每个节点都有一个值，且左子树的值小于父节点的值，右子树的值大于父节点的值。根据这个规则，你可将一个数组依次从根节点开始放起，总能找到一个唯一的位置来存放。建完这棵树后你通过`前序遍历`就可以依次获得数组中从小到大的元素。
  * 但我们一般是要对其进行更新和查询的。所以我们希望对于它的每一次操作的时间复杂度都是`O(logN)`，`N`是树中的节点个数。这也是理论上的最优时间复杂度了。因为树的高度最小也是`logN`。课程中将它的构造过程和快排算法进行比较来说明它们是很相似的，但是我认为只是让同学们更多地了解构造的时间复杂度和为之后的证明做铺垫。但是实际中二叉搜索树有更大的作用。
  * 此课程教授主要是证明了一个`Theorem`: 随机化构造的二叉搜索树的期望的高度是`logN`，这样就能达到理论上查询操作的最优时间复杂度。随机化就是随机地排列数组中原来的元素。`注意这里只有查询操作，因为更新操作会增加树的高度。如何使得更新操作也是logN的时间复杂度请看下节`。
  * 接下来就是证明，太严谨了，连引理教授都证明了。有兴趣的同学可以自己看下。好像B站的视频不完整。

## 平衡搜索树
#### 主要讲的是红黑树，当然其前提是二叉搜索树。
1. 四点性质
  * 每个节点是红色或黑色的。
  * 根节点和叶子节点是黑色的。
  * 每个红色节点的父节点一定是黑色的。
  * 任意节点到其子树中所有的叶子节点的路径所包含的黑色结点数是相同的。
  * `还有一点并不在这四点里面，但是我觉得并不是显然的，就是叶子节点不带数值。`
2. 从性质推断红黑树的高度是`O(lgN)`
  * 先将每个红色节点移动到其父节点一起，所以原来的黑色节点可能会包含1、2或3个原来的节点，而其直接子节点就会有2、3或4个。就变成了一棵2-3-4树，其中有个优雅的性质，所有的叶子节点的深度一样，这其实是个很直观的结论。
  * 其中通过变化后的树的叶节点数是`(n + 1)`（这好像是个结论）。而又因为其是2-3-4树，所以$2^{h'} \leq Leaves \leq 4^{h'}$。所以我们就知道变化后树的高度是`(lgN)`级别的，所以原来的树的高度最多是它的2倍而已`(性质3)`。
3. 如何进行更新操作
  - 谁知道呢。？ hh
  - 主要有2大类或6小类的情况需要讨论，实在有点复杂。而且旋转的操作和节点分布的情况需要借助于图表才能方便理解。
  - 主要说下操作的中心思想：
    1. 先将需要插入的节点放到确定的叶子节点（指的是未插入前为叶子节点）处，并令其为红色，使其保持性质4。
    2. 进行Rotation操作，使得关注的焦点从倒数第二层（也就是插入的点的位置）不断地向上转移并保持子树满足其四个性质。
    3. 最后可能需要将根节点转变为黑色就满足了。
    4. 所以最多反转`O(lgN)`次。
4. 如何进行删除操作
  - 这个教授没讲。将近90min，就是为了把一个操作给吃透。我还没看博客学习过，但根据我的经验应该是将一个节点放到删除节点处，然后再不断地进行分类讨论和旋转操作。

# 上十回，Over！
