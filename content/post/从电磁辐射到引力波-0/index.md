---
title: '从电磁辐射到引力波-0'
description: '大纲篇'
date: 2025-11-25T00:01:02+08:00
slug: '从电磁辐射到引力波-0'
tags:
  - '引力波'
  - '电磁波'
image: 'default.jpg'
toc: true
math: true
comments: true
draft: true
---
自17世纪法拉第发现电磁感应到18世纪麦克斯韦总结出麦克斯韦方程组，到赫兹第一次证明电磁波的存在，电磁波成为塑造现代社会的第一推动，当今信息时代一切的一切都基于电磁波构建。1915年，爱因斯坦写下场方程并预言了一种新的波动形式 —— 时空的涟漪 —— 引力波；一百年后，LIGO成功探测到第一例来自双黑洞并和产生的引力辐射，就此揭开了引力波时代的序幕。

<!--more-->

本系列旨在作为个人在主题学习过程中的笔记，另一个目的是极尽详尽的展现在引力波天体物理当中被大部分文献略过的细节与涉及到的基础知识。

规划大纲如下：
# 数学准备

## 达朗贝尔解


## 推迟势

## Green 函数法

# 电动力学
## 电磁感应

## 麦克斯韦方程

## 推迟解：电磁波

## 折反射定律

## 衍射与干涉

## 电磁辐射

### 单极子、偶极子与四极子

### 八级子

## 动量与能流

## 四维协变理论


# 引力波

## 线性微扰
时空由度规 \(g_{\mu\nu}\) 描述，对时空进行扰动有：\(g_{\mu\nu}=\bar{g}_{\mu\nu}+h_{\mu\nu}+\mathcal{O}^{2}\)，其中 \(\bar{g}_{\mu\nu}\) 为背景时空， 略去高阶项，仅考虑一阶项 \(h_{\mu\nu}\) 即为线性微扰，我们只考虑线性微扰。注意，度规的零阶项、高阶项度规也应该具有同样的对称性，即 \(h_{(\mu\nu)}\)。

此后，假设： \(g^{\mu\nu}=\bar{g}^{\mu\nu}+A^{\mu\nu}\) ，由于： \(g_{\mu\nu}g^{\nu n}=\delta_{\mu}^{\;n}\)，可以得到： \(\bar{g}^{\nu n}\bar{g_{\mu\nu}}+ h_{\mu\nu} \bar{g}^{\nu n} + A^{\nu n}\bar{g_{\mu\nu}}+\mathcal{O}^{2} = \delta_{\mu}^{\;n}\) ，略去二阶项则有： \(A^{\nu n}\bar{g}_{\mu\nu}\bar{g}^{\mu m}=-h_{\mu\nu}\bar{g}^{\nu n}\bar{g}^{\mu m}\)，由此可得：\(A^{\mu\nu}=-h^{\mu\nu}\)，即： \(g^{\mu\nu}=\bar{g}^{\mu\nu} - h^{\mu\nu}\)。现在便可以计算场方程的线性近似了。

首先考虑的是 Christoffe 符号，与度规适配 \((g_{\mu\nu},\Gamma^{\rho}_{\;\mu\nu})\) 即有表达式：

\[
\Gamma^{\rho}_{\;\mu\nu}=\frac{1}{2}\;g^{\rho\sigma}(g_{\sigma\nu,\mu}+g_{\mu\sigma,\nu}-g_{\mu\nu,\sigma})
\]

所谓适配便是:\(\nabla_{\rho} g_{\mu\nu}=0\) ; 线性微扰展开当然有：\(\Gamma^{\rho}_{\;\mu\nu}=\bar{\Gamma}^{\rho}_{\;\mu\nu}+\delta\Gamma^{\rho}_{\;\mu\nu}+\mathcal{O}(\delta^{2}\Gamma^{\rho}_{\;\mu\nu})\)，下面给出各项表达式:

\[
\begin{align}
    \Gamma^{\rho}_{\;\mu\nu}&=\frac{1}{2}(\bar{g}^{\rho\sigma}-h^{\rho\sigma}+\mathcal{O}(h^{2}))\left[\bar{g}_{\sigma\nu,\;\mu}+h_{\sigma\nu,\;\mu}+\mathcal{O}(h^{2})_{,\;\mu} + \bar{g}_{\mu\sigma,\;\nu}+h_{\mu\sigma,\;\nu}+\mathcal{O}(h^{2})_{,\;\nu} - \bar{g}_{\mu\nu,\;\sigma}+h_{\mu\nu,\;\sigma}+\mathcal{O}(h^{2})_{,\;\sigma} \right]\notag\\
    &= \frac{1}{2}\bar{g}^{\rho\sigma}\left( \bar{g}_{\sigma\nu,\;\mu}+\bar{g}_{\mu\sigma,\;\nu}-\bar{g}_{\mu\nu,\;\sigma}\right) + \frac{1}{2}\bar{g}^{\rho\sigma}\left( h_{\sigma\nu,\;\mu}+h_{\mu\sigma,\;\nu}-h_{\mu\nu,\;\sigma} \right) - \frac{1}{2}h^{\rho\sigma}\left( \bar{g}_{\sigma\nu,\;\mu}+\bar{g}_{\mu\sigma,\;\nu}-\bar{g}_{\mu\nu,\;\sigma} \right) + \mathcal{O}(h^{2})\notag\\
    &= \bar{\Gamma}^{\rho}_{\;\mu\nu} + \frac{1}{2}\bar{g}^{\rho\sigma}\left( h_{\sigma\nu;\;\mu}+h_{\mu\sigma;\;\nu}-h_{\mu\nu;\;\sigma} + \Gamma^{\tau}_{\;\sigma\mu}h_{\tau\nu} + \Gamma^{\tau}_{\;\nu\mu}h_{\sigma\tau} + \Gamma^{\tau}_{\;\sigma\nu}h_{\tau\mu} + \Gamma^{\tau}_{\;\mu\nu}h_{\sigma\tau} - \Gamma^{\tau}_{\;\mu\sigma}h_{\tau\nu}-\Gamma^{\tau}_{\;\mu\sigma}h_{\tau\nu}\right) \notag\\
    &- \frac{1}{2}h^{\sigma\rho}\left( \bar{g}_{\sigma\nu;\;\mu}+\bar{g}_{\mu\sigma;\;\nu}-\bar{g}_{\mu\nu;\;\sigma} + \bar{\Gamma}^{\tau}_{\;\sigma\mu}\bar{g}_{\tau\nu} + \bar{\Gamma}^{\tau}_{\;\nu\mu}\bar{g}_{\sigma\tau} + \bar{\Gamma}^{\tau}_{\;\sigma\nu}\bar{g}_{\tau\mu} + \bar{\Gamma}^{\tau}_{\;\mu\nu}\bar{g}_{\sigma\tau} - \bar{\Gamma}^{\tau}_{\;\mu\sigma}\bar{g}_{\tau\nu}-\bar{\Gamma}^{\tau}_{\;\mu\sigma}\bar{g}_{\tau\nu} \right) + \mathcal{O}(h^{2}) \notag\\
    &= \bar{\Gamma}^{\rho}_{\;\mu\nu} + \frac{1}{2}\bar{g}^{\rho\sigma}\left( h_{\sigma\nu;\;\mu}+h_{\mu\sigma;\;\nu}-h_{\mu\nu;\;\sigma} \right)+\bar{g}^{\rho\sigma}\Gamma^{\tau}_{\;\mu\nu}h_{\tau\sigma}-h^{\rho\sigma}\bar{\Gamma}^{\tau}_{\;\mu\nu}\bar{g}_{\tau\sigma}+\mathcal{O}(h^{2})
\end{align}
\]

其中最后一个等号用到了 \((\bar{g}_{\mu\nu},\bar{\Gamma}^{\rho}_{\;\mu\nu})\) 是适配的事实，因此：\(\bar{g}_{\sigma\nu;\;\mu}=\bar{g}_{\mu\sigma;\;\nu}=\bar{g}_{\mu\nu;\;\sigma}=0\) ，又意识到：

\[
\begin{align}
    \bar{g}^{\rho\sigma}\Gamma^{\tau}_{\;\mu\nu} h_{\tau\sigma}&=\bar{g}^{\rho\sigma}\Gamma^{\tau}_{\;\mu\nu} g_{\tau m}h^{mn}g_{\sigma n}=\bar{g}^{\rho\sigma}\Gamma^{\tau}_{\;\mu\nu} \bar{g}_{\tau m}h^{mn}\bar{g}_{\sigma n}+\mathcal{O}(h^{2})=\Gamma^{\tau}_{\;\mu\nu} \bar{g}_{\tau m}h^{m\rho}+\mathcal{O}(h^{2})\notag\\
    &=(\bar{\Gamma}^{\tau}_{\;\mu\nu}+\delta \Gamma^{\tau}_{\;\mu\nu}) \bar{g}_{\tau m}h^{m\rho}+\mathcal{O}(h^{2})=h^{\rho\sigma}\bar{\Gamma}^{\tau}_{\;\mu\nu}\bar{g}_{\tau\sigma}+\mathcal{O}(h^{2})
\end{align}
\]

最后一步是因为 \(\delta\Gamma^{\rho}_{\mu\nu}h^{m \rho} \) 为高阶小量（带入上面 \(\Gamma^{\rho}_{\mu\nu}\) 的微扰展开便知其最少是 \(h^{2}\) 阶项） 。因此便有：

\[
\begin{align}
    \Gamma^{\rho}_{\;\mu\nu}&=\bar{\Gamma}^{\rho}_{\;\mu\nu}+\delta\Gamma^{\rho}_{\;\mu\nu}+\mathcal{O}(h^{2})\notag\\
    &=\bar{\Gamma}^{\rho}_{\;\mu\nu} + \frac{1}{2}\bar{g}^{\rho\sigma}\left( h_{\sigma\nu;\;\mu}+h_{\mu\sigma;\;\nu}-h_{\mu\nu;\;\sigma} \right)+\mathcal{O}(h^{2})\notag\\
    &=\bar{\Gamma}^{\rho}_{\;\mu\nu} + \frac{1}{2}g^{\rho\sigma}\left( h_{\sigma\nu;\;\mu}+h_{\mu\sigma;\;\nu}-h_{\mu\nu;\;\sigma} \right)+\mathcal{O}(h^{2})
\end{align}
\]

关于这一结果，也可以直接对Christoffe符号进行一阶变分，同样可以直接得到 \(\delta\Gamma^{\rho}_{\;\mu\nu}\) 对应的表达式 。 

现在我们可以计算里奇张量的一阶扰动，同样的：

\[
\begin{align}
    R_{\mu\nu}&=\Gamma^{\rho}_{\;\mu\nu,\;\rho}-\Gamma^{\rho}_{\;\mu\rho,\;\nu}+ \Gamma^{\rho}_{\;\lambda\rho}\Gamma^{\lambda}_{\;\mu\nu}-\Gamma^{\rho}_{\;\lambda\nu}\Gamma^{\lambda}_{\;\mu\rho}\notag\\
    &= \bar{R}_{\mu\nu}+\delta R_{\mu\nu} + \mathcal{O}(h^{2})\notag\\
    &=\bar{R}_{\mu\nu} + (\delta\Gamma^{\rho}_{\;\mu\nu})_{,\;\rho}-(\delta\Gamma^{\rho}_{\;\mu\rho})_{,\;\nu}+ \bar{\Gamma}^{\rho}_{\;\lambda\rho}\delta\Gamma^{\lambda}_{\;\mu\nu} + \delta\Gamma^{\rho}_{\;\lambda\rho}\bar{\Gamma}^{\lambda}_{\;\mu\nu}-\bar{\Gamma}^{\rho}_{\;\lambda\nu}\delta\Gamma^{\lambda}_{\;\mu\rho} - \delta\Gamma^{\rho}_{\;\lambda\nu}\bar{\Gamma}^{\lambda}_{\;\mu\rho} + \mathcal{O}(h^{2}) \notag\\
    &=\bar{R}_{\mu\nu} + \left[(\delta\Gamma^{\rho}_{\;\mu\nu})_{,\;\rho}+\Gamma^{\rho}_{\;\lambda\rho}\delta\Gamma^{\lambda}_{\;\mu\nu} - \Gamma^{\lambda}_{\;\mu\rho}\delta\Gamma^{\rho}_{\;\lambda\nu} - \Gamma^{\lambda}_{\;\nu\rho}\delta\Gamma^{\rho}_{\;\lambda\mu}\right] - \left[(\delta\Gamma^{\rho}_{\;\mu\rho})_{,\;\nu}+\Gamma^{\rho}_{\;\lambda\nu}\delta\Gamma^{\lambda}_{\;\mu\rho} - \Gamma^{\lambda}_{\;\mu\nu}\delta\Gamma^{\rho}_{\;\lambda\rho} - \Gamma^{\lambda}_{\;\nu\rho}\delta\Gamma^{\rho}_{\;\lambda\mu}\right] + \mathcal{O}(h^{2}) \notag\\
    &=\bar{R}_{\mu\nu} + (\delta\Gamma^{\rho}_{\;\mu\nu})_{;\;\rho}-(\delta\Gamma^{\rho}_{\;\mu\rho})_{;\;\nu} + \mathcal{O}(h^{2})
\end{align}
\]

最后，由于场方程：\(R_{\mu\nu}-\frac{1}{2}g_{\mu\nu}R=8\pi T_{\mu\nu}\) ，可以得到：\(R_{\mu\nu}=8\pi\left( T_{\mu\nu}-\frac{1}{2}g_{\mu\nu}T \right) \) ， 而能动张量线性微扰展开有：

\[
\begin{align}
    T_{\mu\nu}&=\bar{T}_{\mu\nu}+\delta T_{\mu\nu}+\mathcal{O}(h^{2})\notag\\
    T &= g^{\mu\nu}T_{\mu\nu}=\bar{g}^{\mu\nu}\bar{T}_{\mu\nu} + \bar{g}^{\mu\nu} \delta T_{\mu\nu} - h^{\mu\nu}\bar{T}_{\mu\nu} + \mathcal{O}(h^{2})\notag\\
    &= \bar{T} + \delta T +\mathcal{O}(h^{2})
\end{align}
\]

于是场方程的线性展开为：

\[
\begin{align}
    \bar{R}_{\mu\nu} + \delta R_{\mu\nu}+\mathcal{O}(h^{2})&=8\pi \left(\bar{T}_{\mu\nu}-\frac{1}{2}\bar{g}_{\mu\nu}\bar{T}\right)+ 8\pi \left[ \delta T_{\mu\nu}-\frac{1}{2}\left(\bar{g}_{\mu\nu}\delta T + h_{\mu\nu}\bar{T}  \right) \right] +\mathcal{O}(h^{2}) \notag\\
    &=\bar{R}_{\mu\nu} + 8\pi \left[ \delta T_{\mu\nu}-\frac{1}{2}\left(\bar{g}_{\mu\nu}\bar{g}^{\sigma\rho} \delta T_{\sigma\rho} - \bar{g}_{\mu\nu}h^{\sigma\rho}\bar{T}_{\sigma\rho} + h_{\mu\nu}\bar{T}  \right) \right]+\mathcal{O}(h^{2})
\end{align}
\]

即线性扰动方程为：

\[
\begin{align*}
    \delta R_{\mu\nu}&=(\delta\Gamma^{\rho}_{\;\mu\nu})_{;\;\rho}-(\delta\Gamma^{\rho}_{\;\mu\rho})_{;\;\nu}\notag\\
    &= 8\pi \left[ \delta T_{\mu\nu}-\frac{1}{2}\left(\bar{g}_{\mu\nu}\bar{g}^{\sigma\rho} \delta T_{\sigma\rho} - \bar{g}_{\mu\nu}h^{\sigma\rho}\bar{T}_{\sigma\rho} + h_{\mu\nu}\bar{T}  \right) \right]+\mathcal{O}(h^{2})
\end{align*}
\]

以上便构成了广义相对论当中线性微扰理论的基础，其给出的结果与一阶变分理论一致；这不仅是引力波理论的出发点，也是黑洞微扰理论、宇宙微扰理论的基础。

## 平面波解
对于引力波，由于地球的引力效应很弱，可以近似为平直时空，因此我们考虑闵可夫斯基时空为背景时空来计算在地球所观测到的引力波（远离波源）。

## 规范变换

## 波源：多极矩

## 测地偏移

## 能动张量与能流

## 双星系统

## 谐振子

## 转动系统

## Ringdown：黑洞微扰

## EMRIS

## 宇宙学微扰：标量诱导引力波

## 随机引力波背景





