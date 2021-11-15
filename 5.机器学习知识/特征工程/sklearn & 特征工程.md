# 1 前言

该篇博客主要涉及到sklearn.feature_selection 以及其他相关模型，主要介绍了如何利用sklearn进行特征工程，特征工程在机器学习中占有工程师的大部分精力，目前也有很多成熟的方法和理论，但是结合实际业务背景选择特征仍然是提升模型性能的关键点。sklearn.feature_selection是一个强大的特征处理工具包，磨刀不误砍柴工，熟练使用工具是重中之重！以下是特征工程的概要图。

![特征工程](https://img-blog.csdn.net/20170621085747545?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvTFlfeXN5czYyOQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

# 2 数据预处理

1. 不属于同一量纲：即特征的规格不一样，不能够放在一起比较。无量纲化可以解决这一问题。
2. 缺失值处理：包括缺失值删除及补充。

导入数据集，sklearn具有自动生成数据集工具包例如[sklearn.dataset.make_classification](http://sklearn.lzjqsdd.com/modules/generated/sklearn.datasets.make_classification.html#sklearn.datasets.make_classification)，也有标准数据集比如[sklearn.dataset.load_boston](http://sklearn.lzjqsdd.com/modules/generated/sklearn.datasets.load_boston.html#sklearn.datasets.load_boston),[sklearn.dataset.load_iris](http://scikit-learn.org/stable/modules/generated/sklearn.datasets.load_iris.html#sklearn.datasets.load_iris),下面是鸢尾花IRIS数据的下载程序

```
from sklearn.datasets import load_iris

#导入IRIS数据集
iris = load_iris()
#特征矩阵
dataset = iris.data

#目标向量
labels = iris.target

print "特征矩阵\n",dataset[:5,:]
print '标签\n',set(labels)123456789101112
```

```
特征矩阵
[[ 5.1  3.5  1.4  0.2]
 [ 4.9  3.   1.4  0.2]
 [ 4.7  3.2  1.3  0.2]
 [ 4.6  3.1  1.5  0.2]
 [ 5.   3.6  1.4  0.2]]
标签
set([0, 1, 2])
123456789
```

## 2.1 数据无量纲化

无量纲化使不同规格的数据转换到同一规格。常见的无量纲化方法有标准化和区间缩放法。标准化的前提是特征值服从正态分布，标准化后，其转换成标准正态分布。区间缩放法利用了边界值信息，将特征的取值区间缩放到某个特定的范围，例如[0, 1]等。

### 2.1.1 标准化

标准化需要计算**每个特征**的均值和标准差，公式表达为： 
x′=x−X¯Sx′=x−X¯S

使用preproccessing库的StandardScaler类对数据进行标准化的代码如下： 
标准化之后的数据范围在[-1,1]之间

至于为什么用.fit_transform(),可参考<http://www.cnblogs.com/jasonfreak/p/5448462.html>

```
from sklearn.preprocessing import StandardScaler

#标准化，返回值为标准化后的数据,矩阵形式，下面只显示了前10行数据
StandardScaler().fit_transform(dataset)[:10,:]1234
```

```
array([[-0.90068117,  1.03205722, -1.3412724 , -1.31297673],
       [-1.14301691, -0.1249576 , -1.3412724 , -1.31297673],
       [-1.38535265,  0.33784833, -1.39813811, -1.31297673],
       [-1.50652052,  0.10644536, -1.2844067 , -1.31297673],
       [-1.02184904,  1.26346019, -1.3412724 , -1.31297673],
       [-0.53717756,  1.95766909, -1.17067529, -1.05003079],
       [-1.50652052,  0.80065426, -1.3412724 , -1.18150376],
       [-1.02184904,  0.80065426, -1.2844067 , -1.31297673],
       [-1.74885626, -0.35636057, -1.3412724 , -1.31297673],
       [-1.14301691,  0.10644536, -1.2844067 , -1.4444497 ]])
1234567891011
```

### 2.1.2 归一化处理

标准化与归一化的**区别**： 
简单来说， 
**标准化**是依照特征矩阵（每一列是用同一个特征的不同取值）的列处理数据，其通过求z-score的方法，将样本的每个特征的值转换到同一量纲下。 
**归一化**是依照特征矩阵（每一行是不同特征的取值）的行处理数据，其目的在于样本向量在点乘运算或其他核函数计算相似性时，拥有统一的标准，也就是说都转化为“单位向量”。

规则为L2L2范数的归一化公式如下： 
x′=x∑j=1mx2j√x′=x∑j=1mxj2

使用preproccessing库的Normalizer类对数据进行归一化的代码如下：

```
from sklearn.preprocessing import Normalizer

#归一化，返回值为归一化后的数据的符号与原数据符号相同
Normalizer().fit_transform(dataset)[:10,:]1234
```

```
array([[ 0.80377277,  0.55160877,  0.22064351,  0.0315205 ],
       [ 0.82813287,  0.50702013,  0.23660939,  0.03380134],
       [ 0.80533308,  0.54831188,  0.2227517 ,  0.03426949],
       [ 0.80003025,  0.53915082,  0.26087943,  0.03478392],
       [ 0.790965  ,  0.5694948 ,  0.2214702 ,  0.0316386 ],
       [ 0.78417499,  0.5663486 ,  0.2468699 ,  0.05808704],
       [ 0.78010936,  0.57660257,  0.23742459,  0.0508767 ],
       [ 0.80218492,  0.54548574,  0.24065548,  0.0320874 ],
       [ 0.80642366,  0.5315065 ,  0.25658935,  0.03665562],
       [ 0.81803119,  0.51752994,  0.25041771,  0.01669451]])
1234567891011
```

### 2.1.3 区间缩放法

区间缩放法的思路有多种，常见的一种为利用两个最值进行缩放，公式表达为： 
x′=x−MinMax−Minx′=x−MinMax−Min

使用preproccessing库的MinMaxScaler类对数据进行区间缩放的代码如下：区间缩放之后数据范围在[0,1]之间

```
from sklearn.preprocessing import MinMaxScaler

#区间缩放，返回值为缩放到[0, 1]区间的数据
MinMaxScaler().fit_transform(dataset)[:10,:]1234
```

```
array([[ 0.22222222,  0.625     ,  0.06779661,  0.04166667],
       [ 0.16666667,  0.41666667,  0.06779661,  0.04166667],
       [ 0.11111111,  0.5       ,  0.05084746,  0.04166667],
       [ 0.08333333,  0.45833333,  0.08474576,  0.04166667],
       [ 0.19444444,  0.66666667,  0.06779661,  0.04166667],
       [ 0.30555556,  0.79166667,  0.11864407,  0.125     ],
       [ 0.08333333,  0.58333333,  0.06779661,  0.08333333],
       [ 0.19444444,  0.58333333,  0.08474576,  0.04166667],
       [ 0.02777778,  0.375     ,  0.06779661,  0.04166667],
       [ 0.16666667,  0.45833333,  0.08474576,  0.        ]])
1234567891011
```

## 2.2 缺失值处理

1. 直接除去含有缺失值的样本点
2. 缺失值填补（其中sklearn、numpy、pandas等python工具包都有相应处理方法），在本次博客中，只介绍sklearn工具包中的处理方法(imputer类)，numpy和pandas会在其他博客中给出。

由于IRIS数据包中并没有缺失值，这里会给出人工数据集的测试用例,处理缺失值的一般步骤如下所示： 
\1. 使用字符串’nan’来代替数据集中的缺失值； 
\2. 将该数据集转换为浮点型便可以得到包含np.nan的数据集； 
\3. 使用sklearn.preprocessing.Imputer类来处理使用np.nan对缺失值处理过的数据集。

```
import numpy as np
from sklearn.preprocessing import Imputer

#含有空格的数据
tem='1,2,3, ,3,4,5,6,7,8, ,9'
print tem

#替换空格字符
tem = tem.replace(' ','nan').split(',')
print tem
#将数据转换成浮点型矩阵形式
x = np.array(tem,dtype = float).reshape((4,3))
print x
1234567891011121314
```

```
1,2,3, ,3,4,5,6,7,8, ,9
['1', '2', '3', 'nan', '3', '4', '5', '6', '7', '8', 'nan', '9']
[[  1.   2.   3.]
 [ nan   3.   4.]
 [  5.   6.   7.]
 [  8.  nan   9.]]
1234567
```

```
Imputer().fit_transform(x)1
```

```
array([[ 1.        ,  2.        ,  3.        ],
       [ 4.66666667,  3.        ,  4.        ],
       [ 5.        ,  6.        ,  7.        ],
       [ 8.        ,  3.66666667,  9.        ]])
12345
```

Imputer()的参数有： 
\1. missing_values：默认是（(default=”NaN”)） 
\2. strategy ：string字符格式, optional (default=”mean”)，一般有mean（均值），median（中位数），most_frequent(众数) 
\3. axis:轴向，默认是（列向，axis =0），行是（axis=1） 
可通过help指令显示对Imputer的解释

```
help(Imputer())1
```

# 3 特征构建

特征构建包括以下内容： 
\1. 基于统计方法构造统计量 
\2. 对某些连续定量特征进行离散化，以便去除冗余信息 
\3. 对定性特征进行哑编码，将其转换成定量特征 
\4. 特征交叉

## 3.1 基于统计方法的构造统计量

该方法更多的是针对时间序列数据，例如，一段时间的广告点击量的单位时间均值，一段时间，某个用户对某个商品的浏览总数等。

## 3.2 对某些连续定量特征离散化，以便去除冗余信息

### 3.2.1 对连续定量数据二值化

定量特征二值化的核心在于设定一个阈值，大于阈值的赋值为1，小于等于阈值的赋值为0，公式表达如下： 

x={1,x>threshold0,x≤thresholdx={1,x>threshold0,x≤threshold

使用preproccessing库的Binarizer类对数据进行二值化的代码如下：

```
from sklearn.preprocessing import Binarizer

#二值化，阈值设置为3，返回值为二值化后的数据
Binarizer(threshold = 3).fit_transform(dataset)[:10,:]1234
```

```
array([[ 1.,  1.,  0.,  0.],
       [ 1.,  0.,  0.,  0.],
       [ 1.,  1.,  0.,  0.],
       [ 1.,  1.,  0.,  0.],
       [ 1.,  1.,  0.,  0.],
       [ 1.,  1.,  0.,  0.],
       [ 1.,  1.,  0.,  0.],
       [ 1.,  1.,  0.,  0.],
       [ 1.,  0.,  0.,  0.],
       [ 1.,  1.,  0.,  0.]])
1234567891011
```

## 3.3 定性特征哑编码

特征更多的时候是分类特征，而不是连续的数值特征。

比如一个人的特征可以是`[“male”, “female”]`， [“from Europe”, “from US”, “from Asia”]， [“uses Firefox”, “uses Chrome”, “uses Safari”, “uses Internet Explorer”]。 这样的特征可以高效的编码成整数，

例如 [“male”, “from US”, “uses Internet Explorer”]`可以表示成`[0, 1, 3]，

[“female”, “from Asia”, “uses Chrome”]`就是`[1, 2, 1]。

这个的整数特征表示并不能在scikit-learn的估计器中直接使用，因为这样的连续输入，估计器会认为类别之间是有序的，但实际却是无序的。(例如：浏览器的类别数据则是任意排序的)。

一个将分类特征转换成scikit-learn估计器可用特征的可选方法是使用one-of-K或者one-hot编码,该方法是:class:OneHotEncoder的一个实现。该方法将每个类别特征的 “m” 可能值转换成”m”个二进制特征值，当然只有一个是激活值。

```
from sklearn.preprocessing import OneHotEncoder
array = np.array([[0.1,0,3],[1,1,0],[0.7,2,1],[1,0,2]])
print array
print OneHotEncoder().fit_transform(array).toarray()1234
```

```
[[ 0.1  0.   3. ]
 [ 1.   1.   0. ]
 [ 0.7  2.   1. ]
 [ 1.   0.   2. ]]
[[ 1.  0.  1.  0.  0.  0.  0.  0.  1.]
 [ 0.  1.  0.  1.  0.  1.  0.  0.  0.]
 [ 1.  0.  0.  0.  1.  0.  1.  0.  0.]
 [ 0.  1.  1.  0.  0.  0.  0.  1.  0.]]
123456789
```

```
help(OneHotEncoder)1
```

**注意** 
\1. 该编码方法输入矩阵的数据应为浮点型会整型，当为浮点型时，会自动转为整型，直接保留整数位数据 
\2. 在矩阵array中，共有4行3列，假设每一列对应一个特征，则第一列有两个可能取值（属性）：0，1；第二列有三个可能取值：0，1，2；第三列可能取值：0，1，2，3；这样一来，第一列（第一个特征）可以用2位的二进制码来表示，第二列可以用3位二进制码来表示，第四列用4位二进制码来表示；因此会产生2+3+4=9列，从结果中可以看出第一列第一位0.1=1，0（二进制），第二列第一位0=1，0，0，第三列第一位3=0，0，0，1 
\3. 从结果中还可以看出，编码之前会对每一列的属性值进行排序，属性值对应的编码值，第一位为最小属性值，最后一位为最高属性值。

## 3.4 特征交叉

交叉从理论上而言是为了引入特征之间的交互，也即为了引入非线性性。举个简单的例子，在特征为性别的属性值为对不同种类广告的CTR（广告点击率），当然还有其他方式，包括CTR与年龄，CTR与地域，下面介绍一种多项式特征构建方法

很多情况下，多项式特征是通过考虑输入数据中的非线性特征来增加模型的复杂性，它能捕捉到特征中高阶和相互作用的项。 :class:`PolynomialFeatures`类中可以实现该功能，其代码实现如下所示：

```
from sklearn.preprocessing import PolynomialFeatures
x =  np.arange(6).reshape(3,2)
print x123
```

```
[[0 1]
 [2 3]
 [4 5]]
1234
```

```
poly = PolynomialFeatures()#默认输入两个参数1
```

```
help(poly)1
```

```
poly.fit_transform(x)1
```

```
array([[  1.,   0.,   1.,   0.,   0.,   1.],
       [  1.,   2.,   3.,   4.,   6.,   9.],
       [  1.,   4.,   5.,  16.,  20.,  25.]])
1234
```

上述执行过程是在input_feature = (x1,x2),output_feature = (1,x1,x2,x21,x22,x1x2)(1,x1,x2,x12,x22,x1x2),参数degree =2，interaction_only = False

```
x = np.arange(9).reshape(3, 3)
poly = PolynomialFeatures(degree = 3,interaction_only = True)
poly.fit_transform(x)123
```

```
array([[   1.,    0.,    1.,    2.,    0.,    0.,    2.,    0.],
       [   1.,    3.,    4.,    5.,   12.,   15.,   20.,   60.],
       [   1.,    6.,    7.,    8.,   42.,   48.,   56.,  336.]])
1234
```

上述执行过程是在input_feature = (x1,x2,x3),output_feature = (1,x1,x2,x3,x1x2,x1x3,x2x3,x1x2x3)(1,x1,x2,x3,x1x2,x1x3,x2x3,x1x2x3),参数degree = 3,interaction_only = True

注意多项式特征被隐含地使用在核方法

# 4 特征选择

特征选择主要有三种方法： 
\1. Fliter 过滤法按照发散性或者相关性对各个特征进行评分，设定阈值或者待选择阈值的个数，选择特征。 
\2. Wrapper：包装法，根据目标函数（通常是预测效果评分），每次选择若干特征，或者排除若干特征。 
\3. Embedded：嵌入法，先使用某些机器学习的算法和模型进行训练，得到各个特征的权值系数，根据系数从大到小选择特征。类似于Filter方法，但是是通过训练来确定特征的优劣。

## 4.1 Fliter 过滤法

### 4.1.1 方差选择法

使用方差选择法，先要计算各个特征的方差，然后根据阈值，选择方差大于阈值的特征。使用feature_selection库的VarianceThreshold类来选择特征，是特征选择中的一项基本方法。它会移除所有方差不满足阈值的特征。默认设置下，它将移除所有方差为0的特征，即那些在所有样本中数值完全相同的特征（类似常数特征）。

```
from sklearn.feature_selection import VarianceThreshold

print np.cov(dataset.T)
print dataset[:5,:]
#参数threshold为方差的阈值,该方法会移除方差小于3的特征，人为选择阈值具有主观性
VarianceThreshold(threshold=3).fit_transform(dataset)[:5,:]123456
```

```
[[ 0.68569351 -0.03926846  1.27368233  0.5169038 ]
 [-0.03926846  0.18800403 -0.32171275 -0.11798121]
 [ 1.27368233 -0.32171275  3.11317942  1.29638747]
 [ 0.5169038  -0.11798121  1.29638747  0.58241432]]
[[ 5.1  3.5  1.4  0.2]
 [ 4.9  3.   1.4  0.2]
 [ 4.7  3.2  1.3  0.2]
 [ 4.6  3.1  1.5  0.2]
 [ 5.   3.6  1.4  0.2]]

array([[ 1.4],
       [ 1.4],
       [ 1.3],
       [ 1.5],
       [ 1.4]])
1234567891011121314151617181920
```

可以看到，特征方差为对角线上元素，只有一个特征方差大于3，最后的结果中也至于一列被选出来。

### 4.1.2 相关系数方法

使用相关系数法，先要计算各个特征对目标值的相关系数以及相关系数的P值。用feature_selection库的SelectKBest类结合相关系数来选择特征的代码如下：

```
from sklearn.feature_selection import SelectKBest
from scipy.stats import pearsonr


#选择K个最好的特征，返回选择特征后的数据
#第一个参数为计算评估特征是否好的函数，该函数输入特征矩阵和目标向量，输出二元组（评分，P值）的数组，
#数组第i项为第i个特征的评分和P值。在此定义为计算相关系数
#参数k为选择的特征个数

print np.array(map(lambda x:pearsonr(x,iris.target),dataset.T))

#print dataset.T

SelectKBest(lambda X,Y:np.array(map(lambda x:pearsonr(x,Y),X.T))[:,0],k=2)\
.fit_transform(dataset,iris.target)[:10,:]
12345678910111213141516
```

```
[[  7.82561232e-01   2.89047835e-32]
 [ -4.19446200e-01   9.15998497e-08]
 [  9.49042545e-01   4.15547758e-76]
 [  9.56463824e-01   4.77500237e-81]]

array([[ 1.4,  0.2],
       [ 1.4,  0.2],
       [ 1.3,  0.2],
       [ 1.5,  0.2],
       [ 1.4,  0.2],
       [ 1.7,  0.4],
       [ 1.4,  0.3],
       [ 1.5,  0.2],
       [ 1.4,  0.2],
       [ 1.5,  0.1]])
1234567891011121314151617181920
```

```
help(pearsonr)1
```

```
Help on function pearsonr in module scipy.stats.stats:

pearsonr(x, y)
    Calculates a Pearson correlation coefficient and the p-value for testing
    non-correlation.

    The Pearson correlation coefficient measures the linear relationship
    between two datasets. Strictly speaking, Pearson's correlation requires
    that each dataset be normally distributed, and not necessarily zero-mean.
    Like other correlation coefficients, this one varies between -1 and +1
    with 0 implying no correlation. Correlations of -1 or +1 imply an exact
    linear relationship. Positive correlations imply that as x increases, so
    does y. Negative correlations imply that as x increases, y decreases.

    The p-value roughly indicates the probability of an uncorrelated system
    producing datasets that have a Pearson correlation at least as extreme
    as the one computed from these datasets. The p-values are not entirely
    reliable but are probably reasonable for datasets larger than 500 or so.

    Parameters
    ----------
    x : (N,) array_like
        Input
    y : (N,) array_like
        Input

    Returns
    -------
    r : float
        Pearson's correlation coefficient
    p-value : float
        2-tailed p-value

    References
    ----------
    http://www.statsoft.com/textbook/glosp.html#Pearson%20Correlation
12345678910111213141516171819202122232425262728293031323334353637
```

### 4.1.3 卡方检验

经典的卡方检验是检验定性自变量对定性因变量的相关性。由于它最初是由英国统计学家Karl Pearson在1900年首次提出的，因此也称之为Pearson χ2，其计算公式为.假设自变量有N种取值，因变量有M种取值，考虑自变量等于i且因变量等于j的样本频数的观察值与期望的差距，构建统计量： 
χ2=∑i=1k(Ai−npi)2npi(i=1,2,...,k)χ2=∑i=1k(Ai−npi)2npi(i=1,2,...,k)

其中，Ai为i水平的观察频数，Ei为i水平的期望频数，n为总频数，pi为i水平的期望频率。i水平的期望频数Ei等于总频数n×i水平的期望概率pi，k为单元格数。当n比较大时，χ2统计量近似服从k-1(计算Ei时用到的参数个数)个自由度的卡方分布。 
这个[统计量的含义简而言之就是自变量对因变量的相关性](http://wiki.mbalib.com/wiki/%E5%8D%A1%E6%96%B9%E6%A3%80%E9%AA%8C)。用feature_selection库的SelectKBest类结合卡方检验来选择特征的代码如下：

```
from sklearn.feature_selection import SelectKBest
from sklearn.feature_selection import chi2
print dataset[:5,:]
SelectKBest(chi2,k=2).fit_transform(dataset,labels)[:5,:]1234
```

```
[[ 5.1  3.5  1.4  0.2]
 [ 4.9  3.   1.4  0.2]
 [ 4.7  3.2  1.3  0.2]
 [ 4.6  3.1  1.5  0.2]
 [ 5.   3.6  1.4  0.2]]

array([[ 1.4,  0.2],
       [ 1.4,  0.2],
       [ 1.3,  0.2],
       [ 1.5,  0.2],
       [ 1.4,  0.2]])
12345678910111213141516
```

```
help(chi2)1
```

```
Help on function chi2 in module sklearn.feature_selection.univariate_selection:

chi2(X, y)
    Compute chi-squared stats between each non-negative feature and class.

    This score can be used to select the n_features features with the
    highest values for the test chi-squared statistic from X, which must
    contain only non-negative features such as booleans or frequencies
    (e.g., term counts in document classification), relative to the classes.

    Recall that the chi-square test measures dependence between stochastic
    variables, so using this function "weeds out" the features that are the
    most likely to be independent of class and therefore irrelevant for
    classification.

    Read more in the :ref:`User Guide <univariate_feature_selection>`.

    Parameters
    ----------
    X : {array-like, sparse matrix}, shape = (n_samples, n_features_in)
        Sample vectors.

    y : array-like, shape = (n_samples,)
        Target vector (class labels).

    Returns
    -------
    chi2 : array, shape = (n_features,)
        chi2 statistics of each feature.
    pval : array, shape = (n_features,)
        p-values of each feature.

    Notes
    -----
    Complexity of this algorithm is O(n_classes * n_features).

    See also
    --------
    f_classif: ANOVA F-value between label/feature for classification tasks.
    f_regression: F-value between label/feature for regression tasks.
1234567891011121314151617181920212223242526272829303132333435363738394041
```

### 4.1.4 互信息法

经典的互信息也是评价定性自变量对定性因变量的相关性的，互信息计算公式如下： 
I(X;Y)=∑x∈X∑y∈Yp(x,y)logp(x,y)p(x)p(y)I(X;Y)=∑x∈X∑y∈Yp(x,y)logp(x,y)p(x)p(y)

为了处理定量数据，最大信息系数法被提出，使用feature_selection库的SelectKBest类结合最大信息系数法来选择特征的代码如下：

```
from sklearn.feature_selection import SelectKBest
from minepy import MINE

#和求相关系数方法类似
#由于MINE的设计不是函数式的，定义mic方法将其为函数式的，
#返回一个二元组，二元组的第2项设置成固定的P值0.5
def mic(x,y):
    m = MINE()
    m.compute_score(x,y)
    return [m.mic(),0.5]
# 显示各个特征与目标变量的熵值
feature_mic = []
for i in range(len(dataset.T)):
    feature_mic.append(mic(dataset[:,i],labels))
print np.array(feature_mic)
print dataset[:5,:]
#选择K个最好的特征，返回特征选择后的数据
SelectKBest(lambda X,Y:np.array(map(lambda x:mic(x,Y),X.T))[:,0],k=2)\
.fit_transform(dataset,labels)[:5,:]12345678910111213141516171819
```

```
[[ 0.6421959   0.5       ]
 [ 0.40150415  0.5       ]
 [ 0.91829583  0.5       ]
 [ 0.91829583  0.5       ]]
[[ 5.1  3.5  1.4  0.2]
 [ 4.9  3.   1.4  0.2]
 [ 4.7  3.2  1.3  0.2]
 [ 4.6  3.1  1.5  0.2]
 [ 5.   3.6  1.4  0.2]]





array([[ 1.4,  0.2],
       [ 1.4,  0.2],
       [ 1.3,  0.2],
       [ 1.5,  0.2],
       [ 1.4,  0.2]])
1234567891011121314151617181920
```

### 4.1.5 总结

以上filter过滤法，可以解释成单变量特征选择方法，所求的方差，相关系数，卡方，互信息都是针对某个变量或者是某个变量与目标变量的关系。这样，变量之间的相关性会被忽视，另外，最好特征个数的选取，以及方差阈值选择具有主观性，不过可通过超参数调优来获取最佳性能。

## 4.2 Wrapper 方法

### 4.2.1 递归特征消除法

对于一个为数据特征指定权重的预测模型（例如，线性模型对应参数coefficients），递归特征消除 (RFE)通过递归减少考察的特征集规模来选择特征。首先，预测模型在原始特征上训练，每项特征指定一个权重。之后，那些拥有最小绝对值权重的特征被踢出特征集。如此往复递归，直至剩余的特征数量达到所需的特征数量。 
RFECV 通过交叉验证的方式执行RFE，以此来选择最佳数量的特征。

RFE 实现代码如下所示

```
from sklearn.feature_selection import RFE
from sklearn.linear_model import LogisticRegression

#递归特征消除法，返回特征选择后的数据
#参数estimator为基模型
#参数n_features_to_select为选择的特征个数
print dataset[:5,:]
RFE(estimator = LogisticRegression(),n_features_to_select = 2).fit_transform(dataset,labels）[:5,:]123456789
```

```
[[ 5.1  3.5  1.4  0.2]
 [ 4.9  3.   1.4  0.2]
 [ 4.7  3.2  1.3  0.2]
 [ 4.6  3.1  1.5  0.2]
 [ 5.   3.6  1.4  0.2]]

array([[ 3.5,  0.2],
       [ 3. ,  0.2],
       [ 3.2,  0.2],
       [ 3.1,  0.2],
       [ 3.6,  0.2]])
12345678910111213141516
```

RFECV代码实现如下所示：

```
from sklearn.feature_selection import RFECV
from sklearn.linear_model import LogisticRegression
from sklearn.cross_validation import StratifiedKFold

RFECV(LogisticRegression(),cv= StratifiedKFold(labels,2))\
.fit_transform(dataset,labels)[:5,:]123456
```

```
array([[ 3.5,  1.4,  0.2],
       [ 3. ,  1.4,  0.2],
       [ 3.2,  1.3,  0.2],
       [ 3.1,  1.5,  0.2],
       [ 3.6,  1.4,  0.2]])
123456
```

## 4.3 Embedded

### 4.3.1 基于惩罚项的特征选择法

该方法主要通过类SelectFromModel实现，该类作为一个基转换器，可以用于拟合后任何拥有coef_或feature_importances_属性的预测模型。 
在参数L1惩罚项降维的原理在于保留多个对目标值具有同等相关性的特征中的一个，所以没选到的特征不代表不重要。故，可结合L2惩罚项来优化。具体操作为：若一个特征在L1中的权值为1，选择在L2中权值差别不大且在L1中权值为0的特征构成同类集合，将这一集合中的特征平分L1中的权值，故需要构建一个新的逻辑回归模型：

```
from sklearn.feature_selection import SelectFromModel
from sklearn.linear_model import LogisticRegression

print "前五个样本的特征矩阵\n",dataset[:5,:]
#带L1惩罚项的逻辑回归作为基模型的特征选择
print "所选特征矩阵\n",SelectFromModel(LogisticRegression(penalty='l1',C =0.1)).fit_transform(dataset,labels)[:5,:]123456
```

```
前五个样本的特征矩阵
[[ 5.1  3.5  1.4  0.2]
 [ 4.9  3.   1.4  0.2]
 [ 4.7  3.2  1.3  0.2]
 [ 4.6  3.1  1.5  0.2]
 [ 5.   3.6  1.4  0.2]]
所选特征矩阵
[[ 5.1  3.5  1.4]
 [ 4.9  3.   1.4]
 [ 4.7  3.2  1.3]
 [ 4.6  3.1  1.5]
 [ 5.   3.6  1.4]]
12345678910111213
```

[结合L1，L2的逻辑回归](http://www.cnblogs.com/jasonfreak/p/5448385.html)

[不同算法使用基于L1的特征选择进行文档分类的对比](http://sklearn.lzjqsdd.com/auto_examples/text/document_classification_20newsgroups.html#example-text-document-classification-20newsgroups-py)

### 4.3.2 基于树模型的特征选择法

sklearn中基于树模型的预测模型包括[sklearn.tree](http://sklearn.lzjqsdd.com/modules/classes.html#module-sklearn.tree)和[sklearn.ensembel](http://sklearn.lzjqsdd.com/modules/classes.html#module-sklearn.ensemble),在这里使用feature_selection库的SelectFromModel类结合GBDT模型，来选择特征的代码如下：

```
from sklearn.feature_selection import SelectFromModel
from sklearn.ensemble import GradientBoostingClassifier

#GBDT作为基模型的特征选择
SelectFromModel(GradientBoostingClassifier()).fit_transform(dataset,labels)[:5,:]
```

```
array([[ 1.4,  0.2],
       [ 1.4,  0.2],
       [ 1.3,  0.2],
       [ 1.5,  0.2],
       [ 1.4,  0.2]])
123456
```

# 5 特征提取

这里特征提取，可以考虑如下应用背景，图像像素矩阵的特征提取、多维特征矩阵中存在相关特征子集等。此外，当特征选择完成后，可以直接训练模型了，但是可能由于特征矩阵过大，导致计算量大，训练时间长的问题，因此降低特征矩阵维度也是必不可少的。常见的降维方法除了以上提到的基于L1惩罚项的模型以外，另外还有主成分分析法（PCA）和线性判别分析（LDA），线性判别分析本身也是一个分类模型。PCA和LDA有很多的相似点，其本质是要将原始的样本映射到维度更低的样本空间中，但是PCA和LDA的映射目标不一样：[PCA是为了让映射后的样本具有最大的发散性](http://www.cnblogs.com/LeftNotEasy/archive/2011/01/08/lda-and-pca-machine-learning.html)；而[LDA是为了让映射后的样本有最好的分类性能](http://www.cnblogs.com/LeftNotEasy/archive/2011/01/08/lda-and-pca-machine-learning.html)。所以说PCA是一种无监督的降维方法，而LDA是一种有监督的降维方法。

## 5.1 主成分分析法（PCA）

使用decomposition库的PCA类选择特征的代码如下：

```
from sklearn.decomposition import PCA

#主成分分析法，返回降维后的数据
#参数n_components为主成分数目
print "变换后的特征矩阵选取前两个主元\n",PCA(n_components=2).fit_transform(dataset)[:5,:]12345
```

```
变换后的特征矩阵选取前两个主元
[[-2.68420713  0.32660731]
 [-2.71539062 -0.16955685]
 [-2.88981954 -0.13734561]
 [-2.7464372  -0.31112432]
 [-2.72859298  0.33392456]]
1234567
```

PCA是对原始特征矩阵的一种投影变换，所得到的是在新空间的得分主元，该主元是将原始特征矩阵中具有较大方差变化信息提取到主元中，并用主要作为模型的输入变量。

## 5.2 线性判别分析（LDA）

sklearn有专门的库来解决该问题，代码如下：

```
from sklearn.lda import LDA

#线性判别分析法，返回降维后的数据
#参数n_components为降维后的维数
LDA(n_components =2).fit_transform(dataset,labels)[:5,:]12345
```

```
array([[-8.0849532 ,  0.32845422],
       [-7.1471629 , -0.75547326],
       [-7.51137789, -0.23807832],
       [-6.83767561, -0.64288476],
       [-8.15781367,  0.54063935]])
123456
```

# 6 参考文献

[使用sklearn做单机特征工程](http://www.cnblogs.com/jasonfreak/p/5448385.html) 
[sklearn官方文档特征选择(Feature selection)](http://sklearn.lzjqsdd.com/modules/feature_selection.html#feature-selection) 
[机器学习之特征工程](http://blog.csdn.net/dream_angel_z/article/details/49388733)