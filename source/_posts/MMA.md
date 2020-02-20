---
title: MMA 和 GCMMA 使用笔记
date: 2019-05-27 14:00:42
mathjax: true
tags:
categories:
  - 笔记
  - 最优化
---

## 通用格式

考虑如下优化问题，其中变量 $\boldsymbol{x} = (x_1,x_2,\ldots,x_n)^{\rm{T}}\in\mathbb{R}^n,~\boldsymbol{y} = (y_1,y_2,\ldots,y_m)^{\rm{T}}\in\mathbb{R}^m$ 以及 $z\in \mathbb{R}$. $n$ 为设计变量的维数，$m$ 为约束条件的数目。
<div>
$$
\begin{array}{rl}
\mathrm{minimize:} & \displaystyle f_0(\boldsymbol{x}) + a_0 z + \sum_{i=1}^m(c_iy_i + \frac{1}{2}d_iy_i^2) \\[1ex]
\mathrm{s.t.:}     & f_i(\boldsymbol{x}) - a_i z - y_i \le 0,\quad i=1,2,\ldots,m \\[1ex]
                   & \boldsymbol{x} \in \boldsymbol{X}, \quad\boldsymbol{y} \ge 0, \quad z \ge 0. \\
\end{array}
$$
</div>

其中 $\boldsymbol{X}$ 是由各变量 $x_j$ 满足其上下界构成的区域。各参量 $a,b,c,d$ 满足

$$ a_0>0, \quad a_i\ge 0, \quad c_i \ge 0, \quad d_i \ge 0, \quad c_i+d_i>0 $$

如果 $a_i>0$ 严格成立，还有 $a_ic_i>a_0$.

$x_1,x_2,\ldots,x_n$ 为自然设计变量，它们源自于优化问题本身；$y_1,y_2,\ldots,y_m$ 和 $z$ 则是人造设计变量，通过人为地选择对应的参数值能将通用格式（几乎）等价于各种类型的优化问题，如`最小二乘问题`和 `min-max` 等。

<!-- more -->

## 实际应用时的注意事项

### 目标函数和约束函数缩放

很多情况下约束都能用不等式来描述：

$$ g_i(\boldsymbol{x}) \le g_i^{\rm{max}} $$

比如要求应力值不能超过某个临界值。这种情况下 

$$f_i(\boldsymbol{x}) = g_i(\boldsymbol{x}) - g_i^{\rm{max}}$$

使用时应注意要事先放缩使得

$$ 1 \le g_i^{\rm{max}} \le 100 $$

而不要写成 $g_i^{\rm{max}}=10^{10}$ 这种大数。

目标函数也应放缩至

$$ 1 \le f_0(\boldsymbol{x}) \le 100 $$

设计变量放缩至

$$ 0.1 \le x_j^{\rm{max}} - x_j^{\rm{min}} \le 100 $$


### 人造参数取“大数”

对于如下标准最优化问题：

<div>
$$
\begin{array}{rl}
\mathrm{minimize:} & f_0(\boldsymbol{x})\\[1ex]
\mathrm{s.t.:}     & f_i(\boldsymbol{x}) \le 0,\quad i=1,2,\ldots,m \\[1ex]
                   & \boldsymbol{x} \in \boldsymbol{X}\\
\end{array}
$$
</div>

要使通用格式化为此标准问题，首先令 $a_0=1$ 及 $a_i>0,i>0$，则最优解中一定是 $z=0$。另外，令 $d_i=1$，$c_i$ 为一个“大数”，则变量 $y_i$ 在最优解中也应该为零。这样，通用格式的目标函数和约束函数都化为此标准问题的格式。

上面提到 $c_i$ 要取为一个“大数”，但应尽量避免将其量级取得过大，比如 $10^{10}$。最好是从一个“差不多大的值”开始，如果最优结果中有部分 $y_i=0$ 不成立，则再扩大一定倍数（如100），再重新求解。如果事先按照上一节做了缩放，则这里“差不多大的值”可取为 1000 或者 10000 这样的值。

### 设计变量上下限

在某些问题中，可能并没有对个别变量 $x_j$ 作出范围约束 $\displaystyle x_j^{\rm{min}} \le x_j \le x_j^{\rm{max}}$，则应人为给定某个范围，但是这个范围也不能取得无限制的大。尝试给定一个差不多大的区间范围，进行求解，如果结果显示 $x_j$ 在给出的区间端点，则适当扩大该区间重新求解，以此类推。


> K. Svanberg, MMA and GCMMA -- two methods for nonlinear optimization

> K. Svanberg, The method of moving asymptotes -- a new method for structural optimization,
International Journal for Numerical Methods in Engineering, 1987, 24, 359-373.

> K. Svanberg, A class of globally convergent optimization methods based on conservative
convex separable approximations, SIAM Journal of Optimization, 2002, 12, 555-573.
