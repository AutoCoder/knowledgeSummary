本质区别，一个是bagging， 一个是boosting

集成方式 ： 一个是投票（由于bagging每个基分类器是独立训练的，所以它是可以并行的）， 一个是迭代相加 （后一棵树需要等待前一颗树构建完毕）

基分类器的倾向：

boosting的方法，由于新的基分类器会在前面的基础上，所以偏差相对较小，需要更加关注方差（不同的训练子集之间）， 所以每个基分类器模型不能太复杂，所以一般选择深度较浅的树（一般<=6层）

bagging的方法，由于每个树都是独立的，而且最终的结果是等权投票的，所以方差的问题不大，所以每个基分类器更加关注自身的偏差，所以一般倾向深度较深的树（一般20~30层）, 一般来说，树的增加不会导致过拟合加重，比较容易实现方差和偏差最优的tradeoff

