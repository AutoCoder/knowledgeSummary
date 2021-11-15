**常规步骤**

Regression问题的常规步骤为：

1. 寻找h函数（即hypothesis）；
2. 构造J函数（损失函数）；
3. 想办法使得J函数最小并求得回归参数（θ）



**公式推导**

符号约定：

i-th train example : $$(x^{(i)}, y^{(i)})$$

i-th feature vector : $$X^{T}$$

i-th feature : $$x_{i}$$

i-th parameter: $$\theta _{i}$$

线性归回 hypothesis function ： $$h_{\theta }(X) = \theta ^{T}X$$

线性回归 loss function：$$J_{\theta }(X) = \frac{1}{m} \sum_{i=0}^{m} (\theta ^{T}x^{(i)} - y^{(i)})^{2}$$

最小二乘解（满秩 )  : $$\theta = (X^{T}X)^{-1}X^{T}Y$$

Sigmoid(Logistic )  function :   $$g(z) = \frac{1}{1 + e^{-z}}$$?

逻辑回归 hypothesis function ： $$h_{\theta }(X) = \frac{1}{1 + e^{-\theta ^{T}X}}$$

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20160730143626838) 

概率分布： 

$$P(Y=1|x;\theta) = h_{\theta }(X)$$

$$P(Y=0|x;\theta) = 1 - h_{\theta }(X)$$

转换后的概率分布：$$ \left\{\begin{matrix}P(Y=1|x;\theta) = h_{\theta }(X)\\ P(Y=0|x;\theta) = 1 - h_{\theta }(X) \end{matrix}\right.\qquad \Rightarrow\qquad  P(y|x;\theta) = (1 - h_{\theta }(x))^{1-y} * h_{\theta }(x)^{y}$$

极大似然估计函数（所有样本的联合概率） ： $$\displaystyle L(\theta ) = \prod_{i=0}^{m}P(y^{(i)}|x^{(i)};\theta )$$

即        $$\displaystyle L(\theta ) = \prod_{i=0}^{m}P(y^{(i)}|x^{(i)};\theta ) \qquad  \Rightarrow \qquad    \prod_{i=0}^{m}((1 - h_{\theta }(x^{(i)}))^{1-y^{(i)}} * h_{\theta }(x^{(i)})^{y^{(i)}})$$

转换为对数似然函数(同解问题) ： $$\displaystyle l(\theta) = logL(\theta ) = \sum_{i=0}^{m}(y^{(i)}log(h_{\theta }(x^{(i)})) + (1-y^{(i)})log(1 - h_{\theta }(x^{(i)})))$$

这是需要求解 $$L(\theta )$$  最大值。

变换为 $$-\frac{1}{m}l(\theta) $$ 求极小值



Q1 : 为什么  $$-\frac{1}{m}l(\theta) $$ 前面带负号？

A ： 因为联合概率$$L(\theta )$$的取值一定是[0,1) , 参考log函数曲线， $$l(\theta)$$的取值范围为(-∞ , 0), 所以带负号。

![img](http://d.hiphotos.baidu.com/zhidao/wh%3D600%2C800/sign=902e84aa15ce36d3a2518b360ac316bf/86d6277f9e2f0708e3a35270e924b899a901f218.jpg) 

