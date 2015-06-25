title:Recovering 方法
date:2014-11-20 20:00
category:机器学习
tags:最优化


对于很高的dimension， 比较常用的时random variable， 老金表示利用一下转化效果会更好。
paper参见：recovering the opt solution by dual random projection

```matlab
V=randn(1000,20)*randn(20,100)+randn(1000,100)*0.001; 
[Q, R] = qr(V * randn(100, 30), 0);
[U, S, B] = svds(Q' * V, 30);
A = Q * U;
sumsqr(V-A*S*B')
```

* V是自己造的一个数据，rank是20
* V * randn(100, 30） 是在做那个random proj
* qr(...)是做qr decomposition

