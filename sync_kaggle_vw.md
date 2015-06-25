title:Vowpal Wabbit
date:2015-03-30 16:00
category:机器学习
tags:vw,kaggle

## 简介

为了避免空洞的语法说明， 以kaggle预测餐厅收入的事例说明。

## 数据格式
数据格式有namespace的概念， 不同namespace之间会做n-gram的feature engineing。

数据表示都是kv格式，所以需要做一下转换。

回归数据格式:

```shell
5653753.0 | OpenDate=07171999 City=İstanbul CityGroup=Big Cities Type=IL |P1:4 P2:5.0 P3:4.0 P4:4.0 P5:2 P6:2 P7:5 P8:4 P9:5 P10:5 P11:3 P12:5 P13:5.0 P14:1 P15:2 P16:2 P17:2 P18:4 P19:5 P20:4 P21:1 P22:3 P23:3 P24:1 P25:1 P26:1.0 P27:4.0 P28:2.0 P29:3.0 P30:5 P31:3 P32:4 P33:5 P34:5 P35:4 P36:3 P37:4 
6923131.0 | OpenDate=02142008 City=Ankara CityGroup=Big Cities Type=FC |P1:4 P2:5.0 P3:4.0 P4:4.0 P5:1 P6:2 P7:5 P8:5 P9:5 P10:5 P11:1 P12:5 P13:5.0 P14:0 P15:0 P16:0 P17:0 P18:0 P19:3 P20:2 P21:1 P22:3 P23:2 P24:0 P25:0 P26:0.0 P27:0.0 P28:3.0 P29:3.0 P30:0 P31:0 P32:0 P33:0 P34:0 P35:0 P36:0 P37:0 
```

## 训练命令调用

简单的难以置信：

```shell
vw -d vw_train_v3 -c \
--passes 1000 \
-f model.vw.v6 \
--ngram 5 \
-l 0.0001 \
--l2 0.1 \
--bfgs \
--readable_model text.model.v6
```

* -d : 输入训练数据
* -c: 建立缓存
* -f : 输出模型文件
* --ngram:feature做ngram
* -l : learning rate
* --l2 : l2的正则，数据小的时候设置大一点
* --bfgs : 使用bfgs的bathc model
* --readable_model：可读版本的模型

## 大概性能
不带任何参数跑， 十几秒， 干掉两个benchmark， 包括rf的。

迭代了几轮参数， 轻松200多名了。

## AUC等评测
[perf](http://osmot.cs.cornell.edu/kddcup/software.html)

因为我们并不需要预测结果， 只是用来做auc评估， 所以语法如下:

``` shell
vw -d vw_test -t -i model.vw -r /dev/stdout | perf -roc -files true_label /dev/stdin
```

## 用途

偏好小数据可以快速迭代参数， 找出适合的feature方向。

