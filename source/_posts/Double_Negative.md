---
title: 双负等效材料常数
date: 2020-02-21 13:04:18
mathjax: true
tags:
---

在单胞尺度上进行等效平均（因此需要在 long wavelength 的条件下方可成立），得出等效质量密度 $\rho_{\text{eff}}$ 和等效模量 $K_{\text{eff}},\mu_{\text{eff}}$ 对频率 $\omega$ 的变化。

## 负等效密度

利用牛顿第二定律，由平衡方程
$$
\sum_{\partial V} \boldsymbol{F} = (\rho_{\text{eff}} V) \ddot{\boldsymbol{u}} = -\rho_{\text{eff}} V \omega^2 \boldsymbol{u}
$$

即
$$
\begin{equation}
\begin{Bmatrix}
\sum F_x\\\\
\sum F_y\\\\
\sum F_z
\end{Bmatrix} 
= -\omega^2 V
\begin{bmatrix}
	\rho_{\text{eff}}^{11} & \rho_{\text{eff}}^{12} & \rho_{\text{eff}}^{13}\\\\
	\rho_{\text{eff}}^{21} & \rho_{\text{eff}}^{22} & \rho_{\text{eff}}^{23}\\\\
	\rho_{\text{eff}}^{31} & \rho_{\text{eff}}^{32} & \rho_{\text{eff}}^{33}
\end{bmatrix} 
\begin{Bmatrix}
	u_x\\\\
	u_y\\\\
	u_z
\end{Bmatrix}
\end{equation}
$$

指定单胞边界线（面）上的位移，通过有限元方法求解其支反力，然后由上式得到等效密度各分量。比如设 $\boldsymbol{u}=(1,0,0)\exp(i\omega t)$ 便可得 $[\rho_{\text{eff}}]$ 中第1列各元素，设 $\boldsymbol{u}=(0,1,0)\exp(i\omega t)$ 便可得 $[\rho_{\text{eff}}]$ 中第2列各元素。


## 负等效刚度/模量

以给定单胞边界线（面）上的位移 $\boldsymbol{u}$ 的方式，构造出预想的应变场，然后计算支反力，由能量方程
$$
\sum_{\partial V} \boldsymbol{F} \cdot \boldsymbol{u} = \frac{1}{2}C_{ijkl}^{\text{eff}} \varepsilon_{ij} \varepsilon_{kl} = \frac{1}{2}
\begin{bmatrix}
	\varepsilon_{11} \\\\
	\varepsilon_{22} \\\\
	\varepsilon_{33} \\\\
	2\varepsilon_{23} \\\\
	2\varepsilon_{13} \\\\
	2\varepsilon_{12} \\\\
\end{bmatrix}^{\top}
\begin{bmatrix}
	\lambda+2\mu & \lambda & \lambda & 0 & 0 & 0\\\\
	\lambda & \lambda+2\mu & \lambda & 0 & 0 & 0\\\\
	\lambda & \lambda & \lambda+2\mu & 0 & 0 & 0\\\\
	0 & 0 & 0 & \mu & 0 & 0\\\\
	0 & 0 & 0 & 0 & \mu & 0\\\\
	0 & 0 & 0 & 0 & 0 & \mu\\\\
\end{bmatrix}^{\text{eff}}
\begin{bmatrix}
	\varepsilon_{11} \\\\
	\varepsilon_{22} \\\\
	\varepsilon_{33} \\\\
	2\varepsilon_{23} \\\\
	2\varepsilon_{13} \\\\
	2\varepsilon_{12} \\\\
\end{bmatrix}
$$

令 $\boldsymbol{u}=(x,y,z)\exp(i\omega t)$ 可得预加应变场为 $\varepsilon = \mathbf{I}\_{3\times 3}$，此时 $\sum \boldsymbol{F} \cdot \boldsymbol{u} = \frac{9\lambda+6\mu}{2} = \frac{9}{2}K_{\text{eff}}$，这里 $K_{\text{eff}}$ 为等效体模量。


```matlab
clear
clc
close all

% 导入 Comsol 模型
model = mphload('EffctiveDensity_silicon_pillar_a200_h100_l355.mph');

% 指定关心的频率范围
f = [4:0.02:4.5,4.5:0.002:4.55,4.55:0.02:5.5]*1e6;

% 有限元求解
omega = 2*pi*f;
model.study('std1').feature('freq').set('plist', f);
% 根据需要指定位移场
model.component('comp1').physics('solid').feature('disp1').set('U0', {'0'; '0'; '1e-4'}); 
% 或
% model.component('comp1').physics('solid').feature('disp1').set('U0', {'x'; 'y'; 'z'}); 
% 等等
model.sol('sol1').runAll;

% 积分，求解边界支反力
model.result.numerical.create('int11', 'IntSurface');
model.result.numerical('int11').selection.set([1 2 5 18]);
model.result.numerical('int11').set('expr', {'solid.RFx' 'solid.RFy' 'solid.RFz'});
temp = model.result.numerical('int11').getReal + 1i*model.result.numerical('int11').getImag;
model.result.numerical.remove('int11')

% 以下可跟据上述分析，得到各个等效材料常数
```