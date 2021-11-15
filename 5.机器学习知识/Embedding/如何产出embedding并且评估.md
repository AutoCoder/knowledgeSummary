##### word2vec和item2vec两个部分，前者有较多的比较成熟的度量方案，后者则基本上没有统一认可的方案

##### word2vec

Relatedness：目前大部分工作都是依赖wordsim353等词汇相似性数据集进行相关性度量，并以之作为评价word embedding质量的标准。这种评价方式对数据集的大小、领域等属性很敏感。

Analogy : task也就是著名 A - B = C - D 词汇类比任务（所谓的 analogy task，如 king – queen = man – woman）

Categorization：看词在每个分类中的概率

聚类算法：kmeans 聚类，查看聚类分布效果 。若向量维度偏高，则对向量进行降维，并可视化。如使用pca，t-sne等降维可视化方法



三种方式：语义特性（不适应item2vec）、**用作特征**、**用作初始值** （后两种以线上收益为准）

**item2vec**

1. 抽样人工判别
2. 降维可视化
3. 按照先验类目来判断 （类目app的相似度）

