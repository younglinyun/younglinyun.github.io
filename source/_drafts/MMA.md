---
title: MMA 和 GCMMA 笔记
tags:
---

## "标准格式"

$$
\begin{split}
\text{minimize} & f_0(\boldsymbol{x}) + a_0 z + \sum_{i=1}^m(c_iy_i + \frac{1}{2}d_iy_i^2) \\
\text{s.t.:}    & f_i(\boldsymbol{x}) - a_i z - y_i \le 0,\quad i=1,2,\ldots,m \\
                & \boldsymbol{x} \in \boldsymbol{X}, \boldsymbol{y} \ge 0, z \ge 0.
\end{split}
$$