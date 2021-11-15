# HMM

 $$i$$代表状态，$$o$$代表观测 

![img](https://pic4.zhimg.com/80/v2-d4077c2dbd9899d8896751a28490c9c7_hd.jpg) 

**假设1**：当前的状态之和前一时刻的状态有关

$$ P(i_{t}| i_{t-1}, o_{t-1}, ..., i_{1}, o_{1}) = p(i_{t}|i_{t-1}) $$

**假设2**：当前的观测仅仅与当前的状态有关

$$ P(o_{t}| i_{t}, o_{t-1}, i_{t-1} ..., i_{1}, o_{1}) = p(o_{t}|i_{t}) $$



HMM是用于对序列观测$$O$$做序列标注$$S$$的生成模型，用马尔可夫链（Markov chain）对联合概率$$P(S,O)$$建模： 

$$\displaystyle P(S,O) = \prod_{t} P(s_{t}|s_{t-1}) P(o_{t}|s_{t})$$



**维特比算法（Viterbi）**

StatusSet: 状态值集合

ObservedSet: 观察值集合

TransProbMatrix: 转移概率矩阵

EmitProbMatrix: 发射概率矩阵

InitStatus: 初始状态分布

基于这三个矩阵，利用动态规划的方法，找到联合概率最大的路径。

1. 算出初始节点的prob

2. 每经过一个观测节点，记录下各个状态的最大prob和对应的path

3. 当进入新的观测节点后，更新各个状态的最大prob和更新path，即永远只保留当前观测节点的最大prob和path

4. 一直到最终节点，找出prob最大的state，并取得其path。

   