title:Stock Prediction
date:2015-6-10
category:技术

# Stock Prediction
## Using Machine Learning And Bigdata



# Outline
* Pair Trade
* 涨幅预测
* 股指预测

# Pair Trade
* define : 在价值规律的前提下，寻找价值相背离的一对股票， 做对冲操作。
* key Point : Pair distance的准确性。

# Statistical Pair Trade 
* 采用推荐中的协同过滤公式, 计算整个市场每一只股票的相似度
* 寻找相似股票中， 近期价值偏离的一对pair
* 进行对冲操作
* Problem:难以确认最合适的相似度公式

# Machine Learning Pair Trade
* 将所有的相似度公式作为一个feature
* 利用第二天的偏离作为label
* 直接learn出各个相似度公式的权重

# Machine Learning 涨幅预测
* data source : 股票市场数据，微博、雪球等ugc数据，资讯数据，电商数据
* Feature Engineering: 领域知识特征, n-gram、nn等feature Engineering
* algorithm : MART, Learning To Rank, FTRL, BOPR 

# Data Source
* 原生金融相关数据，包括上市公司的表现， 股票本身的表现
* 金融资讯：来源于资讯数据的抓取， 关键词分析
* 网民的上网数据： 关键词分析
* 标志性的电商数据: 行业表现

依托于odps， 我们可以轻松的存储使用这部分全量数据。

# Feature Engineering
* 领域知识的feature构建, 有明确的金融含义的特征
* n-gram、word2vec : Feature generation  
* nn : 利用多层nn寻找隐藏的factor
* Tree Feature Transformation


# Regression Or Ranking
* 准确的预测绝对值， 大部分情况下正确率是比较低的
* Ranking问题的可预测性则高一点
* "我不知道哪只股票会涨， 但我知道哪只股票涨的比别的多"
* 利用learning To Rank对股票进行ranking

# Method
* learning To Rank : 将两只股票的delta作为loss Function
* 绝对值binary化: 涨幅10%等价于10个positive
* loss:
    $$min \frac{1}{2}||w||^{2}+C\sum\xi _{s}$$
     $$ s.t. w^{T}(x_{s}^{+}-x_{s}^{-})\geq 1-\xi _{s} $$

# Frame
![整体架构](../images/stock.png)
