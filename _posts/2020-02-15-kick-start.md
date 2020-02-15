---
layout:     post
title:      "Kick Start中比较有意思的题(上)"
subtitle:   "不想进Google的程序员不是。。。"
date:       2020-02-15
author:     "JohnReese"
header-img: "img/2020-2-11/stone.jpg"
mathjax: True
tags:
    - Algorithm
---

听说Kick Start上的题质量很高，又是想进Google的笔试唯一渠道？不过其实也没啥特别的想法，就是想再提高下思维能力而已。

官网上最早有记录的是13年，而且近几年的题目都有专门的官方解析，所以我就整理下自己能看懂且比较有意思的题目分享一下吧。总体的感觉难度在ACM银牌水平，CF/Atcoder有2000分左右应该是稳的，但很遗憾，我还是太菜，hh。另外题目整体偏长，有些考验英语阅读能力。不过也没办法，毕竟是外企。所以有些题面相对复杂的我就不转述了，但都会给出链接，不过好像需要翻墙。

至于源码，在ScoreBoBoard能看见别人比赛时的源码，够用了。

这部分是2013年-2016年。

## 2016

### Round E

前三道的题解：[https://blog.csdn.net/fcxxzux/article/details/53055195](https://blog.csdn.net/fcxxzux/article/details/53055195)

还有一道难题。。

### Round D

**Problem D**: [https://code.google.com/codejam/contest/5264486/dashboard#s=p3](https://code.google.com/codejam/contest/5264486/dashboard#s=p3)

每根橡皮筋长L-R，价格为p。问为了达到某个长度Len，最少需要多少钱。其中橡皮筋组合长度就是(L1+L2)-(R1+R2)。

DP和滑动窗口的完美结合。博客写的很好，我就不罗嗦了。

[https://blog.csdn.net/ihsin/article/details/54860708](https://blog.csdn.net/ihsin/article/details/54860708)

### Round C

**Problem D**: [https://code.google.com/codejam/contest/6274486/dashboard#s=p3](https://code.google.com/codejam/contest/6274486/dashboard#s=p3)

一道考验思维能力的博弈题：每个士兵都有A，D两个属性。两个人轮流拿士兵，并且有条件，其中被拿的士兵至少有一个属性的值要大于所有已经被挑选的士兵的对应属性值。不能拿士兵的人就输了。

Alice是先手，假设她拿了A属性最大的士兵；那么Bob一拿D属性最大的士兵，Alice就输了。因此，Alice不能拿A/D属性值最大的士兵。`但是有一个特例，如果A，D属性的最大值集中在一个士兵身上，那么Alice一拿就直接赢了，这也是程序的终止条件`。

综上，谁拿A/D属性最大的士兵谁输，所以对于他们来说，这变成了一个子游戏，因为默认排除了A/D属性最大的士兵，接着子游戏又可以衍生出子游戏，以此类推直到终止条件。

推荐No.3选手的代码。

**Problem B**: [https://code.google.com/codejam/contest/6274486/dashboard#s=p1](https://code.google.com/codejam/contest/6274486/dashboard#s=p1)

我觉得看似简单，实则复杂。强烈推荐看No.2选手的代码——团结，精确，完美。

### Round B

**Problem C**: [https://code.google.com/codejam/contest/5254487/dashboard#s=p2](https://code.google.com/codejam/contest/5254487/dashboard#s=p2)

给出n段区间，任意去掉1个区间，求剩下的n-1段区间的范围的并的最小值。

其实本质上应该算常规题。但对于我这种没接触过的还是蛮有意思的。

首先是带有离散化的思想，只需要对区间的端点进行排序就够了。然后遍历，遇到左端点cnt++，否则cnt--。那么当cnt为1的时候我们知道这段范围是某个区间`单独占有`的，那么我们把单独占有的范围累加到这个区间。最后的结果就是所有的范围减去单独占有范围最大的区间的单独占有范围。

**Problem D**: [https://code.google.com/codejam/contest/5254487/dashboard#s=p3](https://code.google.com/codejam/contest/5254487/dashboard#s=p3)

看AK数就知道难题一道。

就算你想出了这绝妙的DP，还需要进行一定的数学推导进行优化。有兴趣的可以看看这个博客。[https://blog.csdn.net/fcxxzux/article/details/52346364](https://blog.csdn.net/fcxxzux/article/details/52346364)。


### Round A

**Problem B**: [https://code.google.com/codejam/contest/11274486/dashboard#s=p1](https://code.google.com/codejam/contest/11274486/dashboard#s=p1)

二维数组表示海拔高低，问低洼地最多可以蓄多少水。其中边界直接与海相连。

乍一看，似乎很简单。但我却想不到好的解法。

可以参考[https://blog.csdn.net/zqh_1991/article/details/52336482](https://blog.csdn.net/zqh_1991/article/details/52336482)的做法——优先队列加DFS。


## 2015

### Round E

**Problem D**: [https://code.google.com/codejam/contest/8264486/dashboard#s=p3](https://code.google.com/codejam/contest/8264486/dashboard#s=p3)

看AK数你就知道这道题应该很秀。还好网上有解答。

题意就是一个Array有n + (n-1) + (n-2) + ... + 1 = (1 + n) * n / 2 个SubArray。那么我们对所有的SubArray分别求和，形成一个长度为(1 + n) * n / 2的NewArray。然后再求NewArray其中的一段区间(L~R)的和。

乍一看，好像没啥提速的方法。确实看了解答的思路我觉得真滴秀。

令数组a为Array的前缀和，即a[1] = A[1], a[2] = A[1] + A[2]...。再令数组b为a的前缀和。

第一步，通过二分确定NewArray[R或(L-1)]是多大。对于二分时的mid，如何快速确定NewArray中小于mid的元素有多少个呢？通过数组a。比如说a[i] - a[j] <= mid了，那么就有(i - j)个元素是小于mid的了。故外循环i从1到n，内循环j从1到i统计个数，个数和小于R的话mid就大一点，否则小一点。

第二步，ok，目前我们得到了准确的mid，现在就是利用mid来计算NewArray中1~R或（L-1）的前缀和了。同样也是借助第一步的i，j循环，我们知道我们需要把a[i]-a[j], a[i]-a[j+1],...,a[i]-a[i-1]这些元素加到我们最后的结果中。整理一下就是(i-j)*a[i]-(a[j] + a[j+1]... + a[i-1])，其中括号包裹的减数就是b[i-1]-b[j-1]。

最后的结果就是NewArray前R个元素的前缀和减去前（L-1）个元素的前缀和。

### Round D

这一轮我基本全跳了。。。

### Round C

**Problem D**: [https://code.google.com/codejam/contest/4284487/dashboard#s=p3](https://code.google.com/codejam/contest/4284487/dashboard#s=p3)

如果你和我一样还不知道`双端队列维护固定长度的区间最值的二维形式的问题`，那么可以看看[https://blog.csdn.net/ihsin/article/details/55224377](https://blog.csdn.net/ihsin/article/details/55224377)。

### Round B

**Problem D**: [https://code.google.com/codejam/contest/10214486/dashboard#s=p3](https://code.google.com/codejam/contest/10214486/dashboard#s=p3)

计算序列中有多少种合法的DNA序列。计数一般和DP八九不离十。就是如何安排状态及转移条件。

这道题主要是维数高，常见的做法是4维。首先有一维（长度为4）肯定是记录上一位是a/b/c/d。还因为a/c对应，b/d对应所以我们只需要分出两维（长度为N）保存a/b的个数。最后一维表示前i位，这样的话就是N * N * N * 4，数组占差不多10的7次方个int，稍稍有点多。当然，这是可以过的。

当然，优化内存也很简单。就是最后一维不需要N的空间，采取滚动数组的方式。具体可以参考No.1选手的代码。

最后这道题的转移情况反而比较简单，我们只需要分别根据当前的记录状态和当前的字符分别进行分类讨论。

## 2014

### Round A

**Problem D**: [https://code.google.com/codejam/contest/3214486/dashboard#s=p3](https://code.google.com/codejam/contest/3214486/dashboard#s=p3)

给出无限个相同的正方形A，从其中切割出指定的正方形a1,a2...，问最少需要几个正方形A。

我开始没想明白，觉得很复杂。因为一个A可以切割成一个大的和好多小的，不知道如何记录剩余的可用面积。

看了人家的代码发现可以先对a1, a2...由大到小排列。那么每次切割肯定是挑角落进行切割。剩余一个‘L’形的图形。这个时候直接把‘L’分为两个长方形继续利用就ok了。对长方形切割也是一样的思路。

具体为什么可行关键点在于由大到小的排序，略加思索都能想明白。

### Round B

**Problem D**: [https://code.google.com/codejam/contest/4214486/dashboard#s=p3](https://code.google.com/codejam/contest/4214486/dashboard#s=p3)

题意是左括号比右括号“小”，所以我们优先放左括号。那么什么时候不能放了呢？看个例子。

假设当前n=3，意味着我们要输出6个字符。假设当前已经输出了0到i-1（下标）总共i个字符，且左括号比右括号多输出了u个。那么如果第i位，即第i+1个字符我们还要输出左括号的话，就意味着剩余的6-i-1个字符中至少有一种方式右括号比左括号多u+1个。那么联系到k，就意味着k小于par[6-i-1][u+1]，`其中par[i][j]表示i个字符中右括号比左括号多j个的排列数`。这样我们就引出了DP，par可以预处理得到。

接着考虑，如果此时k大于par[6-i-1][u+1]了怎么办，肯定是输出右括号，那么k会有什么变化吗？翻译一下当前的情况：也就是k比第i位放左括号的所有方式的个数都要大。其中`第i位放左括号的所有方式的个数`就是par[6-i-1][u+1]，那么我们令k减去这个数（这个细节我想了很久还是没有完全get）。之后以此类推。

## 2013

### Round A

**Problem A**: [https://code.google.com/codejam/contest/2924486/dashboard#s=p1](https://code.google.com/codejam/contest/2924486/dashboard#s=p1)

很明显是找规律（但我一脸懵逼。。）。但看了别人的提交有种茅塞顿开的感觉。好，不扯淡了。

本质上每个数都是由上一行对应的数选择p/(p+q)或者(p+q)/q操作生成的，也就是题意。然后可以发现每行的个数呈指数级递增，于是想到二进制位对应每一行。尝试验证：1代表1/1，10代表1/2，11代表2/1，100代表1/3，101代表3/2。。。

可以发现，由左到右的0/1分别代表选择哪种操作。比如1/3，忽略第一位的1，因为每个数都经过1/1。然后是0，所以选择p/(p+q)生成1/2，最后也是0，再次选择p/(p+q)生成1/3。

因此只需O(lgN)就搞定了。

**Problem B**：[https://code.google.com/codejam/contest/2924486/dashboard#s=p3](https://code.google.com/codejam/contest/2924486/dashboard#s=p3)

这道题是一个“探险技巧”的BFS模拟：就是在一个普通的迷宫中，你只要一直贴着墙壁沿左方向走就一定能够走出去。

虽然想了很久，但居然想出来了。基础情况处理比如碰壁原地逆时针旋转90度，记录当前的前进方向啥的就不详细说了。主要是每次移动需要两步一起考虑：也就是朝着当前的方向走出一步以后能左转一步的话就直接再左转一次，也就是连走两步；否则就走一步。