---
title: 离散傅里叶变换求解常微分方程
date: 2019-05-20 16:07:40
mathjax: true
tags:
  - FFT
  - 结构动力学
categories:
  - 笔记
  - 数值计算
---

## 单位脉冲响应函数

单自由度有阻尼系统在初始时刻静止，单位脉冲的作用下其运动方程为

$$ m\ddot{x} + c\dot{x} + kx = \delta(t) $$

或者也可写为

$$ \ddot{x} + 2n\dot{x} + \omega_n^2 x = \frac{1}{m}\delta(t) $$

<!-- more -->

脉冲作用的一瞬间（$t=0\rightarrow 0^+$），有加速度 $\ddot{x}=\frac{1}{m}\delta(t)$，积分可得脉冲作用瞬间之后的初速度 $\dot{x}_0$ 和初位移 $x_0$：

$$
\dot{x}_0 = \int_0^{0^+}\ddot{x}(t)\mathrm{d}t=\frac{1}{m}, \quad
x_0       = \int_0^{0^+}\dot{x}(t)\mathrm{d}t = 0
$$

脉冲作用完毕之后系统在初始条件下作自由振动，可解得其响应为

<div>
$$
x(t) = \left\{
\begin{array}{ll}
0 & t<0 \\
\displaystyle\frac{\exp(-\zeta\omega_n t)}{\sqrt{mk(1-\zeta^2)}}\sin\omega_dt & t > 0 \\
\end{array}	
\right.
$$
</div>

阻尼比 $\displaystyle\zeta=\frac{c}{2\sqrt{km}}$，有阻尼固有频率 $\omega_d=\omega_n\sqrt{1-\zeta^2}$。

这个响应即称为脉冲响应函数，也用 $h(t)$ 来表示。


## 单位脉冲响应函数与频响函数的关系

### 原理

我们都知道 $h(t)$ 和 $H(\omega)$ 是傅里叶变换对的关系，这里还是给出验证过程

<div>
$$
\begin{split}
 H(\omega) &= \mathcal{F}(h(t)) = \int_{\infty}^{+\infty} h(t)e^{-i\omega t}\mathrm{d}t = \int_0^{+\infty} h(t)e^{-i\omega t}\mathrm{d}t \\
           &= \frac{1}{\sqrt{mk(1-\zeta^2)}} \frac{\omega_d}{\omega_d^2-(\omega-i\zeta\omega_n)^2} = \frac{1}{k-m\omega_n^2+ic\omega} \quad (\omega\in\mathbb{R})
\end{split}
$$
</div>

反过来，在计算逆变换的时候，$t<0$ 和 $t\ge 0$ 两个区间得到不同的积分结果，也正是 $h(t)$ 的分段表达式。利用 `Mathematica` 可以很快得到该结果

```mathematica
Integrate[
omegad / (omegad^2-(omega-I*zeta*omegan)^2) * Exp[I*omega*t], {omega,-Infinity,Infinity},
Assumptions -> {omegan>0,omegad>0,zeta>0,t>0}
]
```

### 数值计算中的实际考量

傅里叶正变换

$$ \mathcal{F}(f(t)) = \int_{-\infty}^{+\infty} f(t)e^{-i\omega t} \mathrm{d}t 
   \approx \sum_j f(t_j) e^{-i\omega t_j} \Delta t = \mathrm{fft(f_{data})} \Delta t
$$

傅里叶逆变换
<div>
$$ 
\begin{split}
\mathcal{F}^{-1}(F(\omega)) &= \frac{1}{2\pi} \int_{-\infty}^{+\infty} F(\omega)e^{i\omega t} \mathrm{d}\omega 
   \approx \frac{1}{2\pi} \sum_j F(\omega_j) e^{i\omega_j t} \cdot 2\pi\Delta f  \\
   &= \sum_j F(\omega_j) e^{i\omega_j t} \frac{F_s}{n} = F_s \cdot \mathrm{ifft(F_{data})}
\end{split}
$$
</div>

上式中 $F(\omega)$ 应该在实轴正负两端都取值，但通常只给出正半轴的数据，因此这种情况下要额外再乘 2。


## 应用算例

考察外激励力为高斯脉冲

$$ f(t) = \exp(-(t-4)^2) $$

时，此单自由度的位移响应。我们首先求出激励的傅里叶变换 $F(\omega)$，然后得到响应的傅里叶变换 $X(\omega)$，最后由傅里叶逆变换得到 $x(t)$。为了验证结果的正确性，我们同时还用 `ode45` 直接求解了这个方程，两种方法得到的结果列于下图中，可以看到吻合度非常高。

值得注意的是：频率分辨率的高低直接影响求解结果的准确度，特别是初始阶段。提高频率分辨通过延长采样数据，即在原始的时域数据之后补零的方式来实现。

<center>
<img src="/images/SDOFresponse.png" width="75%" height="75%" />
图 高斯脉冲激励及其响应
</center>

附上 `Matlab` 代码：

```matlab
clear
clc
close all

% 系统参数
m = 1;
k = 1;
zeta = 0.05;
c = 2*zeta*sqrt(k*m);
omegan = sqrt(k/m);
omegad = omegan*sqrt(1-zeta^2);

% 离散采样
Fs = 10;
n  = 200*Fs;
dt = 1/Fs;
t  = 0:dt:(n-1)*dt;
freq = 0:Fs/n:(n-1)/n*Fs;
omega = 2*pi*freq;

% 傅里叶变换求解响应
f = exp(-(t-4).^2);
F = fft(f)*dt;
H = 1 ./ (k-m*omega.^2 + 1i*c*omega);
X = H .* F;
x = ifft(X) * 2*Fs;

% ODE45 求解微分方程（认为是精确解）
[tt,y] = ode45(@(t,y) myfun(t,y,m,k,c),[0,t(end)],[0,0]);

% 结果对比
subplot(2,1,1)
plot(t,f,'b-')
xlim([0,10])
ylabel('$f(t)$','interpreter','latex')

subplot(2,1,2)
plot(tt,y(:,1),'b-')
hold on
plot(t,x,'b.')
xlim([0,50])
legend('直接求解','傅里叶变换')
xlabel('$t$','interpreter','latex')
ylabel('$x(t)$','interpreter','latex')


function dy = myfun(t,y,m,k,c)
    dy = zeros(2,1);
    dy(1) = y(2);
    dy(2) = (exp(-(t-4)^2) - k*y(1) - c*y(2))/m;
end
```

## 补充说明

### 初始条件

上述杜哈梅积分过程得到的是系统`零初始响应`，如果初始条件不为零，则在 `ifft` 逆变换之后还需要额外加上`零输入响应`。

> 邱吉宝. 计算结构动力学[M]. 中国科学技术大学出版社, 2009.

### 无阻尼系统

回到杜哈梅积分的表达式

$$
x(t) = \int_0^t f(\tau)h(t-\tau)\mathrm{d}\tau
$$

(1). 由于在计时开始前 ($t<0$) 并没有外力作用，上式的积分下限可延伸至 $-\infty$；

(2). 而脉冲响应函数 $h(t-\tau)$ 的宗量 $t-\tau<0$ 时 $h$ 也为 0，所以积分上限又可以延伸至 $+\infty$.

因此 

$$x(t) = \int_{-\infty}^{+\infty} f(\tau)h(t-\tau)\mathrm{d}\tau = f(t) * h(t) $$

只有在 $f(t)$ 和 $h(t)$ 的傅里叶变换都存在的情况下上式才可以进一步在变换至频域内相乘，而无阻尼系统 $h(t)$ 不满足绝对可积的条件，不能直接 $\displaystyle\mathcal{F}(h(t))=\frac{1}{k-m\omega^2}$。正确的做法应该是回到 $[0,t]$ 区间的积分式，同样对 $\displaystyle h(t)=\frac{1}{m\omega_n}\sin(\omega_n t)$ 进行离散采样，然后通过 `fft` 得到 $H(\omega)$。


<center>
<img src="/images/ClapVib_fft.png" width="75%" height="75%" />
图 无阻尼系统在简谐激励下的拍振
</center>