FM 是基于LR的升级版本，考虑了特征的二阶交叉。

$$\displaystyle Y_{fm} = w_0 + \sum_{i=1}^{n}w_ix_i + \sum_{i}^{n-1}\sum_{j=i+1}^{n}w_{ij}x_ix_j$$  

但是由于对于高维特征的二阶交叉非常稀疏，没有足够的样本用于训练， 所以对$W$进行了改进解决了交叉项参数训练不充分的问题

令交叉项的    $$\hat{W} = VV^T $$ 

$$V = \begin{bmatrix}
 v_{11}  &v_{12} &... &v_{1k} \\ 
 v_{21}  &v_{22} &... &v_{2k} \\ 
 ...  &...  &... &...\\ 
 v_{n1}  &v_{n2} &... &v_{nk} 
\end{bmatrix}_{n\times k} = \begin{bmatrix}
 v_1 \\ v_2 \\ 
 ... \\ v_n 
\end{bmatrix} $$

$$\hat{W} = \begin{bmatrix}
v_1 \\ 
v_2 \\ 
... \\ 
v_n 
\end{bmatrix} \times \begin{bmatrix} v_1^{T} &v_2^{T}  &...  &v_n^{T} \end{bmatrix}$$ 

$$\displaystyle Y_{fm} = w_0 + \sum_{i=1}^{n}w_ix_i + \sum_{i}^{n-1}\sum_{j=i+1}^{n}<v_i,v_j>x_ix_j$$  

$\hat{W}$ 为对称矩阵，$\hat{W}$ 可以从复杂度$O(n^2)$ 降低为$O(kn)$

$$\displaystyle \sum_{i}^{n-1}\sum_{j=i+1}^{n}<v_i,v_j>x_ix_j $$    其中  $ \displaystyle< v_i,v_j> = \sum_{f=1}^{k}v_{ik}v_{jk}$

$$ \displaystyle = \frac{1}{2}\sum_{i=1}^{n}\sum_{j=1}^{n}<v_i,v_j>x_ix_j - \frac{1}{2}\sum_{i=1}^{n}<v_i,v_i>x_ix_i $$  // 简化为矩阵数值和 - 对角线元素的 一半

$$\displaystyle = \frac{1}{2}(\sum_{i=1}^{n}\sum_{j=1}^{n}\sum_{f=1}^{k}v_{if}v_{jf}x_ix_j - \sum_{i=1}^{n}\sum_{f=1}^{k}v_{if}v_{if}x_ix_i)$$

$$\displaystyle = \frac{1}{2}\sum_{f=1}^{k} (\sum_{i=1}^{n}\sum_{j=1}^{n}v_{if}v_{jf}x_ix_j - \sum_{i=1}^{n}v_{if}v_{if}x_ix_i)$$

$$\displaystyle = \frac{1}{2}\sum_{f=1}^{k} ((\sum_{i=1}^{n}v_{if}x_i)(\sum_{i=1}^{n}v_{if}x_i) - \sum_{i=1}^{n}v_{if}^2x_i^2)$$

通过化简， 推理的时间复杂度降低为 $O(kn)$， 训练的时间复杂度降低为 $O(mnk)$ , m 为训练样本数。

**缺点**

在做特征组合的时候，我们不确定是同一域内额特征相组合，比如男性[1,0]这两个维度数据,如果组合的是性别本身这两个维度,就没有意义了.

**FFM使同一个ID类特征one-hot产生的所有特征都归为同一个field，也就是同一个域的特征不会相互交叉**

$$\displaystyle Y_{ffm} = w_0 + \sum_{i=1}^{n}w_ix_i + \sum_{i}^{n-1}\sum_{j=i+1}^{n}<v_{i,f_j},v_{j,f_i}>x_ix_j$$ 

FFM将特征按照事先的规则分为多个场(Field)，特征 ![[公式]](https://www.zhihu.com/equation?tex=x_i) 属于某个特定的场F。每个特征将被映射为多个隐向ield量 ![[公式]](https://www.zhihu.com/equation?tex=V_%7Bi1%7D%2C%E2%80%A6%2CV_%7Bif%7D) ，每个隐向量对应一个场。当两个特征 ![[公式]](https://www.zhihu.com/equation?tex=x_i%2C+x_j) ,组合时，用对方对应的场对应的隐向量做内积：

相比于 FM 的 nk 个二次项参数，FFM有 nkf 个二次项参数，学习和表达能力也更强

