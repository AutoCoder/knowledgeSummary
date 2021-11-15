分享提纲：

RALM的核心诉求

RALM技术上的创新

爱奇艺用户理解的结合点



RALM的面对核心问题：
1. 新item是重要的内容，但是由于缺乏后验数据吃亏了：T+1的离线模型训练，不能及时的帮新item纳入进来，且新item展现和点击uv都不大，属于长尾内容，比较吃亏，一般只能靠策略来补偿新item
 	2. 多样性和长尾问题，规则匹配 -> 协同过滤 -> 线性模型 -> deep learning 以及对样本的采样，逐步缓解了长尾问题。但是仍然没有解决。



RALM的核心诉求：

1. **实时**：新 item 分发不需要重新训练模型，要能实时完成种子用户的扩展
2. **解决长尾 & 高效**：因为线上加到 rank 模型 CTR 的后面， 要保持模型核心指标 CTR 的前提下，再去加强长尾内容分发。（疑问？ 放到CTR排序模型后面，属于重排？前面那部保留了多少candidates，是否真的能解决长尾的问题）



RALM技术上的创新

整体思路：

​     RALM 摒弃了 user item label 这样一种训练模式， 而是把 user {item => seed user} label 这样一种模式， 把item 用 转换为这个item上有过正反馈的user embedding表达， 缓解了对热门item在训练中的偏爱。

​     具体怎么表征 seed user 和 target user 是RALM 中的一个重点， 包括：

- ​	attention机制的运用
- ​    global embedding 和local embedding的概念 （global embedding 仅仅对seed user 做self attention, local embedding 是 和target user做相似度打分进行attention）



![RALM](D:\工作相关\技术积累\knowledgeSummary\7. 知识分享\RALM.png)

最后输出为 $M = aH$

文中采用了tanh的激活函数，有点像加型的score function。

计算score的步骤为：

$u = tanh(W_1H)$

 $score = W_2u^T$ 

$a_i$ 为 alignment function的得到的权重值, $a_i = softmax(score)$,  $a_i$ 的维度为n(特征域的个数)，那么 $W_2$ 的维度为[1, k]

那么 $u$的维度只可能为  [n, k]

但是依照论文的论述，$W_1$ 形状为[k, n]， $H$的形状为[n，m]， 那么$u = tanh(W_1H)$ 的形状只能为 [k, m] 。 和后面计算的预期是矛盾的。

目前按照我的理解，$W_1$ 作为一个参数阵，形状是可以调整的。于是我令 $W_1$ 的形状为[m, k],  且 $u = tanh(HW_1)$ 则 u的形状为 [n，k],

$W_2$ 作为另外一个参数阵， 形状为[1, k],  则 score的形状为 [1, n]





问题：

​     

片外： 各种attention的前世今身    参考（https://zhuanlan.zhihu.com/p/53682800）





attention的经典三步：

- score function

  ![打分socre](https://pic3.zhimg.com/80/v2-981a0c9ab01531c7139e4701574cb056_720w.jpg)

  加性（additive）模型是线性变化后相加

  点积（multiplicative）模型，是矩阵乘法来变换 （缩放点积相对点积模型，是为了softmax阶段梯度大一点学习的快一点）

- alignment function

  - global/local attention   (价值不大，很少用)
    - global 就是一般的soft attention, local attention 是 alignment function 中做了一点儿调整，在 softmax 算出来的attention wieght 的基础上，加了一个以 pt 为中心的高斯分布来调整 alignment 的结果

- generate context vector function

  - hard attention 【理解】按照alignment function得到的权重对 输入向量进行采样
  - soft attention   【理解】按照alignment function得到的权重对 输入向量进行加权平均  （一般都用这个）



大部分的attention区别在于 第一步和第三步， 第二步基本都是softmax。

**比较著名的self-attention从这三个步骤开始解释：**

它的score function用的是缩放点积模型，alignment function 就是一般的 global attention， generate context vector function是 soft attention

它的点积是怎么算的呢？

![self-attention](https://pic4.zhimg.com/80/v2-fcc2df696966a9c6700d1476690cff9f)_720w.jpg



其中multihead-attention，是把输入$X$，通过self-attention转化出多组输出$Z_0$， 然后再线性变化得到最终的$Z$

![multihead-self-attention](https://pic3.zhimg.com/80/v2-00e535ee13e7a3e14651d3269d5fd2a2_720w.jpg)





**attention机制的运用**

从youtube-dnn 到 RALM：

![RALM_1](D:\工作相关\技术积累\knowledgeSummary\7. 知识分享\RALM_1.jpg)

![Deep_Neural_Network_for_youtube_recommendation_1](D:\工作相关\技术积累\knowledgeSummary\7. 知识分享\Deep_Neural_Network_for_youtube_recommendation_1.png)

这个图就是RALM 做user representation learning 的部分。这个图看起来和下图不太一样，其实结构上几乎是一样的。

唯一的区别在于，Merge Layer的不同。RALM的主要考虑相对与YoutubeDNN的变化是把concat layer变化成了带attention的layer。



**其他**

多个领域用户表示学习，然后stacking，这样学习比较充分



**实践过程**

问题 1：

user_embedding layer ，随着学习0值占比非常高，学习到最后达到100%，调低learning_rate 不能解决问题

解决1 ： 

最后nce_loss的中的nce_weight矩阵和nce_bias向量最初采用的是random_uniform_initializer均匀分布

修改成truncated_normal后， 该问题有效解决。



问题2：

youtubeDnn在训练时，loss下降的很慢

解决2：

调节learning_rate从0.001上升 0.1， 训练速度和质量达到最好，说明可以大胆尝试调高learning_rate



问题3：

从youtubeDnn 升级到 RALM以后，增加了attention layer， user_embedding layer的0值占比先上升，后下降，最后保持到40%以上，下降的非常缓慢

解决3：

尝试使用 exponential_decay learning_rate，但是没有起作用，user_embedding layer的0值占比最高达到60%，或者user_embedding layer的0值占比迅速下降到40%后不能进一步下降。loss没有持续下降

attention结构修改， 前后都会导致