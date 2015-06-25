title:combinar
date:2014-11-20 20:00
category:机器学习
tags:算法优化


在做预估的过程中， 其实存在着ctr、cvr两个目标， 解决大致有几种方式:

* pairwise, 以成交>点击的组pair方式完成
* multitask learing
* combinar

combinar一直是一个比较耐操的东西， 在某些情况下， 简单的均值就能得到不错的效果（很多并行化的方式就是求均值）。

另一个问题， 离线版本的训练是天级别的，假设消费者存在购买的一周周期律， 那么就会存在对天的过拟合：比如在周五的效果就特别好。

相比之前的pairwise， combinar方式效果莫名的好， 利用训练体系自动产生多天的ctr、cvr版本， 对weight求均值。
