#### 调参

**eta**, **number of trees**, **ax_depth**, **min_child_weight**,  **gamma**, **subsample**, **colsample_bytree** **之间的联系**

1. 选择较高的学习速率(learning rate)。一般情况下，学习速率的值为0.1。但是，对于不同的问题，理想的学习速率有时候会在0.05到0.3之间波动。选择对应于此学习速率的理想决策树数量。XGBoost有一个很有用的函数“cv”，这个函数可以在每一次迭代中使用交叉验证，并返回理想的决策树数量。
2. 为了确定学习速率和决策树数量，对决策树特定参数(max_depth, min_child_weight, gamma, subsample, colsample_bytree)调优。
3. xgboost的正则化参数的调优。(lambda, alpha)。这些参数可以降低模型的复杂度，从而提高模型的表现。
4. 降低学习速率，确定理想参数。



**理解**：决策树数量和决策树深度 呈反比，一般决策树深度约大，那么也就意味着更少的决策树数量。越浅越简单的决策树，就需要更多的数量来达到相似的效果



#### feature importance

```python
def get_score(self, fmap='', importance_type='weight'):
	"""Get feature importance of each feature.
       Importance type can be defined as:
           'weight' - the number of times a feature is used to split the data across all trees.
           'gain' - the average gain of the feature when it is used in trees
           'cover' - the average coverage of the feature when it is used in trees
	"""
	...
```

**理解**: 默认是  用所有生成树中，哪个feature split越多哪个feature越重要。但是每棵树的feature作为节点的次数不同，第一棵树的权重最高，如果后面的树feature和第一课树的feature节点区别较大，那么feature的 importance 会有偏差。