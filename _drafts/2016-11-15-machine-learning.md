---
layout: post
title: 机器学习
---

第四次科研革命，未来就在其中。

# 如何选择合适的算法

1. 使用机器学习算法的目的，想要算法完成何种任务，比如是预测明天下雨的概率还是对投票者按兴趣分组。
2. 需要分析或收集的数据是什么。

首先考虑使用机器学习算法的目的。若果想要预测目标变量的值，则可以选择监督学习算法，否则可以选择
无监督学习算法。确定选择监督学习算法之后，需要进一步确定目标标量类型，如果目标变量是离散型，如
是/否、1/2/3、A/B/C或红/黄/绿等，则可以选择分类算法；如果目标变量是连续型数值，如0.0~100.00、
-999~999等，则需要选择回归算法。

如果不想预测目标变量的值，则可以选择无监督学习算法。进一步分析是否需要将数据划分为离散数组。如果
这是唯一需求，则使用聚类算法；如果还需要估计数据与每个分组的相似度，则需要使用密度估计算法。


# 开发机器学习应用程序的步骤

1. 收集数据
2. 准备输入数据
3. 分析输入数据
4. 训练算法
5. 测试算法
6. 使用算法

# k-近邻算法（k Nearest Neighbors, kNN）

k-近邻算法采用测量不同特征值之间的距离方法进行分类。

算出来目标离哪一类近就是哪一类的，物以类聚的思想。

- 优点： 精度高、对异常值不敏感、无数据输入假定。
- 缺点： 计算复杂度高、空间复杂度高。
- 适用数据范围： 数值型和标称型。

kNN不需要训练算法。

一个典型通用的分类程序：

```py
from numpy import *
import operator

def classify0(inX, dataSet, labels, k):
    """根据给定数据集，对目标向量进行分类。

    Args:
      inX: 用于分类的输入目标向量，类型是一般数组。
      dataSet: 输入的训练样本集，类型是NumPy的array，所以支持矩阵运算。
      labels: 样本集的标签，即类别。
      k: 用于选择最近邻居的数目。
    Retruns:
      inX的类别。
    """
    dataSetSize = dataSet.shape[0]
    # tile，“铺”的意思，对inX进行复制，从而和dataSet的规模一样。相减后得到每个样本与目标从diff
    diffMat = tile(inX, (dataSetSize,1)) - dataSet  
    sqDiffMat = diffMat**2
    # 求平方和，axis=1表示横着sum，然后输出
    sqDistance = sqDiffMat.sum(axis=1)
    distances = sqDistance**0.5

    # argsort，返回数组从小到大的索引。比如array([3,1,2]).argsort()，返回array([1,2,0])
    # 得到按距离排序的数组index
    sortedDistIndicies = distances.argsort()

    # 统计根据前k个样本得到的目标的分类情况
    classCount={}
    for i in range(k):
        voteIlabel = labels[sortedDistIndicies[i]]
        classCount[voteIlabel] = classCount.get(voteIlabel,0) + 1

    sortedClassCount = sorted(classCount.iteritems(), key=operator.itemgetter(1),
        reverse=True)

    return sortedClassCount[0][0]
```

可见，对每一个目标进行分类，都需要把其与样本里所有的距离算一遍，让后选出最近的k个，并统计这k个类别，
得到目标的类别。

kNN就是根据样本来学习的，为了准确率，我们又要提供足够多的样本集。可见其计算复杂度。


# 决策树

通过特征将原始数据划分为几个数据子集。

一直迭代到所有实例具有相同的分类。

- 优点： 计算复杂度不高，输出结果易于理解，对于中间值确实不敏感，可以处理不相关特征数据。
- 缺点： 可能会产生过度匹配问题。
- 适用数据类型： 数值型和标称型。

那么如何选择特征呢？信息论里的熵越大越好，越大说明这个特征的区分度越高。

信息熵的公式：

$$H=-\sum_{i=1}^n p(x_i)log_2p(x_i)$$

计算代码：

```py
def calcShannonEnt(dataSet):
    numEntries = len(dataSet)
    labelCounts = {}
    for featVec in dataSet:
        currentLabel = featVec[-1]
        labelCounts[currentLabel] = labelCounts.get(currentLabel, 0) + 1

    shannonEnt = 0.0
    for key in labelCounts:
        prob = float(labelCounts[key])/numEntries
        shannonEnt -= prob * log(prob,2)

    return shannonEnt
```


# 感想
在编程中其实用到过上面提到的算法，原来就是机器学习啊！

如果上学的时候，配着机器学习学概率论、线性代数等，那该多好啊。它们不再是死公式，而是活知识，
而且直接可以在代码中计算出来。
