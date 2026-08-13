---
title: 'The Theory of Kerr TimeLike Geodesic Motion'
description: 'Use Mino Time'
date: 2026-03-17T11:22:58+08:00
slug: 'the-theory-of-kerr-timelike-geodesic-motion'
image: 'default.jpg'
toc: true
math: true
comments: true
draft: true
---
Kerr时空中测试质量的运动具有重要意义，是EMRI理论中的关键一环，因此本文将遵循 BlackHoleToolKit / KerrGeodesic 中使用Mino Time 计算其束缚轨道的子程序部分的表达式，结合该程序从度规出发详细介绍测试质量类时测地运动的解析理论。

<!--more-->
## Kerr 度规
在BL系下Kerr度规写为：

\[
\begin{align}
    		\mathrm{d}s^{2} &= -(1-\frac{2Mr}{\rho^{2}})\mathrm{d}t^{2} + \frac{\rho^{2}}{\Delta}\mathrm{d}r^{2} + \rho^{2}\mathrm{d}\theta^{2} \notag\\&+ \left[(r^{2}+a^{2})\sin^{2}\theta + \frac{2Mra^{2}\sin^{4}\theta}{\rho^{2}}\right]\mathrm{d}\varphi^{2} - \frac{4 M r a\sin^{2}\theta}{\rho^{2}}\mathrm{d}t \mathrm{d}\varphi \\&
    		\rho^{2}\equiv r^{2}+a^{2}\cos^{2}\theta,\quad \Delta \equiv r^{2}-2Mr +a^{2}
    	\end{align}
\]

其具有两个 Killing 矢量：\(k^{\mu},m^{\mu}\) ，因此对应有 \(t,\varphi\) 两个对称性，即能量 \(E\) 与角动量 \(L\) 守恒。
此外还存在一个隐藏的 Killing 矢量，对应 Carter 常数。
## 运动方程
考虑一个测试点粒子，我们的目的便是得到其类时测地线方程。哈密顿-雅可比框架是一个简单快速的方法，并且可以直接得到 Carter 常数；接下来我们便采用这一方法导出测地线。

粒子的拉格朗日量为：

\[
\mathcal{L}=\frac{1}{2}g_{\mu\nu}\dot{x}^{\mu}\dot{x}^{\nu}
\]

容易发现由于两个 Killing 矢量的存在，对应的 \(t,\varphi\) 是循环坐标，正是对应了两个守恒量。利用 Killing 矢量的性质可以直接给出,其中 \(\mu=0,1,2,3\)：

\[
\begin{align}
    E = -u^{\mu}k_{\mu} = -u_{t} = -g_{t\mu}u^{\mu} = A\;\dot{t}+ B\; \dot{\varphi}\\
    L_{z} = u^{\mu}m_{\mu} = u_{\varphi}=g_{\varphi\mu}u^{\mu}= -B\;\dot{t}+ C\; \dot{\varphi}
\end{align}
\]

## 运动频率

## 运动轨迹

## Reference