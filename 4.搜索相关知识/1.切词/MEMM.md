# MEMM

**MEMM**并没有像**HMM**通过联合概率建模，而是直接学习条件概率 $$P(s_{t}|s_{t-1}，o_{t})$$

MEMM与HMM的第二个假设不一样: 当前的<u>**状态**</u>仅仅与当前的<u>**观测**</u>有关，如图

![img](https://images2015.cnblogs.com/blog/399159/201612/399159-20161220111525011-885271482.png) 

根据最大熵MaxEnt 对 $$P(s_{t}|s_{t-1}，o_{t})$$ 进行建模得到 $$\displaystyle P(s|s',o) = \frac{ exp \left( \sum_a \lambda_a f_a(o,s) \right)}{ Z(o,s')}$$

$$\displaystyle Z(o, s^{'})  = \sum_{y}(exp(\sum_{a}\lambda_{a}f_{a}(o,s^{'}))) $$

其中，$$\lambda_{a}$$ 为学习参数；$$a=<b, s>$$ 且b为feature，s为destination state；特征函数$$f_{a}(o,s)$$的示例如下 :

![img](https://images2015.cnblogs.com/blog/399159/201612/399159-20161220111655839-605104853.png) 

其他的部分和HMM一样，也是通过Viterbi 算法来进行求解。

但是根据 $$\displaystyle P(s|s',o)​$$ 的定义，它是针对当前节点进行归一化的，也就是说它会出现标注偏置的问题，也就是说MEMM倾向于选择拥有更少转移的状态。

![img](https://pic4.zhimg.com/80/v2-40f9945cdffb12cfec84bebc7b7e3be5_hd.jpg) 