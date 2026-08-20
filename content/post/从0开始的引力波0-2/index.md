---
title: '从电磁波到引力波-0.2'
description: '数学基础篇之 格林函数法求解二阶常微分方程'
date: 2026-02-09T21:30:32+08:00
slug: '从电磁波到引力波-0-2'
categories:
  - '从电磁辐射到引力波'
tags:
  - '数学'
  - '格林函数'
image: 'DSC_7463.jpg'
toc: true
math: true
comments: true
draft: false
---
上一篇已经介绍了 Delta 函数与格林函数的基本内容，本节我们将介绍如何使用格林函数求解数学物理方程。
<!--more-->

## 二阶常微分方程
考虑任意满足以下形式的方程：

$$
\begin{align}
    \frac{\mathrm{d^{2}}}{\mathrm{d}x^{2}}g(x) + p(x)\frac{\mathrm{d}}{\mathrm{d}x}g(x) + q(x) = f(x)
\end{align}
$$

其中 $p(x),q(x)$ 依次满足定义域内一阶可导与二阶可导，由此便是一个二阶常微分方程问题，一般而言可以根据物理问题区分为边界问题与初值问题。

### 边界问题
对于边界问题，考虑满足 $f(x;\xi)=\delta(x-\xi)$ 时的方程解：格林函数 $G(x;\xi)$ ，以及对应边界 $(a,b)$ ，$G(a,\xi)=E,\;G(b,\xi)=F$ ，根据上一节，可以考虑一组解：

$$
\begin{align}
    G(x;\xi)&=\left\{\begin{matrix} G_{<}(x;\xi)=A(\xi)\; y_{1} + B(\xi)\; y_{2}\quad;\;a<x<\xi \\\\ G_{>}(x;\xi)=C(\xi)\; y_{1} + D(\xi) \;y_{2}\quad;\;\xi<x<b \end{matrix} \right.\notag\\\notag\\
    &= G_{<}-(G_{<}-G_{>})H(x-\xi)
\end{align}
$$