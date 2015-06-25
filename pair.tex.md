title:Down Sampling Pair Wise
date:2014-09-12 10:54
category:机器学习

# Down Sampling PairWise
> Written by haoyuan hu

# introduce
* PairWise Loss:
$ L_{rank} (f;S) = \frac {1} {mn} \sum_{j=1}^{m}\sum_{i=1}^{n} 1{\hskip -2.5 pt}\hbox{I}(f(x_i)^+ \le (f(x_j^-)) $

* Down Sampling PairWise Loss:
$ L_{rank} (f;S) = \frac {1} {n} \sum_{i=1}^{n} 1{\hskip -2.5 pt}\hbox{I}(f(x_i)^+ \le rand(f(x^-)) $


# function
云梯2（odps环境）：benhua_fastpair

# example
``` sql
create table t_benhua_brand_pair as
select benhua_fastpair(user_id,  features, labels) as (features, label) 
from r_user_prefer_raw_trainset 
where 
action="click" 
and pt =20140824 
and domain='brand_id' 
and bc_type=1  
distribute by user_id
sort by user_id;
```

# data format
* input table : 必须包含bucket id（大部分情况下为user id）， 特征（features）， target（labels）三项， 样例数据如下：
 
uid|features|label
--|--|--
134158497  | alipaytotal^B1510.58^Acollecttotal^B9012.4| 0


* output table : 组对结果， 样例数据如下：

features   | label    
------------|------------
followbrandtotal60^B2.66^Aclickalipayrate90^B0.015^A| 1

# TODO
正在整合进入离线训练评估框架， 届时可以直接指定pairwise的参数调用数据准备、训练、预测、评估过程。


