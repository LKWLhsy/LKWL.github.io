---
title: '从电磁波到引力波-0.1'
description: '数学基础篇之 Delta 函数到格林函数法'
date: 2025-11-28T19:37:04+08:00
slug: '从电磁波到引力波-0-1'
categories:
  - '从电磁辐射到引力波'
tags:
  - '数学'
  - '格林函数'
image: 'DSC_4205.jpg'
toc: true
math: true
comments: true
draft: false
---
在求解波动问题时，往往需要面对非齐次的有源情况以及各种情况的边界条件，而对于源项分布复杂的情况寻常方法难以求得解析表达，因此需要引入特殊函数的方法来简化求解过程，其中 Delta 函数扮演者重要角色，物理中的点源可以由其表达，对于更加复杂的场源则可以通过点源模型叠加得到，因此利用 Delta 函数可以有效求解有源场的解析表达，这也是本篇文章的动机。
<!--more-->

## Delta 函数
\(\delta\) 函数最早由狄拉克引入以描述物理中的点量，属于特殊函数，在物理学中有广泛运用。以一维点电荷的电荷密度为例，为：

\[
\begin{align}
    \delta(x)=\left\{
        \begin{matrix} 0\quad\text{,}\;x\neq 0 \\\infty\quad\text{,}\;x=0 
        \end{matrix} \right.
\end{align}
\]

这便是经典的 \(\delta\) 函数定义，对全空间积分便得到单位电荷：

\[
\begin{align}
    \int^{\;+\infty}_{\;-\infty}\delta(x) \;\mathrm{d}x =1=\int^{a}_{b}\delta(x)\mathrm{d}x
\end{align}
\]

这便是 \(\delta\) 函数的积分形式，其中 \(a\, b<0\) ，实际上，描述 \(\delta\) 函数要在积分表达下去理解。作为一个从物理问题中提出的特殊函数，对各类点量均能有如此描述。最后还有必要知道 \(\delta\) 函数的导数定义，同样必须从积分视角去理解：

\[
\begin{align}
    \int^{\;+\infty}_{\;-\infty}\;f(x)\;\delta^{'}(x)\;\mathrm{d}x = f(x)\;\left.\delta(x)\right|_{\pm \infty}\;-\int^{\;+\infty}_{\;-\infty}f^{'}(x)\;\delta(x)\mathrm{d}x = -f^{'}(0)
\end{align}
\]

<font color=red>需要注意：</font> 当在“尖点”两侧的区域，\(\delta(x)=0\) 的 n 阶导数应该恒为 0 ！由此可以导出：

\[
\begin{align}
    \int^{\infty}_{-\infty}f(x)\frac{\mathrm{d}^{2}}{\mathrm{d}x^{2}}\delta(x-x_{0})\mathrm{d}x &= \left.f(x)\frac{\mathrm{d}}{\mathrm{d}x}\delta(x-x_{0})\right|^{\infty}_{-\infty}-\int^{\infty}_{-\infty} \delta(x-x_{0})\frac{\mathrm{d}}{\mathrm{d}x}f(x)\mathrm{d}x \notag\\
    &=\left.\frac{\mathrm{d}}{\mathrm{d}x}f(x)\right|_{x=x_{0}}
\end{align}
\]

推广便有：

\[
\begin{align}
    \int^{\infty}_{-\infty}f(x)\frac{\mathrm{d}^{n}}{\mathrm{d}x^{n}}\delta(x-x_{0})\mathrm{d}x =\left.\frac{\mathrm{d}^{n}}{\mathrm{d}x^{n}}f(x)\right|_{x=x_{0}}
\end{align}
\]

此外 \(\delta(x)\) 函数具有如下几点基础性质：

1. 挑选性：

\[
\begin{align}
    \int^{\;+\infty}_{-\infty}f(x)\delta(x-x_{0})\;\mathrm{d}x=f(x_{0})
\end{align}
\]

2. 如果 \(\varphi(x)=0\) 的根都为实单根，则有 

    \[
    \delta[\varphi(x)]=\sum\limits_{n}\frac{\delta(x-x_{n})}{\left|\varphi^{'}(x)\right|}
    \]

    下面给出证明：

    实单根导致 

    \[
    \begin{align}
        \delta[\varphi(x)]=\sum\limits_{n}c_{n}\;\delta(x-x_{n})=\left\{
            \begin{matrix} 0\quad\text{,}\;\varphi(x)\neq 0 \\\infty\quad\text{,}\;\varphi(x)=0 
            \end{matrix} \right.
    \end{align}
    \]

    第一个等号两边同时积分：

    \[
    \begin{align}
        \int^{\;x_{n}+\varepsilon}_{x_{n}-\varepsilon}\delta[\varphi(x)]\;\mathrm{d}x &= \int^{\;\varphi(x_{n}+\varepsilon)}_{\varphi(x_{n}-\varepsilon)}\frac{\delta[\varphi]}{\varphi^{'}}\;\mathrm{d}\varphi=\frac{1}{|\varphi^{'}(x_{n})|}\notag\\&=\int^{\;x_{n}+\varepsilon}_{x_{n}-\varepsilon}\sum\limits_{n}c_{n}\;\delta(x-x_{n})\mathrm{d}x=c_{n}\int^{\;x_{n}+\varepsilon}_{x_{n}-\varepsilon}\sum\limits_{n}\;\delta(x-x_{n})\mathrm{d}x\notag\\
        &=c_{n}
    \end{align}
    \]

    得证。
    
    一个典型的例子便是：

    \[
    \begin{align}
        \delta(a\,x)=\frac{\delta(x)}{|a|}
    \end{align}
    \]

3. 对称性：从定义式即可看出，\(\delta(x)\) 是偶函数，而从其导数定义可以看出：

    \[
    \begin{align}
        \int^{\;+\infty}_{\;-\infty}\;f(x)\;\delta^{'}(-x)\;\mathrm{d}x=-\int^{\;+\infty}_{\;-\infty}\;f(-t)\;\delta^{'}(t)\;\mathrm{d}t=f(-0)=f(0)
    \end{align}
    \]

    因此 \(\delta^{'}(x)\) 是奇函数。

4. 阶跃函数（亥维塞单位函数）：

    \[
    \begin{align}
    H(x):=\int^{x}_{-\infty}\delta(t)\mathrm{d}t= \left\{ \begin{matrix}  0\quad x<0 \\ 1 \quad x>0\end{matrix}\right.
    \end{align}
    \]

    因此也有：\(\delta(x)=\frac{\mathrm{d}H(x)}{\mathrm{d}x}\) 。
5. 物理上的瞬时过程：

    考虑冲量定理中有：\(\mathrm{d}I=f(\tau)\mathrm{d}\tau\) ，持续的作用力可以分解为无穷个瞬时力作用接续作用，则有：

    \[
    \begin{align}
        f(t)=\sum\limits_{\tau}f(\tau)\delta(t-\tau)\mathrm{d}\tau = \int^{a}_{b}f(\tau)\delta(t-\tau)\mathrm{d}\tau
    \end{align}
    \]

    这实际上是给出了作用力随时间的变化关系。
6. 傅里叶变换

    回顾复函数的傅里叶变换：

    \[
    \begin{align}
        \left\{\begin{matrix} F(\omega)=\mathfrak{F}f(t)=\frac{1}{2\pi}\int^{\;+\infty}_{\;-\infty}f(t)e^{-\mathrm{i}\omega \,t}\mathrm{d}t =\frac{1}{\sqrt{2}\pi}\int^{\;+\infty}_{\;-\infty}f(t)e^{-\mathrm{i}\omega \,t}\mathrm{d}t\\  \\ f(t)=\mathfrak{F}^{-1}F(\omega)=\int^{\;+\infty}_{\;-\infty}F(\omega)e^{\;\mathrm{i}\omega\;t}\mathrm{d}\omega =\frac{1}{\sqrt{2}}\int^{\;+\infty}_{\;-\infty}F(\omega)e^{\;\mathrm{i}\omega\;t}\mathrm{d}\omega\end{matrix} \right.
    \end{align}
    \]

    由于挑选性：\(\frac{1}{2\pi}\int^{\;+\infty}_{\;-\infty}\delta(t)e^{-\mathrm{i}\omega\;t}\mathrm{d}t=\frac{1}{2\pi}\)，由逆变换给出：

    \[
    \begin{align}
        \delta(t) = \frac{1}{2\pi}\int^{\;+\infty}_{\;-\infty}e^{\mathrm{i}\omega\;t}\mathrm{d}\omega
    \end{align}
    \]

    这一点在物理理论的积分中有着不少作用。

最后，三维 \(\delta(x)\) 函数为：

\[
\begin{align}
    \delta(\vec{r})=\delta(x)\,\delta(y)\,\delta(z)
\end{align}
\]

补充：

1. 对于上面并没考虑的当 \(x=0\) 时的情况，可以做如下考虑：

\[
\lim\limits_{\varepsilon\rightarrow 0}\int^{\;+\varepsilon}_{\;-\varepsilon}\delta(x)\mathrm{d}x=\lim\limits_{\varepsilon\rightarrow 0}1=1
\]

2. 实际上在数学上 \(\delta\) 函数应该被视为数列函数取极限：

    \[
    \begin{align}
        \delta_{l}(x)=\left\{\begin{matrix}0\;,\quad x>\frac{1} {2\,l},x<-\frac{1}{2\,l} \\\\ \frac{1}{l}\;,\quad\quad   \frac{1}{-2\,l}<x<\frac{1}{2\,l}\end{matrix}\right.
    \end{align}
    \]

    可以发现当取数列极限：\(\lim\limits_{l\rightarrow \infty}\delta_{l}(x)=\delta(x)\) 。由此便是数学上利用数列极限对 \(\delta(x)\) 函数的描述，代回前面的讨论可以得到一样的结果。
## Green 函数
了解了 \(\delta\) 函数后便可以介绍Green函数，所谓格林函数，便是常微分方程(这里讨论的是实数域)：

\[
\begin{align}
    &\frac{\mathrm{d}}{\mathrm{d}x}\left(p(x)\frac{\mathrm{d}\;G(x;\xi)}{\mathrm{\mathrm{d}}x}\right)+q(x)\;G(x;\xi)=\delta(x-\xi)\quad; \; a<x,\xi<b \notag\\
    \notag\\
    &G(a;\xi)=A\;;G(b;\xi)=B 
\end{align}
\]

在给定边界条件时的解 \(G(x,x_{0})\;\) （对于初值问题则变量是 \(G(t,\tau)\) ，对应的是 \(p(t),q(t)\) ，定义域为：\((t,\tau>t_{0})\) ；对应初值条件为：\(G(t_{0};\tau)=A\,;\,\left.\frac{\mathrm{d}G(t;\tau)}{\mathrm{d}t}\right|_{t=t_{0}}=B\) ），<font color=red>我们只选取简单的齐次边界（初值）条件进行考虑</font>（且对与初值一般考虑 \(\;t_{0}=0\) ）。下面先研究由 \(\delta(x-\xi)\) 引起的连续性问题：

仅需要考虑非齐次方程本身而不必考虑边界或初值条件，对于线性方程总能有线性无关的解 \(y_{1}(x),y_{2}(x)\) 构成解：

\[
\begin{align}
    G(x;\xi)&=\left\{\begin{matrix} G_{<}(x;\xi)=A(\xi)\; y_{1} + B(\xi)\; y_{2}\quad;\;a<x<\xi \\\\ G_{>}(x;\xi)=C(\xi)\; y_{1} + D(\xi) \;y_{2}\quad;\;\xi<x<b \end{matrix} \right.\notag\\\notag\\
    &= G_{<}-(G_{<}-G_{>})H(x-\xi)
\end{align}
\]

\(H(x-\xi)\) 为阶跃函数，其中每一个函数（ \(G_{<},G_{>},G,y_{1},y_{2}\) ）及其线性叠加均是我们考虑的常微分方程的齐次情况的解。将 \(G=G_{<}-(G_{<}-G_{>})H(x-\xi)\) 代回我们的非齐次方程（注意到由于阶跃函数，在非齐次情况 \(x=\xi\) 也有些模糊不清了，因此我们需要积分去研究）且注意到 \(G_{<},G_{>}\) 为齐次情况的解而化简后积分有：

\[
\begin{align}
    &\left[\frac{\mathrm{d}}{\mathrm{d}x}\left(p(x)\frac{\mathrm{d}}{\mathrm{d}x}G_{<}\right)+q(x)G_{<}\right]-\left[\frac{\mathrm{d}}{\mathrm{d}x}\left(p(x)\frac{\mathrm{d}}{\mathrm{d}x}(G_{<}-G_{>})\right)+q(x)(G_{<}-G_{>})\right]H(x-\xi)\notag\\
    &-\left[\frac{\mathrm{d}}{\mathrm{d}x}\left(p(x)\frac{\mathrm{d}}{\mathrm{d}x}H(x-\xi)\right)\right]\left(G_{<}-G_{>}\right)=\delta(x-\xi)+2p(x)\frac{\mathrm{d}H(x-\xi)}{\mathrm{d}x}\frac{\mathrm{d}(G_{<}-G_{>})}{\mathrm{d}x}\notag\\
    &\Rightarrow \notag\\
    &-p(x)\delta^{'}(x-\xi)\left(G_{<}-G_{>}\right)=\delta(x-\xi)\left(1+2p(x)\frac{\mathrm{d}(G_{<}-G_{>})}{\mathrm{d}x} + p^{'}(G_{<}-G_{>})\right)\notag\\
    &\Rightarrow\notag\\
    &\int -p(x)\delta^{'}(x-\xi)\left(G_{<}-G_{>}\right)f(x)\mathrm{d}x=\int \delta(x-\xi)\left(1+2p(x)\frac{\mathrm{d}(G_{<}-G_{>})}{\mathrm{d}x} + p^{'}(G_{<}-G_{>})\right)f(x)\mathrm{d}x\notag\\
    &\Rightarrow\notag\\
    &\left.-p(G_{<}-G_{>})\delta(x-\xi)f(x)\right|^{+\infty}_{-\infty}+\int \left[p\,(G_{<}-G_{>})f(x)\right]^{'}\delta(x-\xi)\mathrm{d}x\notag\\
    &=\left[1+\left[2p(x)\frac{\mathrm{d}(G_{<}-G_{>})}{\mathrm{d}x}\right]_{x=\xi}+\left[p^{'}(G_{<}-G_{>})\right]_{x=\xi}\right]f(x)\notag\\
    &\Rightarrow\notag\\
    &\left[1+\left[p(x)\frac{\mathrm{d}(G_{<}-G_{>})}{\mathrm{d}x}\right]_{x=\xi}\right]f(\xi) = \left[p(G_{<}-G_{>})\right]_{x=\xi} f^{'}(\xi)
\end{align}
\]

由于 \(f(x)\) 为任意函数且与其一阶导数相互独立（显然线性无关），因此要等式成立只能有：

\[
\begin{align}
    \left\{\begin{matrix}\left[\frac{\mathrm{d}G_{<}}{\mathrm{d}x}-\frac{\mathrm{d}G_{>}}{\mathrm{d}x}\right]_{x=\xi}=-\frac{1}{p(x)}\quad\Rightarrow\quad \left.\frac{\mathrm{d}G}{\mathrm{d}x}\right|^{x=\xi_{+}}_{x=\xi_{-}}=\frac{1}{p(x)} \\\\ \left[G_{<}-G_{>}\right]_{x=\xi}=0 \quad\Rightarrow\quad \left.G\right|^{x=\xi_{+}}_{x=\xi_{-}}=0  \end{matrix}\right.
\end{align}
\]

这便是非齐次项为 \(\delta\) 函数的常微分方程解的连续性讨论，这对解提出了非边界（初值）的要求。在确定格林函数时非常有用。

接下来还需要证明格林函数在齐次边界（初值）问题中的对称性以便我们能够用格林函数法求解其他非齐次方程定解问题。所谓对称性，对于边界条件问题：

\[
\begin{align}
    G(x,\xi)=G(\xi,x)
\end{align}
\]

对于边界情况，可以直接求解方程：

\[
\begin{align}
    &\frac{\mathrm{d}}{\mathrm{d}x}\left(p(x)\frac{\mathrm{d}\;G(x;\xi)}{\mathrm{\mathrm{d}}x}\right)+q(x)\;G(x;\xi)=\delta(x-\xi)\quad; \; a<x,\xi<b \notag\\
    \notag\\
    &G(a;\xi)=0\;;G(b;\xi)=0 
\end{align}
\]

从解可以直接看出对称性，这仅仅涉及四个方程组的求解问题，由于直接套公式得解的情况不多，在此不多赘述。为了数学上证明对称性，选 \(G(x,x_{1})\) ，对应有方程：

\[
\begin{align}
    &\frac{\mathrm{d}}{\mathrm{d}x}\left(p(x)\frac{\mathrm{d}\;G(x;x_{1})}{\mathrm{\mathrm{d}}x}\right)+q(x)\;G(x;x_{1})=\delta(x-x_{1})\quad; \; a<x,x_{1}<b \notag\\
    \notag\\
    &G(a;x_{1})=0\;;G(b;x_{1})=0 
\end{align}
\]

乘上 \(G(x,\xi)\) 后与 \(G(x,\xi)\) 的方程乘上 \(G(x,x_{1})\) 相减，得到：

\[
\begin{align}
    \frac{\mathrm{d}}{\mathrm{d}x}\left(p(x)\frac{\mathrm{d}\;G(x;x_{1})}{\mathrm{\mathrm{d}}x}\right)G(x;\xi)-\frac{\mathrm{d}}{\mathrm{d}x}\left(p(x)\frac{\mathrm{d}\;G(x;\xi)}{\mathrm{\mathrm{d}}x}\right)G(x;x_{1})=\delta(x-x_{1})G(x;\xi)-\delta(x-\xi)G(x;x_{1})
\end{align}
\]

积分得：

\[
\begin{align}
    &\int \left[\frac{\mathrm{d}}{\mathrm{d}x}\left(p(x)\frac{\mathrm{d}\;G(x;x_{1})}{\mathrm{\mathrm{d}}x}\right)G(x;\xi)-\frac{\mathrm{d}}{\mathrm{d}x}\left(p(x)\frac{\mathrm{d}\;G(x;\xi)}{\mathrm{\mathrm{d}}x}\right)G(x;x_{1})\right]\mathrm{d}x\notag\\
    &=\int \mathrm{d}\left[p(x)\left(\frac{\mathrm{d}\;G(x;x_{1})}{\mathrm{\mathrm{d}}x}G(x;\xi)-\frac{\mathrm{d}\;G(x;\xi)}{\mathrm{\mathrm{d}}x}G(x;x_{1})\right) \right]\notag\\
    &=\left[p(x)\left(\frac{\mathrm{d}\;G(x;x_{1})}{\mathrm{\mathrm{d}}x}G(x;\xi)-\frac{\mathrm{d}\;G(x;\xi)}{\mathrm{\mathrm{d}}x}G(x;x_{1})\right) \right]^{x=a}_{x=b}=G(x_{1};\xi)-G(\xi;x_{1})
\end{align}
\]

可以看到，在齐次边界条件下有：\(G(x_{1};\xi)=G(\xi,x_{1})\) ，即证明了对称性。最后强调，仅在齐次边界条件成立！

按照这一流程，可以将其中一组定解问题换成需要求解的任意非齐次方程且配以非齐次边界条件，同样可以给出类似上式积分的表达式作为解。

对于初值问题则还需要假设 \(p(t)=p(-t),q(t)=q(-t)\) ，这是为了保证常微分方程 算子 的时间反演不变，对应有对称性：

\[
\begin{align}
    G(t,\tau)=G(-\tau,-t)
\end{align}
\]

考虑方程：

\[
\begin{align}
    &\frac{\mathrm{d}}{\mathrm{d}t}\left(p(t)\frac{\mathrm{d}\;G(t;\tau)}{\mathrm{\mathrm{d}}t}\right)+q(t)\;G(t;\tau)=\delta(t-\tau)\quad; \; -\infty<t,\tau<\infty \notag\\
    \notag\\
    &\left.G(t;\tau)\right|_{t>\tau}=0\;;\left.\frac{\mathrm{d}G(t;\tau)}{\mathrm{d}t}\right|_{t>\tau}=0 
\end{align}
\]

以及方程：

\[
\begin{align}
    &\frac{\mathrm{d}}{\mathrm{d}(-t)}\left(p(-t)\frac{\mathrm{d}\;G(-t;-t_{1})}{\mathrm{\mathrm{d}}(-t)}\right)+q(-t)\;G(-t;-t_{1})=\delta(t_{1}-t)\quad; \; -\infty<t,t_{1}<\infty \notag\\
    \notag\\
    &\left.G(-t;-t_{1})\right|_{-t<-t_{1}}=0\;;\left.\frac{\mathrm{d}G(-t;-t_{1})}{\mathrm{d}-t}\right|_{-t<-t_{1}}=0 
\end{align}
\]

注意到，由于需要考虑时间反演，因此需要将定义域扩充到 \((-\infty,\infty)\) ，当然对于 \(G(-t;-t_{1})\) 的方程而言，初值关系也要是反过来的（向负半轴方向传播）。此后与边界条件情况相同处理便可证明齐次初值下的对称性。

综上，我们以及介绍了格林函数法所需要的基本条件，其中连续性问题为格林函数的求解提供帮助，而对称性则为得到具有正确自变量的任意非齐次常微分方程的解具有重要作用。

由于在下一节格林函数法中就要运用，因此例题暂时便不多赘述，基本方法就是把要求解的方程其非齐次项 \(f(x)\) 写成 \(f(\xi)\) ，然后利用证明对称性的流程即可得到解。

## Green 函数法
在介绍格林函数法之前，还需要引入格林定理：

考虑函数 \(u,v\) ，它们在空间 \(T\) 内二阶连续可导，在曲面 \(\Sigma\) 上一阶连续可导，则考虑有：

\[
\begin{align}
    \iint\limits_{\Sigma}u\vec{\nabla} v\cdot\mathrm{d}\vec{S}&=\iiint\limits_{T}\vec{\nabla}\cdot (u\vec{\nabla}v)\mathrm{d}V\notag\\
    &=\iiint\limits_{T}(\vec{\nabla}u)\cdot(\vec{\nabla}v)\mathrm{d}V+\iiint\limits_{T}u\Delta v\mathrm{d}V\notag\\
    \iint\limits_{\Sigma}u\frac{\partial \,v}{\partial\,n}\mathrm{d}S &=\iiint\limits_{T}(\vec{\nabla}u)\cdot(\vec{\nabla}v)\mathrm{d}V+\iiint\limits_{T}u\Delta v\mathrm{d}V
\end{align}
\]

此为格林第一公式。

将 \(u,v\) 调换，相减得：

\[
\begin{align}
    \iint\limits_{\Sigma}(u\vec{\nabla}v-v\vec{\nabla}u)\cdot\mathrm{d}\vec{S}&=\iiint\limits_{T}(u\Delta v-v\Delta u)\mathrm{d}V\notag\\
    \Rightarrow \iint\limits_{\Sigma}(u\frac{\partial \,v}{\partial\,n}-v\frac{\partial \,u}{\partial\,n})\mathrm{d}S &= \iiint\limits_{T}(u\Delta v-v\Delta u)\mathrm{d}V
\end{align}
\]

此为格林第二公式。

### 泊松方程的格林函数法
对于格林函数法，首先考虑的便是泊松方程：\(\Delta u(\vec{x}) = f(\vec{x}),\vec{x}\in T^{3}\) ，对应的格林函数可以考虑方程：\(\Delta g(\vec{x};\vec{x}_{0}) = \delta(\vec{x}-\vec{x}_{0})\) ，这对应一个位于 \(\vec{x}_{0}\) 的点电荷产生的电场，因此有：

\[
\begin{align}
    \iiint\limits_{T^{3}}(g\Delta u-u\Delta g)\mathrm{d}V&=\iiint\limits_{T^{3}} g(\vec{x};\vec{x}_{0})f(\vec{x})\mathrm{d}V-\iiint\limits_{T^{3}}u(\vec{x})\delta(\vec{x}-\vec{x}_{0})\mathrm{d}V\notag\\
    &=-\iint\limits_{\Sigma^{2}}(u\frac{\partial \,g}{\partial\,n}-g\frac{\partial \,u}{\partial\,n})\mathrm{d}S
\end{align}
\]

考虑到格林公式需要定义域一、二阶可导，而点源显然不可导，因此我们需要考虑一个包围点源的无限小曲面 \(\sigma^{2}\) ，以此挖去涵盖点源的体积 \(t^{3}\) ，在剩下的区域运用格林公式，因此有：

\[
\begin{align}
    \iiint\limits_{T^{3}-t^{3}}g(\vec{x};\vec{x}_{0})f(\vec{x})\mathrm{d}V &= -\iint\limits_{\Sigma^{2}}(u\frac{\partial \,g}{\partial\,n}-g\frac{\partial \,u}{\partial\,n})\mathrm{d}S - \iint\limits_{\sigma^{2}}(u\frac{\partial \,g}{\partial\,n}-g\frac{\partial \,u}{\partial\,n})\mathrm{d}S
\end{align}
\]

根据静电学点源场理论知道，点源应该具有解电势场：\(g(\vec{r};\vec{r}_{0})=-\frac{1}{4\pi |\vec{r}-\vec{r}_{0}|}\) ，在 \(\sigma^{2}\) 无穷小曲面上有 \(\vec{r}\rightarrow \vec{r_{0}}\) ，因此有 \(g=-\frac{1}{4\pi \varepsilon},\varepsilon=|\vec{r}-\vec{r}_{0}|\rightarrow 0\) ，因此可以简化积分：

\[
\begin{align}
    \iint\limits_{\sigma^{2}}(u\frac{\partial \,g}{\partial\,n}-g\frac{\partial \,u}{\partial\,n})\mathrm{d}S &= -\iint\limits_{\sigma^{2}}\frac{u}{4\pi \varepsilon^{2}}\varepsilon^{2}\sin\theta\mathrm{d}\theta\mathrm{d}+\iint\limits_{\sigma^{2}}\frac{1}{4\pi \varepsilon}\frac{\partial u}{\partial n}\varepsilon^{2}\sin\theta\mathrm{d}\theta\mathrm{d}\varphi\varphi\notag\\
    &=-u(\vec{r}_{0})+\left.\varepsilon\frac{\partial \,u}{\partial\,n}\right|_{\sigma^{2}}=-u(\vec{r}_{0})
\end{align}
\]

由此可以得到

\[
\begin{align}
    u(\vec{r}_{0}) = \iiint\limits_{T^{3}}g(\vec{r};\vec{r}_{0})f(\vec{r})\mathrm{d}V + \iint\limits_{\Sigma^{2}}\left(u(\vec{r})\frac{\partial \,g(\vec{r};\vec{r}_{0})}{\partial\,n}-g(\vec{r};\vec{r}_{0})\frac{\partial \,u(\vec{r})}{\partial\,n}\right)\mathrm{d}S
\end{align}
\]

此时便需要使用边界问题下格林函数的对称性，令 \((\vec{r},\vec{r}_{0})\) 互换便得到解：

\[
\begin{align}
    u(\vec{r}) = \iiint\limits_{T^{3}}g(\vec{r};\vec{r}_{0})f(\vec{r}_{0})\mathrm{d}V + \iint\limits_{\Sigma^{2}}\left(u(\vec{r}_{0})\frac{\partial \,g(\vec{r};\vec{r}_{0})}{\partial\,n}-g(\vec{r};\vec{r}_{0})\frac{\partial \,u(\vec{r}_{0})}{\partial\,n}\right)\mathrm{d}S
\end{align}
\]

附加第一类与第三类边界条件，可以进一步确定表达式。

对于边界条件，有如下一般表达式：

\[
\begin{align}
    \alpha u(x)+\beta \frac{\partial u(x)}{\partial n}=\varphi(x)
\end{align}
\]

对于格林函数选择齐次的边界，同样交叉相乘后相减，那么便有：

\[
\begin{align}
    \beta \left(g\frac{\partial u(x)}{\partial n}-u\frac{\partial g}{\partial n}\right)=g\varphi(x)
\end{align}
\]

这便是利用边界条件对格林函数解的进一步确定。注意到对于第二类边界条件，考虑一个点热源（考虑静电场的高斯定律同样可以给出这个结论），则该边界意味着在边界上没有热量扩散出去，即系统绝热，这导致系统热量积累而温度不断升高，无法达成热稳定系统，因此对于格林函数而言是非物理的边界条件，要考虑第二类边界条件则必须考虑非齐次的情况（ \(\left.\frac{\partial u}{\partial n}\right|_{\Sigma}=A\) 该常数可以对微分方程运用高斯定理求出来）。

注意到，如果选择最简单的第一类齐次边界条件组合：

\[
\begin{align}
    \left.u,g\right|_{\Sigma^{2}}=0
\end{align}
\]

那么就会得到一个简单实用的表达式：

\[
\begin{align}
    u(\vec{r}) = \iiint\limits_{T^{3}}g(\vec{r};\vec{r}_{0})f(\vec{r}_{0})\mathrm{d}V
\end{align}
\]

这代表对 \(f(\vec{r}_{0})\) （满足边界的）的非零区（连续分布的源区）进行体积分，以场论理解就是，给出的是满足边界条件下的<font color=red>整个连续分布源</font> \(f(\vec{r})\) 所造成<font color=red>总的场强度</font> 。

PS：
1. 对称性证明的另一种方法：对于齐次边界问题的格林函数对称性还可以利用格林第二公式证明，方法是挖去两个点，分别对应一个格林函数，对这两个格林函数使用格林公式后，利用格林函数的调和函数性质（无源区）以及齐次边界条件便可以证出对称性。这样可以证明任意齐次边界下的对称性。

2. 关于边界条件与格林函数唯一性问题：如果公用一个边界，那么在边界内考虑不同位置的点源，看似对应不同的格林函数，实际上都是同样的物理问题，因此对应的格林函数是一样的，其满足的边界也是一样的，仅相差一个平移变换。

3. 关于函数 \(u\) 对应的边界与 \(g\) 对应的边界，公用了一套边界表达算符（即系数一样）的解释：同一个定义域，同样的微分方程算符，就应该是同样的边界表达式算符。或者可以这样理解，对于需要求解的非齐次方程，我们构造出对应的格林函数，这样一个格林函数按照我们的要求继承目标方程的一套算子且为齐次情况。

事实上有了以上这些知识已经足够在大多数物理场景使用了，剩下的就是采取何种方法构造得到格林函数的解。此外不含时的格林函数称为稳定的，后面我们还需要在介绍演化的格林函数。



## 参考文献
[1] 数学物理方法3ed，吴崇试 高春媛

[2] 数学物理方法，梁昆淼