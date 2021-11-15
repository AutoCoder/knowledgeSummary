ABSTRACT
How to quickly and reliably learn the preferences of new users remains a key challenge in the design of recommender systems. In
this paper we introduce a new type of online learning algorithm, cluster-based bandits, to address this challenge. This exploits the
fact that users can often be grouped into clusters based on the similarity of their preferences, and this allows accelerated learning of
new user preferences since the task becomes one of identifying which cluster a user belongs to and typically there are far fewer
clusters than there are items to be rated. Clustering by itself is not enough however. Intra-cluster variability between users can
be thought of as adding noise to user ratings. Deterministic methods such as decision-trees perform poorly in the presence of such
noise. We identify so-called distinguisher items that are particularly informative for deciding which cluster a new user belongs to
despite the rating noise. Using these items the cluster-based bandit algorithm is able to efficiently adapt to user responses and rapidly
learn the correct cluster to assign to a new user.

如何快速可靠的学习到用户的兴趣偏好，是推荐系统中的一大挑战。这篇文章介绍一种在线学习的，基于聚类的多臂老虎机方法。它基于用户一般可以被划分成一些人群聚类（这些聚类具有类似的偏好）的现实假设，它加速了用户的偏好学习，因为它把任务转化成了聚类即判断用户属于哪个聚类，并且一般来说，仅仅只有少数几个人群聚类（相对推荐系统要排序的item的数量）。尽管如此 仅仅靠它自己的聚类是不够的。 聚类内部用户的多样性可以被当作用户排序时参杂的噪声，具有决定性的方法比如决策树一般在存在这种噪声时往往表现不好。

尽管在有排序干扰噪声的情况下，我们发现所谓的区分项对于判断用户属于哪个聚类具有十分强的指向性。 用这些区分项，基于聚类的多臂老虎机算法可以十分有效的利用用户的反馈，快速的学习到用户到底属于哪一个聚类。



In this paper we revisit the cold-start task for new users of a recommender system, which remains a core challenge. When a new user joins the system it initially has no knowledge of the preferences of the user and so would like to quickly learn these1. The recommender system therefore initially starts in an “exploration” phase where the first few items that it asks the new user to rate are chosen with the aim of discovering the user’s preferences. For brevity we focus on the simplest setup where a user explicitly rates items presented to them, e.g. on a 1-5 scale or binary like/dislike feedback,
and the aim of the recommender system is to predict other items that the user may like.

文章中，我们回顾了在推荐系统中对于新用户的冷启动任务， 是一个核心的挑战。当新用户加入，初始时推荐系统不知道他的兴趣偏好，所以它必须快速学习。因此最开始是一个探索阶段，在这个阶段，带着学习用户偏好的目的会推送少量的内容给新用户让他去打分。为了简单，我们利用用户显式打分做为一个简单的方法，比如1-5分，或者喜欢或者不喜欢。



One common approach to this new user cold-start task is to take ratings already collected from a population of users, use these to cluster users into groups and then train a decision-tree to learn a mapping from item ratings to the user group, see for example Figure 1(a). When a new user joins the system this decision-tree is used to decide which items the user is initially asked to rate and in this way the group to which the user belongs is initially estimated. Once the group is estimated, the system recommends items liked by members of that group e.g. using matrix factorisation or another
collaborative filtering approach.

一个常见的方法，是利用一定量用户打分，用这些去聚类人群，然后训练一个决策树根据新用户的打分来把用户分群，如图1(a)。 当新用户来的时候，决策树被用来决定给用户哪些内容去打分， 通过这种方式，用户属于哪个人群就被估计出来了。一旦人群被估计出来，系统就会推荐这个人群喜欢的内容， 采用的方法有矩阵分解，或者写协同过滤方法。



However, typically users clustered in the same group do not give identical ratings to an item. Rather there is a spread of ratings, and
this intra-cluster variability between users can be thought of as adding noise to the ratings. Unfortunately, decision trees can easily make mistakes in the face of such noise. For example, Figure1(b) shows the measured decision-tree accuracy for Netflix data clustered into 16 groups (see later for more details). It can be seen that the accuracy is as low as 50-60% for a number of groups

尽管如此，一个人群中的典型用户并不一定给出相同的打分对于同一个内容， 相反会存在不同的评分，并且这种人群内部的用户之间的多样性被认为给评分增加了干扰。不幸的的是，决策树很容易在这种情况下犯错。比如图1(b) 所示，决策树基于奈飞数据16分类的精度，可以看出有些分类的精度仅仅在50-60%。



Multi-arm bandits (MABs), and more generally online convex learning, has been the subject of much interest in recent years.
However, naive application of standard bandit algorithms to the cold-start task leads to poor performance. If we think of each recommender
system item as an arm of a MAB then we run into the difficulty that (i) there are many arms and so learning is slow and (ii) repeated pulls of the same arm tend to be highly correlated 2. One remedy is to associate an arm with each group rather than each item. For each group the available items are sorted in descending order of their predicted rating by users in that group. Pulling the arm for a group then corresponds to asking the user to rate
the next item from this sorted list, i.e. the unrated item predicted to have the highest rating for members of the group. While this greatly reduces the number of arms in the MAB, as we will see later the learning rate remains very slow. This is because items rated highly by members of one group tend to also be rated highly by members of at least some of the other groups, and so the user ratings for these items do not serve to strongly distinguish between groups and so allow rapid learning.

多臂老虎机和更通用的在线凸优化， 是近些年来的研究热点。尽管如此，标准老虎机算法得的简单应用对于冷启动问题来说效果不好。假设我们把每个推荐项考虑成一个老虎机的一个臂，我们面临2个问题： （1）有太多臂了，使得学习的非常慢 （2）重复拉相同臂会导致高相关性。 

一个补救方法是关联一个臂到一个人群，而不是一个推荐项。对于每个人群，所有的可选推荐项被按照他们在这个人群中的预测打分倒排。 拉对应人群的臂对应着让这个用户对倒排上的推荐项进行打分，没有被打过分的推荐项在被预测为该人群的最高分。同时当老虎机的臂大大减少以后我们仍会发现，学习的速度还是非常慢。 这个是因为，那些总是被一个人群评高分的推荐项也总是容易被另外几个人群评高分， 所以用户的这些评分不会带来不同人群之间的区分性以及快速学习效果。



In this paper we propose a novel cluster-based bandit algorithm that achieves fast learning in cold-start, e.g. for the standard Netflix
dataset < 10 items need to be rated in order to reliably distinguish between 16 user groups and < 12 items to reliably distinguish between
32 groups. We show that the group of a user is identified with significantly higher accuracy than with a decision-tree without
incurring higher regret i.e the learning performance is fundamentally superior to that of a decision-tree.

这篇文章，我们提出了一个新颖的基于聚类的多臂老虎机算法，实现了冷启动场景的快速学习。比如，对于标准的奈飞数据集，为了有效构建16个人群的区分性，少于10个推荐项需要被评分。为了有效构建32个人群的区分性，至少12个推荐项需要被评分。我们展出出来的，用户在被归类时的准确性显著高于决策树，并且没有导致任何其他副作用，也就是它的学校表现彻底由于决策树。



For a recent survey of solutions to the cold-start see [4, 5]. Passive approaches include recommending popular items, use of item based
recommendation (once user starts rating items an item-based approach is used to recommend similar items), transfer learning
from another recommender system previously used by a user, and asking new users to rate a fixed list of items. Examples of early
work on active learning include IGCN (information gain through clustered neighbors) which uses a decision tree with user clusters
as leaves [8] and the ternary decision-tree approach of [7]. More recently, [1] uses representative items i.e. after completing
the ratings matrix 𝑅, 𝑘 columns of 𝑅 are selected, the ratings of the other items are represented as a linear combination of these and
during cold start a new user is asked to rate these representative items. This approach is extended to use a decision-tree approach
by [9]. In [10] a matrix factorization approach is proposed whereby a decision-tree is trained to map from item ratings to the latent feature
vector for a user. Use of multi-arm bandits for cold-start has also received attention. In [6] after completing the ratings matrix 𝑅 its rows are clustered and the average ratings vector for each cluster is used as a representative user. During cold start an MAB is used to select the average ratings vector to use and the user is asked to rate next highest item in the vector – this is akin to the cluster-bandit algorithm with no exploration phase. In [2] a MAB
is used to select between recommender strategies, typically recommending popular items initially for a new user and later switching to a kNN or matrix factorisation model.

引用4,5 是一些观影冷启动问题的调研。一些比较有效的方法，包括推送热门内容，基于内容相关性的推荐， 迁移从另外一个推荐系统中学习的模型，以及直接询问新用户（给他们提供一些固定选项）。一些更早的工作，比如主动学习包括IGCN（information gain through clustered neighbors），这种方法使用了一个决策树和三元决策树方法。 更近期的一些研究，使用表示学习，在完成了评分矩阵R以后，其中K列被选择，其他列的评分被表达为这K列的线性组合并且在冷启动阶段，会让新用户不断对这些内容进行评分。这种方法扩展了使用决策树方法 By [9].  文章[10] 中，提出了一个矩阵分解的方法，通过一个决策树来映射内容评分到一个隐式特征向量给用户。 使用多臂老虎机算法在冷启动领域，也引起了注意力机制，文章[6]中，在完成评分矩阵R后，按行被聚类，行的打分均值为认做每一个聚类中的典型用户。在冷启动阶段，MAB被用来提取均分打分向量，让用户对最有可能的内容进行打分 - 这种方法接近于没有探测阶段的多臂老虎机。

文章[2] 中， MAB被用来选择推荐策略，比如是热门推荐给新用户，还是切换到KNN和矩阵分解模型。



3 CLUSTER-BASED BANDIT ALGORITHM
We have a set 𝐺 of user groups. Each user belongs to one group 𝑔 ∈ 𝐺. We also have a set of items 𝑉 . Given a new user our task is
to quickly learn which group they belong to by asking the user to rate items in 𝑉 , using the fact that the distribution of item ratings
varies depending on the user’s group.

假设，我们有个人群集合G， 所有人属于其中人群g。 我们有一个内容集合V。 当新用户来的时候，我们的任务是通过用户的打分行为快速学习这个用户属于那个人群。前提事实条件是，不同的人群，他们在内容上的评分具有区分性。



3.1 Group Indicator Vector
What we would like to measure is an indicator vector 𝐼 i.e. a vector for which as we collect more user ratings the element 𝐼 (𝑔) tends
towards 1 when a new user belongs to group 𝑔 and all other elements 𝐼 (ℎ), ℎ ≠ 𝑔 tend towards 0. We can obtain such a vector as follows. Start by defining：

我们想要计算的是一个0-1向量 $I(g)$，所有属于g人群的观影评分趋近于1， 同时其他人群h的观影内容则趋近于0。通过如下公式，我们可以得到这样的一个向量：

$$ \displaystyle R_n(g,h) = \sum_{i=1}^{n}\alpha_{i}^{n}(g,h) \frac{r(v_i) - \mu(h,v_i)}{\mu(g, v_i) - \mu(h,v_i)}$$,

其中$v_i$  是用户评分过推荐序列的第i项， $r(v_i)$ 是其评分，$\mu(g,v_i)$ 是人群g中所有人对$v_i$的评分均值，$\alpha_{i}^{n}(g,h)$ 是权重，其中$\displaystyle\sum_ {i=1}^{n}\alpha_{i}^{n}(g,h) =1$

$\mu(g,v_i)$ 的估计值是可知的，比如从现存的用户中的评分得到。注意，$R_n(g,h)$ 的计算仅要求用户对每个内容评分一次，当然了如果存在用户重复评分$v_i$序列会存在重复值。



To streamline notation suppose the new user is in group 1, we can always relabel the groups so that this holds3.  Then . 

$$ \displaystyle R_n(1,h) = \sum_{i=1}^{n}\alpha_{i}^{n}(1,h) \frac{r(v_i) - \mu(h,v_i)}{\mu(1, v_i) - \mu(h,v_i)}$$,

Intuitively, the deviations 𝑟 (𝑣𝑖 ) − 𝜇(1, 𝑣), 𝑖 = 1, . . . , 𝑛 tend to fluctuate around 0, as otherwise there would be a consistent offset between the user’s
ratings and the group ratings in which case the user would better be assigned to a different group. Therefore 𝑅𝑛 (1, ℎ) → 1 as 𝑛 → ∞ for ℎ ≠ 1. By the same argument, 𝑅𝑛 (𝑔, 1) → 0 as 𝑛 → ∞for 𝑔 ≠ 1. Hence,

$$I(g) = \underset{h\;\in G\;\backslash\;\{g\}}{min}R_n(g,h)$$

is an indicator vector of the desired form i.e. 𝐼 (1) → 1 and 𝐼 (𝑔) →0, 𝑔 ≠ 1 as 𝑛 → ∞. We then estimate the group $\hat{g}$ that the new user
belongs to using

$$\hat{g} \in \underset{h\;\in G\;\backslash\;\{g\}}{argmax} I(g) $$   

The key to fast learning is that 𝐼 (𝑔) converges quickly to a (0, 1) vector.

为了理解公式，假设新用户属于Group1。 有如下公式：

$$ \displaystyle R_n(1,h) = \sum_{i=1}^{n}\alpha_{i}^{n}(1,h) \frac{r(v_i) - \mu(h,v_i)}{\mu(1, v_i) - \mu(h,v_i)}$$,

直观的说，𝑟 (𝑣𝑖 ) − 𝜇(1, 𝑣)的差，应该在趋近于在0附近波动，也就是用户打分和所属人群的平均打分应该趋向一致。因此$R_n(1,h)$趋向于1，且$R_n(g,1), g \neq 1 $ 趋向于0。

用$$\hat{g} \in \underset{h\;\in G\;\backslash\;\{g\}}{argmax} I(g) $$    这个公式来估计新用户属于哪个group。快速学习的关键就在于如何让$I(g)$ 快速变成一个a(0,1) 的向量



Intuitively, an item 𝑣 helps to distinguish whether a user belongs to group 𝑔 rather than group ℎ when (i) the mean rating of 𝑣 by
users in group 𝑔 is very different from that of users in group ℎ i.e. (𝜇(𝑔, 𝑣) − 𝜇(ℎ, 𝑣))2 is large, and (ii) when the ratings tend to
be consistent/reliable i.e. the variance 𝜎2(𝑔, 𝑣) is small. That is, we expect that

直观的看，用户一个观影内容的评分接近人群g，而且非常异于人群h可以帮助判断他属于人群g

