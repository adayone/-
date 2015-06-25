title:tree feature transform
date:2014-11-19
category:机器学习

单纯的gbrt算法有较高的时间复杂度， 同时需要迭代获取最佳参数， 这方面lr更好。

但是， lr无法处理非线性的特征， 需要自己将非线性的特征映射为线性的特征。


所以， facebook采用了一种利用GBRT的叶子节点作为LR的输入特征的方式解决了这一问题。

paper：Practical Lessons from Predicting Clicks on Ads at Facebook.

# step 1
使用GBRT进行训练， 利用grid search获得最佳参数。

# step 2
对于GBRT的预测结果，首先对所有子树叶子节点编号，如果样本在第一棵子树落入2， 在第二棵子树落入4， 那么就输出[2, 4]，以此类推, 如果是稠密特征的话， 就是[0,1,0,1,0]。

![](http://orange-tree.oss-cn-hangzhou.aliyuncs.com/blog-images/IMG_0378.JPG)

# step 3
将预测数据的结果做为输入的feature， 运行线性模型进行训练预测， 产生最终的预测结果。

# improvement
一方面， 利用tree做feature transform可以消除correlation的影响， 另一方面以较低的代价获得了非线性的优势。

sample一部分数据进行GBRT训练还可以快速grid最佳参数。

# NEED
完成基于lr数据格式的训练和预测模块， 完成上述的级联封装过程。
