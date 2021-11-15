# CRF

![img](https://pic2.zhimg.com/80/v2-714c1843f78b6aecdb0c57cdd08e1c6a_hd.jpg) 

[MMEM](http://www.cnblogs.com/en-heng/p/6201893.html)也是用最大熵模型建模$$P(Y|X)$$， MEMM和CRF都没有HMM的那两个独立性假设，可以引入unigram和bigram的特征函数，可以学习一些长程的相关性。

不同的是， **CRF**是其采用无向图模型，而MEMM只考虑$$o_{j}$$对$$s_{j}$$的影响，而没有把$O$作为整体来考虑，导致的是本地归一化。

下图是**CRF**的无向图：

![img](https://pic3.zhimg.com/80/v2-c5e2e782e35f6412ed65e58cdda0964e_hd.jpg) 

【最大团】

![img](https://pic4.zhimg.com/80/v2-1d8faeb71d690d02e110c7cd1d39eed3_hd.jpg) 

假设有如下无向图，那么$$(X_{1}, X_{3}, X_{4})$$和$$(X_{2}, X_{3}, X_{4})$$为其中的最大团

对于CRF链，它的最大团情况如下图

![ææ¯åäº"å¾ç](http://image.mamicode.com/info/201805/20180521141608014108.png) 

【势函数】 势函数是针对最大团的函数，![ \psi_{c}(Y_{c} )](https://www.zhihu.com/equation?tex=+%5Cpsi_%7Bc%7D%28Y_%7Bc%7D+%29) 是一个最大团 ![C](https://www.zhihu.com/equation?tex=C) 上随机变量们的联合概率 

​                        $$\displaystyle \psi_{c}(Y_{c} ) = e^{-E(Y_{c})} =e^{\sum_{k}\lambda_{k}f_{k}(c,y|c,x)}$$

【概率无向图的联合概率分布】

​                        $$\displaystyle P(Y )=\frac{1}{Z(x)} \prod_{c}\psi_{c}(Y_{c}) $$

​                        $$\displaystyle P(Y | X)=\frac{1}{Z(x)} \prod_{c}\psi_{c}(Y_{c}|X ) = \frac{1}{Z(x)} \prod_{c} e^{\sum_{k}\lambda_{k}f_{k}(c,y|c,x)} = \frac{1}{Z(x)} e^{\sum_{c}\sum_{k}\lambda_{k}f_{k}(y_{i},y_{i-1},x,i)}$$   

【归一化因子】                                  

​                        $$\displaystyle Z(x) = \sum_{Y} \prod_{c}\psi_{c}(Y_{c} )$$       

【CRF建模公式】

​                        $$\displaystyle P(I | O)=\frac{1}{Z(O)} \prod_{c}\psi_{i}(I_{i}|O ) = \frac{1}{Z(O)} \prod_{c} e^{\sum_{k}\lambda_{k}f_{k}(O,I_{i-1},I_{i},i)} = \frac{1}{Z(O)} e^{\sum_{c}\sum_{k}\lambda_{k}f_{k}(O,I_{i-1},I_{i},i)}$$



**[MEMM]**  $$\displaystyle P(s|s',o) = \frac{ exp \left( \sum_a \lambda_a f_a(o,s) \right)}{ Z(o,s')}$$

以上公式和MEMM的形式很像。  ![P(I|O)](https://www.zhihu.com/equation?tex=P%28I%7CO%29) 这个表示什么？具体地，表示了在给定的一条观测序列 ![O=(o_{1},\cdots, o_{i})](https://www.zhihu.com/equation?tex=O%3D%28o_%7B1%7D%2C%5Ccdots%2C+o_%7Bi%7D%29) 条件下，我用CRF所求出来的隐状态序列 ![I=(i_{1},\cdots, i_{i})](https://www.zhihu.com/equation?tex=I%3D%28i_%7B1%7D%2C%5Ccdots%2C+i_%7Bi%7D%29) 的概率序列，所以这个是全局归一化的，不会引起标注偏置的问题。



参考文献：

1. [CRF++ 特征模板的使用参考](http://www.52nlp.cn/%E4%B8%AD%E6%96%87%E5%88%86%E8%AF%8D%E5%85%A5%E9%97%A8%E4%B9%8B%E5%AD%97%E6%A0%87%E6%B3%A8%E6%B3%954) 
2. [【中文分词】条件随机场CRF](http://www.cnblogs.com/en-heng/p/6214023.html)

