rank i2i

LTR  = Learning to rank

为啥搜索的排序算法要考虑用pairwise的样本，而推荐的却往往用pointwise？

pairwise 训练对搜索是有用的，对推荐的作用较小

搜索是带 query 的、有意识的被动推荐，对于搜索而言，相关性是及其重要的事情

推荐是发散的、无意识的主动推荐，不太考虑相关性，反而考虑多样性。不关心两个结果的比较。反而是召回的目标是找到item之间的相关性，比较适合pairwise的建模。

搜索一般是一次给出多个结果，只是排序不一样。

推荐某些情况是可出一个或者少量结果，所以比较关系这些少量结果之间的较量

搜索的正负样本采集方式：

1 点击正样本

2 展示出来的未点击负样本

