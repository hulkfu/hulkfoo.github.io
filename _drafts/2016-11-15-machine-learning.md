---
layout: post
title: 机器学习
permalink: ml
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

那么如何选择特征呢？信息论里的熵越大说明这个信息的信息量越多。

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

### 划分数据集

```py
def splitDataSet(dataSet, axis, value):
    """根据指定维度axis，筛选出其值是value的子数据集。

    Args:
      dataSet: 待划分数据集。
      axis: 划分数据集的特征。
      value: 特征值。
    return:
      筛选的子集。
    """
    retDataSet = []
    for featVec in dataSet:
        if featVec[axis] == value:

            # 这两行的意思就是这个vec，除了axis那个元素的value不要，前后的都要
            reducedFeatVec = featVec[:axis]
            reducedFeatVec.extend(featVec[axis+1:])

            retDataSet.append(reducedFeatVec)
    return retDataSet
```

### 选择最优的特征

关于dataSet的格式：

1. 由列表组成，且所有列表长度相同。
2. 列表的最后一个元素是当前实例的类别标签。

```py
def chooseBestFeatureToSplit(dataSet):
    numFeatures = len(dataSet[0]) - 1
    baseEntropy = calcShannonEnt(dataSet)
    bestInfoGain = 0.0;
    bestFeature = -1;
    # 遍历每个特征
    for i in range(numFeatures):
        # 获取所有唯一特征值
        featList = [item[i] for item in dataSet]
        uniqueVals = set(featList)
        newEntropy = 0.0
        for value in uniqueVals:
            subDataSet = splitDataSet(dataSet, i, value)
            prob = len(subDataSet)/float(len(dataSet))
            newEntropy += prob * calcShannonEnt(subDataSet)
        # 信息增益，
        infoGain = baseEntropy - newEntropy
        if (infoGain > bestInfoGain):
            bestInfoGain = infoGain
            bestFeature = i
    return bestFeature
```

### 递归构建决策树

递归结束的条件是：程序遍历完所有划分数据集的属性，或者每个分支下的实例都具有相同的分类。

如果已经处理了所有属性，但是标签依然不是唯一的，那么可以采用多数表决法决定该叶子节点的分类。

```py
def majorityCnt(classList):
    import operator
    classCount = {}
    for vote in classList:
        classList[vote] = classList.get(vote, 0) + 1
    sortedClassCount = sorted(classCount.items(), key=operator(1), reverse=True)
    return sortedClassCount[0][0]
```

```py
def createTree(dataSet, labels):
    classList = [item[-1] for item in dataSet]
    # 所有数据集一个类别
    if classList.count(classList[0]) == len(classList):
        return classList[0]
    # 已经遍历完了所有特征，但剩下的还是不一个类别
    if len(dataSet[0]) == 1:
        return majorityCnt(classList)
    bestFeature = chooseBestFeatureToSplit(dataSet)
    bestFeatureLable = labels[bestFeature]
    myTree = {bestFeatureLable:{}}
    del(labels[bestFeature])
    featureValues = [item[bestFeature] for item in dataSet]
    uniqueVals = set(featureValues)
    for value in uniqueVals:
        subLabels = labels[:]
        myTree[bestFeatureLable][value] = createTree(splitDataSet\
            (dataSet, bestFeature, value), subLabels)
    return myTree
```


# 朴素贝叶斯分类法

所谓朴素：

1. 上下文不相关。
2. 每个特征的权重一样。

- 优点： 在数据较少的情况下仍有效，可以处理多类别问题。
- 缺点： 对于输入数字的准备方式较为敏感。
- 适用数据类型： 标称型数据。

贝叶斯准则：

$$ p(c|x) = \frac{p(x,y|c_i)p(c_i)}{p(x,y)} $$

因此，可以定义贝叶斯分类准则：

如果P(c_1|x, y) > P(c_2|x, y)，那么属于类别c_1，反之属于c_2.

# 框架
## [TensorFlow](https://www.tensorflow.org/)

```bash
# without gpu
pip install tensorflow
# with gpu
pip install tensorflow-gpu
```
## [Scikit-Learn](https://github.com/scikit-learn/scikit-learn)

## [PyBrain](https://github.com/pybrain/pybrain)


# [吴恩达的机器学习公开课](https://www.coursera.org/learn/machine-learning)
虽然有斯坦福的公开课，但是 Coursera 上的课程安排更适合远程学习，有很好的课程规划，正常是 11 周学完。

## Week 1
介绍了机器学习，主要分为监督学习和无监督学习。主要区别在提供的样本是否给出了相应的结果。

并主要讲了用梯度下降算法来解决线性回归的问题。算法之前高数时学过，还做过题，就是不知道干嘛用的。

最后对线性代数做了简介。




# 感想
在编程中其实用到过上面提到的算法，原来就是机器学习啊！

如果上学的时候，配着机器学习学概率论、线性代数等，那该多好啊。它们不再是死公式，而是活知识，
而且直接可以在代码中计算出来。

虽然现在人工智能很火，在我看来，机器学习是路径，正是通过学习实现了人工智能。就像人通过学习，才变得有智慧。
