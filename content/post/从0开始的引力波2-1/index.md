---
title: '从电磁辐射到引力波-2-1'
description: '引力波篇-线性微扰论'
date: 2025-11-27T16:06:48+08:00
slug: '从电磁辐射到引力波-2-1'
categories:
  - '从电磁辐射到引力波'
tags:
  - '引力波'
  - '微扰论'
image: 'DSC_3748.jpg'
toc: true
math: true
comments: true
draft: false
---
为了描述引力波，可以采取解析求解爱因斯坦方程、微扰、数值相对论的方法，其中解析求解场方程非常困难，由于其为二阶非线性方程，通常只有当时空具有球对称、柱对称等特殊对称性时有解，但其能够描述由于非线性导致的不同于电磁波的相互作用；微扰理论则将波源作为背景时空的微小扰动而不必施加额外对称性，仅需要考虑（已知）背景时空的信息；当时空无法施加额外对称性且背景时空信息缺失时，便只能通过数值的方法直接求解完整的非线性方程，非线性使得方程的数值求解非常困难，因此专门发展出了一条分支：数值相对论。作为入门引力波的经典路径，我们优先从微扰理论出发来开启引力波之旅。

<!--more-->
### 准备
时空由度规 \(g_{\mu\nu}\) 描述，对时空进行扰动有：\(g_{\mu\nu}=\bar{g}_{\mu\nu}+h_{\mu\nu}+\mathcal{O}^{2}\)，其中 \(\bar{g}_{\mu\nu}\) 为背景时空， 略去高阶项，仅考虑一阶项 \(h_{\mu\nu}\) 即为线性微扰，我们只考虑线性微扰。注意，度规的零阶项、高阶项度规也应该具有同样的对称性，即 \(h_{(\mu\nu)}\)。

此后，假设： \(g^{\mu\nu}=\bar{g}^{\mu\nu}+A^{\mu\nu}\) ，由于： \(g_{\mu\nu}g^{\nu n}=\delta_{\mu}^{\;n}\)，可以得到： \(\bar{g}^{\nu n}\bar{g_{\mu\nu}}+ h_{\mu\nu} \bar{g}^{\nu n} + A^{\nu n}\bar{g_{\mu\nu}}+\mathcal{O}^{2} = \delta_{\mu}^{\;n}\) ，略去二阶项则有： \(A^{\nu n}\bar{g}_{\mu\nu}\bar{g}^{\mu m}=-h_{\mu\nu}\bar{g}^{\nu n}\bar{g}^{\mu m}\)，由此可得：\(A^{\mu\nu}=-h^{\mu\nu}\)，即： \(g^{\mu\nu}=\bar{g}^{\mu\nu} - h^{\mu\nu}\)。现在便可以计算场方程的线性近似了。

### Christoffe symbol
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

其中最后一个等号用到了 \((\bar{g}_{\mu\nu}\text{,}\;\bar{\Gamma}^{\rho}_{\;\mu\nu})\) 是适配的事实，因此：\(\bar{g}_{\sigma\nu;\;\mu}=\bar{g}_{\mu\sigma;\;\nu}=\bar{g}_{\mu\nu;\;\sigma}=0\) ，又意识到：

\[
\begin{align}
    \bar{g}^{\rho\sigma}\Gamma^{\tau}_{\;\mu\nu} h_{\tau\sigma}&=\bar{g}^{\rho\sigma}\Gamma^{\tau}_{\;\mu\nu} g_{\tau m}h^{mn}g_{\sigma n}=\bar{g}^{\rho\sigma}\Gamma^{\tau}_{\;\mu\nu} \bar{g}_{\tau m}h^{mn}\bar{g}_{\sigma n}+\mathcal{O}(h^{2})=\Gamma^{\tau}_{\;\mu\nu} \bar{g}_{\tau m}h^{m\rho}+\mathcal{O}(h^{2})\notag\\
    &=(\bar{\Gamma}^{\tau}_{\;\mu\nu}+\delta \Gamma^{\tau}_{\;\mu\nu}) \bar{g}_{\tau m}h^{m\rho}+\mathcal{O}(h^{2})=h^{\rho\sigma}\bar{\Gamma}^{\tau}_{\;\mu\nu}\bar{g}_{\tau\sigma}+\mathcal{O}(h^{2})
\end{align}
\]

最后一步是因为 \(\delta\Gamma^{\rho}_{\mu\nu}h^{m\rho}\) 为高阶小量（带入上面 \(\Gamma^{\rho}_{\mu\nu}\) 的微扰展开便知其最少是 \(h^{2}\) 阶项） 。因此便有：

\[
\begin{align}
    \Gamma^{\rho}_{\;\mu\nu}&=\bar{\Gamma}^{\rho}_{\;\mu\nu}+\delta\Gamma^{\rho}_{\;\mu\nu}+\mathcal{O}(h^{2})\notag\\
    &=\bar{\Gamma}^{\rho}_{\;\mu\nu} + \frac{1}{2}\bar{g}^{\rho\sigma}\left( h_{\sigma\nu;\;\mu}+h_{\mu\sigma;\;\nu}-h_{\mu\nu;\;\sigma} \right)+\mathcal{O}(h^{2})\notag\\
    &=\bar{\Gamma}^{\rho}_{\;\mu\nu} + \frac{1}{2}g^{\rho\sigma}\left( h_{\sigma\nu;\;\mu}+h_{\mu\sigma;\;\nu}-h_{\mu\nu;\;\sigma} \right)+\mathcal{O}(h^{2})
\end{align}
\]

关于这一结果，也可以直接对Christoffe符号进行一阶变分，同样可以直接得到 \(\delta\Gamma^{\rho}_{\;\mu\nu}\) 对应的表达式 。 

### Ricci tensor and Einstein equation
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

最后，由于场方程：\(R_{\mu\nu}-\frac{1}{2}g_{\mu\nu}R=8\pi T_{\mu\nu}\) ，可以得到： \(R_{\mu\nu}=8\pi\left(T_{\mu\nu}-\frac{1}{2}g_{\mu\nu}T\right)\) ， 而能动张量线性微扰展开有：

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

此外，可以发现实际上线性近似下的场方程实际上复杂度仍然很高，仍然是十个非线性二阶偏微分方程，求解起来非常困难，因此根据不同的问题还需要进行进一步针对性简化，例如在黑洞微扰理论中对微扰度规利用具有特定对称性的特殊函数进行构造，在宇宙微扰理论中采取的是球对称的FW度规作为背景，在引力波中通常采取最简单的闵氏度规为背景。

### 补充
1. 实际上，结合变分角度看，有：\(g_{\mu\nu}=g^{0}_{\mu\nu}+\delta g_{\mu\nu}+\delta^{2} g_{\mu\nu}+\dots\) ， 因此在许多文献中通常有： \(h_{\mu\nu}=\delta g_{\mu\nu}\)

2. 关于 \(h_{\mu\nu}\) 的导数是一阶小量的证明：

    首先，对于场方程而言 \(g_{ab}\) 是一个确定的解，因此 \(h_{\mu\nu}\ll 1\) 也是确定的，因此其并非动态的无限性，而仅仅是一个很小（一阶小）的数值。

    此外，考虑单参度规群 \(g_{ab}(s)\) ，有 \(g_{ab}(0)=g^{0}_{ab}\)，则有：
    
    \(A_{ab}=\left.\frac{\mathrm{d}g_{ab}}{\mathrm{d}s}\right|_{s\rightarrow0} = \lim\limits_{s\rightarrow0}\frac{g_{ab}(s)-g^{0}_{ab}}{s}\) ，因此其小量展开有：\(g_{ab}(s)=g^{0}_{ab}+sA_{ab}+\mathcal{O}(s^{2})\) ， 因此有：\(h_{ab}(s)=sA_{ab}+\mathcal{O}(s^{2})\) ， 可以看到 \(h_{ab}(s)\) 是按小参量 \(s\) 变化的，而对 \(h_{ab}\) 的求导是对时空坐标 \(x^{a}\neq s\) 进行的，因此 \(\partial_{c} h_{ab}(s)\sim s\partial_{c}A_{ab}\) ，而 \(g_{ab}\) 及其对时空坐标的 \(n\) 阶导数自然不是小量（因为微分流形已经是定义光滑的），因此对于 \(\partial_{c} h_{ab}(s)\) 是一阶小的，回到确定的场方程当中，\(s\) 取确定的一阶小数值，则 \(\partial_{c} h_{ab}\) 也是一阶小量了。
    