title:线性规划
date:2011-03-23 10:00
category:机器学习
tags:lp, matlab

# function define

 
$$minf^Tx$$
$$A \cdot x \leq b$$
$$A_eq \cdot x = b_eq$$
$$lb \leq x \leq ub$$



函数形式:
linprog(f, A, b, Aeq, beq, lb, ub)

如果不存在就是[]

# problem

Find x that minimizes

f(x) = –5x1 – 4x2 –6x3,

subject to

x1 – x2 + x3 ≤ 20
3x1 + 2x2 + 4x3 ≤ 42
3x1 + 2x2 ≤ 30
0 ≤ x1, 0 ≤ x2, 0 ≤ x3.

# code

``` matlab
% 注意， 是列向量
f = [-5; -4; -6];
A =  [1 -1  1
3  2  4
3  2  0];
b = [20; 42; 30];
lb = zeros(3,1);
linprog(f,A,b,[],[],lb, [])
```

示例代码没有最后那个[],好像跑不过。

似乎返回值也没办法那么对， 原先是:
[x,fval,exitflag,output,lambda] = linprog(f,A,b,[],[],lb);

# 无约束非线性

还有一个无约束非线性的:fminunc

Minimize the function f(x)=3x


Create a file myfun.m:

function f = myfun(x)
f = 3*x(1)^2 + 2*x(1)*x(2) + x(2)^2;    % Cost function
Then call fminunc to find a minimum of myfun near [1,1]:

x0 = [1,1];
[x,fval] = fminunc(@myfun,x0);
