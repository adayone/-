title:learning to rank
date:2014-09-12
category:机器学习

# 历史阶段

1.  Optimizing search engines using clickthrough data, T Joachims, KDD 02——ranksvm的提出。
2.  Efficient algorithms for ranking with SVMs, O Chapelle——提出primal SVM提升效率。
3.  Person Re-Identification by Support Vector Ranking, BMVC2010——提出Ensemble RankSVM解决memory consumption问题。

# svm算法描述
svm求解的目标是能够正确划分数据集并且几何间隔最大的分离超平面， 与感知机的求解误分类最小的策略有所不同。我们这里仅以线性svm为例进行介绍。

对于超平面(w, b), 样本点$(x_i, y_i)$, 几何间隔定义为：

$\gamma_i = y_i (\frac{w}{||w||} \cdot x_i + \frac{b}{||w||})$

超平面(w, b)对于训练数据集T的几何间隔定义为：

$\gamma = min(\gamma_i)$


几何间隔最大化直观的解释是，以充分大的确信度对训练数据进行分类， 也就是说， 不仅能将正负实例点分开， 而且对最难分的实例点（离超平面最近的点）也有足够大的确定度将它们分开， 这样的超平面对未知的新实例有很好的分类与猜测能力。 
具体的， 这个问题可以表述为：
$max \ \gamma$

$s.t. y_i (\frac{w}{||w||} \cdot x_i + \frac{b}{||w||}) \ge \gamma$

经过变化后， 问题等价位求如下的最优化问题：

$min \frac {1}{2}||w||^2$

$s.t.    y_i(w \cdot xi + b) - 1 \ge 0$

现实中的问题很少能完全做到线性可分， 必然存在一些奇异点的干扰， 我们对每个样本点引进一个松弛变量$\xi$, 相应的， 问题等价为：
$min \frac{1}{2}||w||^2 + C\sum{\xi}$

$s.t.    y_i(w \cdot xi + b)  \ge 1 - \xi$

# ranksvm算法描述

如上所述，在svm中是要找到分类超平面使正负例的几何间隔最大化，在ranking问题中不存在绝对的正负例，而是要使得正确匹配的得分$x_{s}^{+}$大于错误匹配的得分$x_{s}^{-}$，即
$(x_{s}^{+}-x_{s}^{-})>0$,并且使得$(x_{s}^{+}-x_{s}^{-})$最大化。


定义查询样本x对于样本xi匹配的得分$x_{s}:  x_{s}=w^{T}|x-x_{i}| $; $|x-x_{i}|$表示每一维特征都对应相减，得到差值向量，预测阶段基于差值做出预测和排序。

# 训练过程
为了获得$w^{T}$，我们定义：

* 训练集$X={（xi，yi）}，i=1,2，……m；$
* $xi$为特征向量
* yi为+1或者-1
* 对于每一个训练样本xi，X中与xi匹配的正例样本构成集合$d_{i}^{+} = \lbrace x_{i,1}^{+},x_{i,2}^{+}...x_{i,m^{+}}^{+}  \rbrace$
* X中与xi匹不配的负例样本构成集合$d_{i}^{-}= \lbrace x_{i,1}^{-},x_{i,2}^{-}...x_{i,m}^{-}  \rbrace；m^{+},m^{-}$分别为正负样本数量，存在关系$m^{+}+m^{-}=m$。

归纳起来，就是对训练样本集中每个样本xi，都存在若干正负例的score pairwise，把训练集中所有score pairwise的集合用P表示，即$P={（x_{s}^{+},x_{s}^{-}）}$,作为训练阶段的输入;

综合之前讲述的svm最优化表述，RankSVM就可得到如下形式：
$ MAX w^{T}(x_{s}^{+}-x_{s}^{-});  $

$s.t. w^{T}(x_{s}^{+}-x_{s}^{-})>0$

即：
$ min \frac{1}{2}||w||^{2}+C\sum\xi _{s} $

$ s.t. w^{T}(x_{s}^{+}-x_{s}^{-})\geq 1-\xi _{s}。 $

以上就是RankSVM的基本思想，ranksvm目前广泛用于pair wise的训练方式。

# 优化

在实际应用中， postive和negative的数量都比较巨大的情况下， ranksvm几乎不可用， 按照工业界以能够处理所有的数据优先于设计精巧的模型原则， 我们修改了ranksvm的loss，使得他做到线性的复杂度。

* PairWise Loss:
$ L_{rank} (f;S) = \frac {1} {mn} \sum_{j=1}^{m}\sum_{i=1}^{n} 1{\hskip -2.5 pt}\hbox{I}(f(x_i)^+ \le (f(x_j^-)) $

* Down Sampling PairWise Loss:
$ L_{rank} (f;S) = \frac {1} {n} \sum_{i=1}^{n} 1{\hskip -2.5 pt}\hbox{I}(f(x_i)^+ \le rand(f(x^-)) $

通过这种方式， 能将复杂度从 $ 2 \cdot n \cdot m  $ 下降到 $2 \cdot n$。
同时在实践中表明， 由于能够处理的数据样本数目和维度大大提升， down sampling的方式表现优于标准的ranksvm。

# tips

* L1范数:  ||x|| 为x向量各个元素绝对值之和。公式：$|X|=\sum{|x_i|}$
* L2范数:  ||x||为x向量各个元素平方和的1/2次方，L2范数又称Euclidean范数或者Frobenius范数。公式：$||X||=\sqrt{\sum{x_i^2}})$
* 几何间隔：
* 样本点几何间隔：超平面(w, b)和样本点(xi, yi)的距离$|w \cdot x_i + b|$表示分类预测的确信程度， 而$w \cdot x_i + b$符号是否一致表示了分类是否正确， 做归一化之后表示为：$\gamma_i = y_i (\frac{w}{||w||} \cdot x_i + \frac{b}{||w||})$
* 平面几何间隔： 超平面(w, b)关于训练数据集T的几何间隔定义为： $\gamma = min(\gamma_i)$
* 支持向量：样本点中与超平面距离最近的点。


