---
title: '从0开始的EMRI'
date: 2026-05-19T00:14:49+08:00
slug: '从0开始的EMRI'
categories:
  - '综述'
tags:
  - 'EMRI'
  - '黑洞微扰理论'
  - 'BlackHoleToolKit'
image: 'DSC_7896.JPG'
toc: true
math: true
comments: true
draft: false
---
极端质量比旋进描述的是  “小天体系统”  在超大质量黑洞附近被 捕获-绕转-坠入 并不断辐射引力波的过程。通常其质量比在 10^5~10^8 之间。EMRI 作为作为天体在超大质量黑洞强场区域的动力学过程，是天然的检验强场下引力理论的实验室，是未来空基引力波探测阵列 LISA、TaiJi、TianQin 的主要科学目标之一。
<!--more-->

目前，EMRI 波形的理论建模主要有两种路径：
+ 基于黑洞微扰理论，求解由小质量天体系统的近测地运动构成源项的 Teukolsky 方程。
+ 基于小质量天体运动轨迹而直接利用引力波辐射近似公式得到 Kludge 数值波形。

前者基于微扰的理念正好适配极端质量比的情况，能够比较全面的对系统进行半解析建模；后者采用数值方法，能够快速的生成波形。而在未来的引力波探测中，往往需要快速的生成波形模板进行数据分析，因此如何快速的生成EMRI波形成为当下的主要问题，而 Kludge 波形正是应运而生。

接下来先展示如何通过黑洞微扰理论来得到EMRI的波形snapshot。
## OverLook

黑洞微扰理论描述EMRI系统主要分为三大步：

+ 构建小质量天体系统的运动轨迹，并给出对应的能动张量。
+ 求解该源项对应的Teukolsky方程，得到静态的EMRI快照。
+ 加入引力波辐射等因素造成的轨道衰减，重新迭代计算引力波形得到演化的EMRI波形。
+ 
本文内容描述了前两部分的流程，具体包含：
- 概述：
	- 从 Kerr 度规利用 NP 形式得到 Teukolsky 方程
	- 齐次径向方程的解法，得到了格林函数法的生成函数；齐次解的得到有如下几类方法：
		- MST法：利用超几何函数的级数展开构造（本文理论上概述了）
		- 基于 Heun 函数的数值方法（本文数值计算使用，因为速度比MST快近十倍）
		- SN法：考虑SN变换将径向方程变为具有短程势的形式
		- JH法：在无穷远和视界处级数展开，最后用常点处的级数展开进行拼接
- 详细推导：
	- 有源径向方程的格林函数解，最终应变波形的构造
	- 测地运动的 频率与轨迹解析解、数值计算理论
	- 利用BlackHoleToolKit 程序包数值计算实现 EMRI波形
- 下一步：
	- 学习方向：代码构建/齐次解推导/能流修正->绝热演化->EOB理论/MPD方程->自力理论
	- 研究想法：
		- 一般性的非真空轴对称Type-D度规（暗物质）/球对称薄盘度规 （Meng Kun 2026）
		- b-EMRI 有效单体描述 or 其他类似方法 （韩文标老师组大概在干这个）
		- 电磁对应：拉氏量加个电磁项/考虑 黑洞-中子双星的 EMRI/b-EMRI（报告听来的）
		- 记忆效应修正（和辐射反应的高阶修正一样弱）

其中对于Teukolsky方程的求解已经颇为成熟，主流方法是求解齐次方程后利用格林函数法求得非齐次方程的解。对于齐次Teukolsky方程，黑洞微扰理论中发展了多种方法，其中能够满足EMRI精度要求的方法有三种：MST法，SN法，Jiang-Han法。其中MST法为完全解析求解，精度最高但计算效率较慢；SN法在于将径向方程变换为类薛定谔方程形式来求解，轻巧便捷但精度较为一般；最新提出的Jiang-Han法则建立在连续分数法的基础上，在保持计算精度的前提下计算速度大幅提升。

对于小质量天体系统的动力学构建上，最早是考虑测试无自旋中性点粒子的类时测地运动，随后的MPD方程描述了带自旋的测试质量运动，以其为基础发展的Dixon形式描述了带自旋且具有内部多级矩结构的质量运动。需要注意的是，从点粒子到带自旋多极矩的测试质量，运动轨迹是从测地线到测地偏离运动发展，对应的运动方程也从具有解析解到目前只能数值求解，实际上对于EMRI系统，最难的部分就是运动轨迹的建模。

NOTE：本文表达式均遵循 BlackHoleToolKit 程序。
### Teukolsky 方程与齐次情况求解

采取NP形式可以直接推导出Teukolsky方程，对NP方程进行形式理论推导后的主要计算量在于对各个非零旋量繁杂的显式计算，不过可以用已有的 Mathematica 程序完成，此处暂时掠过（至少对于Kerr黑洞以及非真空 TYPE-D 度规而言计算过程都没有很大的困难）：

\[
\begin{align}
&\left[\frac{(r^{2}+a^{2})^{2}}{\Delta}-a^{2}\sin^{2}\theta\right]\frac{\partial^{2} {}_s\Psi}{\partial t^{2}}
+\frac{4Mar}{\Delta}\frac{\partial^{2} {}_s\Psi}{\partial t\,\partial\varphi}
+\left[\frac{a^{2}}{\Delta}-\frac{1}{\sin^{2}\theta}\right]\frac{\partial^{2} {}_s\Psi}{\partial \varphi^{2}}
\notag \\
&-\Delta^{-s}\frac{\partial}{\partial r}\left(\Delta^{s+1}\frac{\partial {}_s\Psi}{\partial r}\right)
-\frac{1}{\sin\theta}\frac{\partial}{\partial\theta}\left(\sin\theta\,\frac{\partial {}_s\Psi}{\partial \theta}\right)
\notag \\
&-2s\left[\frac{a(r-M)}{\Delta}+\frac{i\cos\theta}{\sin^{2}\theta}\right]\frac{\partial {}_s\Psi}{\partial\varphi}
-2s\left[\frac{M(r^{2}-a^{2})}{\Delta}-r-ia\cos\theta\right]\frac{\partial {}_s\Psi}{\partial t}
\notag \\
&+(s^{2}\cot^{2}\theta-s)\,{}_s\Psi
=4\pi \Sigma\,\hat{T}
\end{align}
\]

\[
\begin{align}
&\Psi(t,r,\theta,\varphi)=\int \mathrm{d}\omega \sum_{l,m}
R_{lm}(r)\,{}_sS_{lm}(\theta)\,e^{-i\omega t + im\varphi}
\\
&4\pi\Sigma \hat{T}=\int \mathrm{d}\omega \sum_{l,m}
\mathcal{T}_{lm}(r)\,{}_sS_{lm}(\theta)\,e^{-i\omega t + im\varphi}
\end{align}
\]

利用自旋加权球谐函数的正交归一性质，可以得到有源情况下的 Teukolsky 分离方程组：

\[
\begin{align}
&\Delta^{-s}\frac{\mathrm{d}}{\mathrm{d}r}
\left(\Delta^{s+1}\frac{\mathrm{d}R_{lm}}{\mathrm{d}r}\right)
+\left(\frac{K^{2}-2is(r-M)K}{\Delta}-\lambda\right)R_{lm}
=\mathcal{T}_{lm}
\notag \\
&\frac{1}{\sin\theta}\frac{\mathrm{d}}{\mathrm{d}\theta}
\left[\sin\theta\frac{\mathrm{d}{}_sS_{lm}}{\mathrm{d}\theta}\right]
+\Bigg[
-\frac{(m+s\cos\theta)^{2}}{\sin^{2}\theta}
-a^{2}\omega^{2}\sin^{2}\theta
-2a\omega s\cos\theta
\notag \\
&\qquad\qquad
+s+2ma\omega+\lambda_{lm}
\Bigg]{}_sS_{lm}=0
\notag \\
&\Delta=r^{2}-2Mr+a^{2},
\qquad
K=(r^{2}+a^{2})\omega-am,
\qquad
\Sigma=r^{2}+a^{2}\cos^{2}\theta
\end{align}
\]

注意到角向方程就是自选加权椭谐函数，对于该函数的研究较为成熟，一般直接调用现有程序包。因此绝大部分注意力都在径向方程的求解。最主要的是齐次情况的径向方程求解，对于黑洞要求满足最基本的边界条件：

\[
\begin{align} 
  &R_{lm}\sim e^{-\mathrm{i}kr_{*}}\quad,\quad r\to r_{+} \\
  &R_{lm}\sim e^{\mathrm{i}\omega r_{*}}\quad,\quad r\to\infty
\end{align}
\]

在目前主流的方法是半解析的 MST，主要使用超几何函数展开得到解：

\[
\begin{align}
 & R^{in}(x)=e^{\mathrm{i}\epsilon\kappa x}(-x)^{-s-\mathrm{i}\frac{(\epsilon+\tau)}{2}}(1-x)^{-\mathrm{i}\frac{(\epsilon+\tau)}{2}}\sum\limits_{n=-\infty}^{\infty}f_{n}{}_{2}F_{1}(n+\nu+1-\mathrm{i}\tau,-n-\nu-\mathrm{i}\tau;1-s-\mathrm{i}(\epsilon+\tau);x) \\
 & R^{up}(x)=2^{\nu}e^{-\pi\epsilon}e^{-\mathrm{i}\pi(1+s+\nu)}e^{\mathrm{i}\hat{z}}\hat{z}^{\nu+\mathrm{i}\epsilon_{+}}(\hat{z}-\epsilon\kappa)^{-s-\epsilon_{+}} \\
 & \times \sum\limits_{n=-\infty}^{\infty}\mathrm{i}^{n}\frac{\nu+1+s-\mathrm{i}\epsilon}{\nu+1-s+\mathrm{i}\epsilon}f_{n}(2\hat{z})^{n}U(n+\nu+1+s-\mathrm{i}\epsilon,2n+2\nu+2;-2\mathrm{i}z)\quad,\quad \hat{z}=\omega(r-r_{-})
\end{align}
\]

也有更轻便的替代方法：SN形式，其将Teukolsky方程进行变换为具有短程势垒的形式而求解。
近期也有具有更快速度和较好精度的JH 方法，其分别在无穷远与视界处进行级数展开，最后在常点处进行级数展开来进行匹配拼接。而数值上一般利用 Heun 函数去数值求解。总之，不管是哪种方法，最后都是产生一对满足相同边界条件的线性无关的解函数 \(R^{in}\) , \(R^{up}\) ：

\[
\begin{align} 
  &R^{in}_{lm}\longrightarrow \left\{\begin{matrix}B^{trans}_{lm}\Delta^{-s}e^{-\mathrm{i}kr_{*}}\quad,\quad r\to r_{+}\\ \\
  r^{-1-2s}B^{ref}_{lm}e^{\mathrm{i}\omega r_{*}}+r^{-1}B^{inc}_{lm}e^{-\mathrm{i}\omega r_{*}}\quad,\quad r\to \infty\end{matrix}\right.\\ \\
  &R^{up}_{lm}\longrightarrow \left\{\begin{matrix}C^{up}_{lm}e^{\mathrm{i}kr_{*}}+C^{ref}_{lm}\Delta^{-s}e^{-\mathrm{i}kr_{*}}\quad,\quad r\to r_{+}\\ \\
  r^{-1-2s}C^{trans}_{lm}e^{\mathrm{i}\omega r_{*}}\quad,\quad r\to \infty\end{matrix}\right.
\end{align}
\]

其中 `"in"` 解满足在视界处纯入射、`"up"` 解满足在无穷远纯出射 的边界条件，并且能够高效精准的计算渐近振幅：\(B^{trans},B^{ref},B^{inc};C^{trans},C^{ref},C^{up}\) 。
note：一般会令 \(C^{trans}=1\) 来归一化。 

对于需要考虑的EMRI系统，只要不对度规进行修改或者考虑高阶扰动，那么所需要的齐次解是不变的，因此以上仅先做简单介绍，具体推导细节暂时搁置。
### 有源Teukolsky 方程的求解

实际上是对非齐次径向方程求解，采用Green 函数法。因此首先概述所用到的Green函数法：

考虑格林函数满足的二阶常微分方程：

\[
\begin{align}
 & \frac{\mathrm{d}}{\mathrm{d}r}\left(p(r;\xi)\frac{\mathrm{d}G}{\mathrm{d}r}\right)+q(r;\xi)=\delta(r-\xi)
\end{align}
\]

由黑洞视界纯入射、无穷远纯出射波的边界条件，可以假设方程具有如下形式的解：

\[
\begin{align}
 & G(r;\xi)=\left\{\begin{matrix}c_{1}(\xi)y_{1}(r)\quad,\quad r_{+}<r<\xi\\c_{2}(\xi)y_{2}(r)\quad,\quad r>\xi\end{matrix}\right.
\end{align}
\]

其中 \(y_{1},y_{2}\) 是满足齐次情况的解，利用 \(\delta\) 函数的连续性方程容易得到：

\[
\begin{align}
 & c_{1}=\frac{y_{2}(\xi)}{p(\xi)w(\xi)}=\frac{y_{2}(\xi)}{W(\xi)}\quad,\quad c_{1}=\frac{y_{1}(\xi)}{p(\xi)w(\xi)}=\frac{y_{1}(\xi)}{W(\xi)}
\end{align}
\]

其中 \(W(\xi)\) 为朗斯基行列式，由边界条件可以给出：

\[
\begin{align}
 & G(r;\xi)=\left\{\begin{matrix}\frac{1}{W}R^{up}_{lm}(\xi)R^{in}_{lm}(r)\quad,\quad r_{+}<r<\xi\\\frac{1}{W}R^{in}_{lm}(\xi)R^{up}_{lm}(r)\quad\quad\quad,\quad r>\xi\end{matrix}\right.
\end{align}
\]

可以证明，朗斯基行列式是变量无关的常数，即可以证明等式 \(\left[p(r)(y_{1}y'_{2}-y'_{1}y_{2})\right]^{b}_{a}=0\) 成立，方法是考虑\(y_{1},y_{2}\)均是齐次情况方程解，联立整理后求分步积分即可。因为对于引力波辐射最终要的是无穷远处的波形，因此通常在无穷远处计算该行列式，稍作计算并整理后结果为：

\[
\begin{align}
  W(\infty)&=\left[p(r)\left(y_{1}\frac{\mathrm{d}y_{2}}{\mathrm{d}r}+y_{2}\frac{\mathrm{d}y_{1}}{\mathrm{d}r}\right)\right]_{r\to\infty} \\
 & =2\mathrm{i}\omega B^{inc}_{lm}C^{trans}_{lm}
\end{align}
\]

最后，可以证明对于一般性的边界条件，格林函数具有对称性 \((r;\xi)\leftrightarrow(\xi,r)\) ，由此可以得到非齐次径向方程的格林函数解：

\[
\begin{align}
  R_{lm}(r)&=\int^{\infty}_{r_{+}}G(r;\xi)\Delta^{s}\mathcal{T}_{lm}\mathrm{d}\xi=\int^{\infty}_{r_{+}}G(\xi;r)\Delta^{s}\mathcal{T}_{lm}\mathrm{d}\xi \\
 &= \frac{1}{W_{lm}}\left\{R^{up}_{lm}(r)\int^{r}_{_{+}}R^{in}_{lm}\Delta^{s}(\xi)\mathcal{T}_{lm}(\xi)\mathrm{d}\xi+R^{in}_{lm}(r)\int^{\infty}_{_{r}}R^{up}_{lm}\Delta^{s}(\xi)\mathcal{T}_{lm}(\xi)\mathrm{d}\xi\right\}
\end{align}
\]

注意到由此便有：

\[
\begin{align} 
  &R_{lm}(r\to\infty)= \frac{R^{up}_{lm}(r\to\infty)}{W_{lm}}\int^{r}_{_{+}}R^{in}_{lm}\Delta^{s}(\xi)\mathcal{T}_{lm}(\xi)\mathrm{d}\xi=\widetilde{Z}^{in}_{lm}R^{up}_{lm}(r\to\infty) \\
  &R_{lm}(r\to r_{+})= \frac{R^{in}_{lm}(r\to r_{+})}{W_{lm}}\int^{r}_{_{+}}R^{up}_{lm}\Delta^{s}(\xi)\mathcal{T}_{lm}(\xi)\mathrm{d}\xi=\widetilde{Z}^{up}_{lm}R^{in}_{lm}(r\to\infty)
\end{align}
\]

接下来计算 源项 \(T_{lm}\) ，由傅里叶逆变换与自旋椭谐函数的正交归一性有：

\[
\begin{align}
 	& \mathcal{T}_{lm}=\frac{1}{2\pi}\int^{+\infty}_{-\infty}\mathrm{d}t\int\mathrm{d}\Omega\;4\pi\Sigma \hat{T}e^{\mathrm{i}\omega t-\mathrm{i}m\varphi}{}_{s}{S_{lm}}(\theta)
\end{align}
\]

此处选取 \(\int\left|{}_{s}{S_{lm}}(\theta)\right|^{2}\mathrm{d}\theta=\frac{1}{2\pi}\) (与 BlackHoleToolkit 保持一致)，而源项 \(4\pi\Sigma\hat{T}\) 为：

\[
\begin{align}
  4\pi\Sigma\hat{T}&=4\pi\Sigma(2\rho^{-4}T_{4}) \\
  T_{4}& = B_{2}'+B_{2}^{*'} \\
  &=-\frac{\rho^{8}\bar{\rho}}{2}L_{-2}[\rho^{-4}L_{0}(\rho^{-2}\bar{\rho}^{-1}T_{nn})]+\frac{\Delta^{2}\rho^{8}\bar{\rho}}{2\sqrt{2}}L_{-1}[\rho^{-4}\rho^{-2}J_{+}(\rho^{-2}\bar{\rho}^{-2}\Delta^{-1}T_{n\bar{m}})] \\
  &+\frac{\Delta^{2}\rho^{8}\bar{\rho}}{2\sqrt{2}}J_{+}[\rho^{-4}\rho^{-2}\Delta^{-1}L_{-1}(\rho^{-2}\bar{\rho}^{-2}T_{n\bar{m}})]-\frac{\Delta^{2}\rho^{8}\bar{\rho}}{4}J_{+}[\rho^{-4}J_{+}(\rho^{-2}\bar{\rho}T_{\bar{m}\bar{m}})] \\
  \rho&=-\frac{1}{r-\mathrm{i}a\cos\theta}\quad,\quad\bar{\rho}=-\frac{1}{r+\mathrm{i}a\cos\theta}\quad,\quad \Delta=r^{2}-2rM+a^{2}\quad,\quad \Sigma=r^{2}+a^{2}\cos\theta \\
  J_{+}&=\frac{\partial}{\partial r}+\frac{\mathrm{i}K(r)}{\Delta}\quad,\quad L_{s}=\frac{\partial}{\partial \theta}+m\csc\theta-a\omega\sin\theta+s\cot\theta
\end{align}
\]

其中 \(T_{ab}=T^{\alpha\beta}n_{\alpha}m_{\beta}\) 便是测试粒子的能动张量在NP标架两个方向的投影。对于点粒子，其能动张量可以构造为：

\[
\begin{align}
 T^{\alpha\beta}&=\mu\int u^{\alpha}u^{\beta}\delta^{(4)}[x-z(\tau)]\mathrm{d}\tau \\
 & = \mu\int u^{\alpha}u^{\beta}(-g)^{-\frac{1}{2}}\delta[t-\widetilde{t}(\tau)]\delta[\theta-\widetilde{\theta}(\tau)]\delta[r-\widetilde{r}(\tau)]\delta[\varphi-\widetilde{\varphi}(\tau)]\mathrm{d}\tau
\end{align}
\]

其中 \(g\) 为度规对应的行列式，考虑到粒子的坐标时间与固有时间是对应的（局域上实际上就是一回事），因此能动张量化为：

\[
\begin{align}
 & T^{\alpha\beta}=\mu\frac{u^{\alpha}u^{\beta}}{\dot{t}\Sigma\sin\theta}\delta[\theta-\widetilde{\theta}(t)]\delta[r-\widetilde{r}(t)]\delta[\varphi-\widetilde{\varphi}(t)]
\end{align}
\]

而 NP 标架为：

\[
\begin{align}
 & n_{\alpha}=\frac{1}{2}\left[\frac{\Delta}{\Sigma},1,0,-\frac{a\Delta\sin^{2}\theta}{\Sigma}\right]\quad,\quad l_{\alpha}=\frac{1}{\Delta}[\Delta,-\Sigma,0,a\Delta\sin^{2}\theta] \\
 & m_{\alpha}=-\frac{\bar{\rho}}{\sqrt{2}}\left[,-\Sigma,0,a\Delta\sin^{2}\theta\right]\quad,\quad \bar{m}
\end{align}
\]

因此：

\[
\begin{align}
 & T_{ab}=\mu\frac{C_{ab}}{\sin\theta}\delta[\theta-\widetilde{\theta}(\tau)]\delta[r-\widetilde{r}(\tau)]\delta[\varphi-\widetilde{\varphi}(\tau)] \\
 & C_{nn}=\frac{\mathrm{d}\lambda}{\mathrm{d}t}\frac{1}{4\Sigma}\left[\mathcal{E}(r^{2}+a^{2})-aL_{z}+\frac{\mathrm{d}r}{\mathrm{d}\lambda}\right]^{2} \\
 & C_{n\bar{m}}=\frac{\mathrm{d}\lambda}{\mathrm{d}t}\frac{\rho}{2\sqrt{2}\Sigma}\left[\mathcal{E}(r^{2}+a^{2})-aL_{z}+\frac{\mathrm{d}r}{\mathrm{d}\lambda}\right]\left[\frac{\mathrm{d}\theta}{\mathrm{d}\lambda}+\mathrm{i}\sin\theta\left(a\mathcal{E}-\frac{L_{z}}{\sin^{2}\theta}\right)\right] \\
 & C_{\bar{m}\bar{m}}=\frac{\mathrm{d}\lambda}{\mathrm{d}t}\frac{\rho^{2}}{2\Sigma}\left[\frac{\mathrm{d}\theta}{\mathrm{d}\lambda}+\mathrm{i}\sin\theta\left(a\mathcal{E}-\frac{L_{z}}{\sin^{2}\theta}\right)\right]^{2}
\end{align}
\]

又由于：

\[
\begin{align}
  \int f(r,\theta)\delta[\varphi-\widetilde{\varphi}(t)]e^{-\mathrm{i}m\varphi}\mathrm{d}\Omega&=\int^{\pi}_{0} f(r,\theta)\mathrm{d}\theta\int^{2\pi}_{0}\delta[\varphi-\widetilde{\varphi}(t)]e^{-\mathrm{i}m\varphi}\mathrm{d}\varphi \\
 & =\int^{\pi}_{0} f(r,\theta)e^{-\mathrm{i}m\widetilde{\varphi}(t)}\mathrm{d}\theta
\end{align}
\]

接下来就是处理 \(\delta[r-\widetilde{r}],\delta[\theta-\widetilde{\theta}]\) ，因为 \(\delta\) 函数在能动张量中，而源项表达式中能动张量被算符 \(J_{+},L_{s}\) 包裹，因此需要将其运用分部积分提取出来；因为径向部分要在非齐次解中进行积分，因此首先处理角向部分，步骤为：首先对于角向部分只需要考虑将其从 \(L_{s}\) 算符中提取出来，其次将被积函数全部展开化简，最终所有的角向算符 \(L_{s},L^{+}_{s}\) 均只作用在自选加权椭谐函数  \({}_{-2}S_{lm}(\theta)\)  （后续均是 \(s=-2\)  的结果）；主要运用到了三个分部积分等式：

\[
\begin{align}
 & \int g(r,\theta)L_{s}[h(r,\theta)]\sin\theta\mathrm{d}\theta=-\int h(r,\theta)L^{+}_{1-s}[g(r,\theta)]\sin\theta\mathrm{d}\theta \\
 & J_{+}(gh)=gJ_{+}(h)+h\frac{\partial }{\partial r}g \\
 & L_{s}(gh)=gL_{s}(h)+h\frac{\partial }{\partial \theta}g
\end{align}
\]

以及一个对易关系：

\[
\begin{align}
  & [J_{+},L_{s}]=0
\end{align}
\]

以上四个关系中有：

\[
\begin{align}
   L^{+}_{s}=\frac{\partial}{\partial \theta}-m\csc\theta+a\omega\sin\theta+s\cot\theta
\end{align}
\]

其中 \(L^{+}_{s},L_{s}\) 本质是一致的，满足一样的等式关系。最后经过近十页的整理并且根据径向导数的阶数整理，可以得到结果：

\[
\begin{align}
\mathcal{T}_{lm}
&=\mu\int e^{i[\omega t-m\widetilde{\varphi}(t)]}\,\mathrm{d}t\,\Delta^{2}\Bigg\{
\\
&\Bigg[
-\frac{2\rho^{-3}\bar{\rho}^{-1}}{\Delta^{2}}C_{nn}
\left(
L^{+}_{1}L^{+}_{2}({}_{-2}S_{lm})
+2ia\sin\theta\,L^{+}_{2}({}_{-2}S_{lm})
\right)
\notag \\
&\quad
-\frac{2\sqrt{2}\rho^{-3}}{\Delta}C_{n\bar m}
\left[
\left(\frac{iK}{\Delta}-\rho-\bar{\rho}\right)L^{+}_{2}({}_{-2}S_{lm})
+\frac{iK}{\Delta}(\rho-\bar{\rho})\,ia\sin\theta\,{}_{-2}S_{lm}
\right]
\notag \\
&\quad
-\rho^{-3}\bar{\rho}\,C_{\bar m\bar m}\,{}_{-2}S_{lm}
\left[
\left(\frac{iK}{\Delta}\right)^{2}
+2\rho\frac{iK}{\Delta}
+\frac{\partial}{\partial r}\left(\frac{iK}{\Delta}\right)
\right]
\Bigg]\delta[r-\widetilde r(t)]
\notag \\
&\quad
+\frac{\mathrm{d}}{\mathrm{d}r}
\Bigg[
-\frac{2\sqrt{2}\rho^{-3}}{\Delta}C_{n\bar m}
\left(
L^{+}_{2}({}_{-2}S_{lm})
+ia\sin\theta(\rho-\bar{\rho})\,{}_{-2}S_{lm}
\right)
\notag \\
&\qquad
-2\rho^{-3}\bar{\rho}^{-1}
\left(\frac{iK}{\Delta}-\rho\right)
{}_{-2}S_{lm}C_{\bar m\bar m}
\Bigg]\delta[r-\widetilde r(t)]
\notag \\
&\quad
+\frac{\mathrm{d}^{2}}{\mathrm{d}r^{2}}
\Big[
-\rho^{-3}\bar{\rho}^{-1}\,{}_{-2}S_{lm}
C_{\bar m\bar m}\delta[r-\widetilde r(t)]
\Big]
\Bigg\}
\notag \\
&=\mu\int e^{i[\omega t-m\widetilde{\varphi}(t)]}\,\mathrm{d}t\,\Delta^{2}
\Bigg\{
A_{0}\,\delta[r-\widetilde r(t)]
+\frac{\mathrm{d}}{\mathrm{d}r}\Big[A_{1}\delta[r-\widetilde r(t)]\Big]
+\frac{\mathrm{d}^{2}}{\mathrm{d}r^{2}}\Big[A_{2}\delta[r-\widetilde r(t)]\Big]
\Bigg\}
\end{align}
\]

最后便可以计算非齐次径向方程的格林函数解：

\[
\begin{align}
  R_{lm}(r)&=\int^{\infty}_{r_{+}}G(r;\xi)\Delta^{s}\mathcal{T}_{lm}\mathrm{d}\xi=\int^{\infty}_{r_{+}}G(\xi;r)\Delta^{s}\mathcal{T}_{lm}\mathrm{d}\xi \\
 &= \frac{1}{W_{lm}}\left\{R^{up}_{lm}(r)\int^{r}_{_{+}}R^{in}_{lm}\Delta^{s}(\xi)\mathcal{T}_{lm}(\xi)\mathrm{d}\xi+R^{in}_{lm}(r)\int^{\infty}_{_{r}}R^{up}_{lm}\Delta^{s}(\xi)\mathcal{T}_{lm}(\xi)\mathrm{d}\xi\right\}
\end{align}
\]

最关心的是无穷远处的出射波，因此仅计算：

\[
\begin{align} 
  &R_{lm}(r\to\infty)= \frac{R^{up}_{lm}(r\to\infty)}{W_{lm}}\int^{r}_{_{+}}R^{in}_{lm}\Delta^{s}(\xi)\mathcal{T}_{lm}(\xi)\mathrm{d}\xi=\widetilde{Z}^{in}_{lm}R^{up}_{lm}(r\to\infty) =r^{3}e^{\mathrm{i}\omega r_{*}}\widetilde{Z}^{in}_{lm} \\
\end{align}
\]

\[
\begin{align} 
   \widetilde{Z}^{in}_{lm}&=\frac{\mu}{2\mathrm{i}\omega B^{inc}_{lm}}\int^{\infty}_{r_{+}}R^{in}_{lm}(\xi)\int^{\infty}_{-\infty} e^{\mathrm{i}[\omega t-m\widetilde{\varphi}(t)]}\mathrm{d}t\; \Big\{ \Big[A_{0}(\xi,\widetilde{\theta}(t))\delta[r-\widetilde{r}(t)]\Big] \\
 & +\frac{\mathrm{d}}{\mathrm{d}r}\Big[A_{1}(\xi,\widetilde{\theta}(t))\delta[r-\widetilde{r}(t)]\Big]+\frac{\mathrm{d}^{2}}{\mathrm{d}r^{2}}\Big[A_{2}(\xi,\widetilde{\theta}(t))\delta[r-\widetilde{r}(t)]\Big]  \Big\}\mathrm{d}r'
\end{align}
\]

利用 \(\delta\) 函数及其导数的性质

\[
\begin{align} 
  \frac{\mathrm{d}\delta(x)}{\mathrm{d}x}=0\quad,\quad x\neq 0
\end{align}
\]

最终得到振幅表达式：

\[
\begin{align} 
   \widetilde{Z}^{in}_{lm}&=\frac{\mu}{2\mathrm{i}\omega B^{inc}_{lm}}\int^{\infty}_{-\infty} e^{\mathrm{i}[\omega t-m\widetilde{\varphi}(t)]}\mathrm{d}t\; \Big\{ A_{0}R^{in}_{lm}-A_{1}\left.\frac{\mathrm{d}R^{in}_{lm}}{\mathrm{d}\xi}\right|_{\xi=\widetilde{r}(t)}+A_{0}\left.\frac{\mathrm{d}^{2}R^{in}_{lm}}{\mathrm{d}\xi^{2}}\right|_{\xi=\widetilde{r}(t)} \Big\} \\
   &=\frac{\mu}{2\mathrm{i}\omega B^{inc}_{lm}}\int^{\infty}_{-\infty} e^{\mathrm{i}[\omega t-m\widetilde{\varphi}(t)]}\mathrm{d}t\;I_{lm}
\end{align}
\]

由测地线运动可知：\(t=\Gamma\lambda+\Delta t[\widetilde{r}(\lambda),\widetilde{\theta}(\lambda)]\quad,\widetilde{\varphi}=\Upsilon_{\varphi}\lambda+\Delta\varphi[\widetilde{r}(\lambda),\widetilde{\theta}(\lambda)]\)  ，由此振幅化为：

\[
\begin{align} 
   \widetilde{Z}^{in}_{lm}
   &=\frac{\mu}{2\mathrm{i}\omega B^{inc}_{lm}}\int^{\infty}_{-\infty} e^{\mathrm{i}[\omega t-m\widetilde{\varphi}(t)]}\mathrm{d}t\;I_{lm} \\
   &=\frac{\mu}{2\mathrm{i}\omega B^{inc}_{lm}}\int^{\infty}_{-\infty} e^{\mathrm{i}\lambda[\omega\Gamma-m\Upsilon{\varphi}]}\mathrm{d}\lambda\;e^{\mathrm{i}(\omega\Delta t-m\Delta\varphi)}I_{lm}\frac{\mathrm{d}t}{\mathrm{\lambda}} \\
   &=\frac{\mu}{2\mathrm{i}\omega B^{inc}_{lm}}\int^{\infty}_{-\infty} e^{\mathrm{i}\lambda[\omega\Gamma-m\Upsilon{\varphi}]}\mathrm{d}\lambda\;J_{lm}
\end{align}
\]

对于被积分函数 \(J_{lm}\) 同样由于角向与径向的可分离性，能够进行傅里叶展开：

\[
\begin{align}
 & J_{lm}=\sum_{k,n}J_{lmnk}e^{\mathrm{i}(n\Upsilon_{r}+k\Upsilon_{\theta})\lambda} \quad,\quad \omega_{r}=n\Upsilon_{r},\omega_{\theta}=n\Upsilon_{\theta}\\
 & J_{lmnk}=\frac{1}{4\pi^{2}}\int^{2\pi}_{0}\mathrm{d}\omega_{r}\int^{2\pi}_{0}\mathrm{d}\omega_{\theta}\;e^{\mathrm{i}(n \omega_{r}+k \omega_{\theta})}J_{lm}[r(\omega_{r}),\theta(\omega_{\theta}),r,\lambda]
\end{align}
\]

最后借助 \(\delta\) 函数的傅里叶变换方程得到振幅最终表达式：

\[
\begin{align} 
   \widetilde{Z}^{in}_{lm}&=\frac{\mu}{2\mathrm{i}\omega B^{inc}_{lm}}\int^{\infty}_{-\infty} e^{\mathrm{i}\lambda[\omega\Gamma-m\Upsilon{\varphi}]}\mathrm{d}\lambda\;J_{lm} \\
   &=\frac{\mu}{2\mathrm{i}\omega B^{inc}_{lm}}\int^{\infty}_{-\infty} \sum_{n,k}e^{\mathrm{i}\lambda[(\omega\Gamma-m\Upsilon{\varphi})-(n\Upsilon_{r}+k\Upsilon_{\theta})]}J_{lmnk}\mathrm{d}\lambda \\
   &=\frac{\mu}{2\mathrm{i}\omega B^{inc}_{lm}}\int^{\infty}_{-\infty} \sum_{n,k}e^{\mathrm{i}\lambda\Gamma[\omega-(m\frac{\Upsilon{\varphi}}{\Gamma}+n\frac{\Upsilon_{r}}{\Gamma}+k\frac{\Upsilon_{\theta}}{\Gamma})]}J_{lmnk}\mathrm{d}\lambda \\
   &=\frac{\mu}{2\mathrm{i}\omega B^{inc}_{lm}}\sum_{n,k}\frac{2\pi J_{lmkn}}{\Gamma}\delta(\omega-\omega_{mnk})\quad,\quad \omega_{mnk}=m\frac{\Upsilon{\varphi}}{\Gamma}+n\frac{\Upsilon_{r}}{\Gamma}+k\frac{\Upsilon_{\theta}}{\Gamma}
\end{align}
\]

最终无穷远处的波动函数写为：

\[
\begin{align}
	\psi^{r\to\infty}_{4}&=\rho^{4}\sum\limits_{l,m,n,k}\int\mathrm{d}\omega R_{lm}(r){}_{s}{S_{lm}}(\theta)e^{-\mathrm{i}\omega t + \mathrm{i}m\varphi} \\
	&=\frac{\mu}{r}\sum\limits_{l,m,n,k}\frac{e^{-\mathrm{i}\omega_{mnk}(t-r_{*})}}{2\mathrm{i}\omega_{mnk} B^{inc}_{lm}}\frac{2\pi J_{lmkn}}{\Gamma}{}_{-2}{S^{a\omega}_{lm}}(\theta,\varphi)
\end{align}
\]

对应的引力波应变为：

\[
\begin{align}
	h_{+}-\mathrm{i}h_{\times}&=-\frac{2\psi^{r\to\infty}_{4}}{\omega^{2}_{mnk}} \\
	&=-\frac{2\mu}{r}\sum\limits_{l,m,n,k}\frac{e^{-\mathrm{i}\omega_{mnk}(t-r_{*})}}{\omega^{2}_{mnk}}\frac{2\pi J_{lmkn}}{2\mathrm{i}\omega_{mnk} B^{inc}_{lm}\Gamma}{}_{-2}{S^{a\omega}_{lm}}(\theta,\varphi) \\
	&=\frac{2\mu}{r}\sum\limits_{l,m,n,k}\frac{e^{-\mathrm{i}\omega_{mnk}(t-r_{*})}}{\omega^{2}_{mnk}}\widetilde{Z}^{in}_{lmnk}{}_{-2}{S^{a\omega}_{lm}}(\theta,\varphi)
\end{align}
\]

至此便完成了对于无演化的波形快照建模，可以发现在数值计算上需要做的就是计算那个产生于傅里叶展开的二重积分，也是这个积分的计算需要调用径向方程的齐次解，因此该积分的计算速度几乎是等于波形的计算速度，该积分的收敛性基本决定了波形的收敛性，对于径向上当高偏心可能出现的积分发散也是体现在这个二重积分上。

最后关于初值的问题，当不在转折点开始时，需要加上一个初始相位，体现在时间上便是，如果从转折点起步，则是 \(\lambda=0\) 开始运动，而如果具有初始相位，则不再从转折点起步，此时计时应该是从 \(\lambda=\lambda_{0}\) 开始，令 \(\lambda=\widetilde{\lambda}+\lambda_{0}\) ，其中起始有 \(\widetilde{\lambda}=0\) ，对应到波形计算过程便是存在一个相位修正：

\[
\begin{align} 
   \widetilde{Z}^{in}_{lm}\Longrightarrow \widetilde{Z}^{in}_{lm}e^{\mathrm{i}[\omega(q_{t_{0}}-\Delta t_{0}-n\omega_{r_{0}})-m(q_{\varphi_{0}}-\Delta \varphi_{0}+k\omega_{r_{0}})]}
\end{align}
\]

不过一般考虑初始相位等于 0 。
### 测试粒子的轨迹 —— 能动张量的构造

#### Kerr 度规

在BL系下Kerr度规写为：

\[
\begin{align}
	\mathrm{d}s^{2} &= -(1-\frac{2Mr}{\rho^{2}})\mathrm{d}t^{2} + \frac{\rho^{2}}{\Delta}\mathrm{d}r^{2} + \rho^{2}\mathrm{d}\theta^{2} \notag\\&+ \left[(r^{2}+a^{2})\sin^{2}\theta + \frac{2Mra^{2}\sin^{4}\theta}{\rho^{2}}\right]\mathrm{d}\varphi^{2} - \frac{4 M r a\sin^{2}\theta}{\rho^{2}}\mathrm{d}t \mathrm{d}\varphi \\&
	\rho^{2}\equiv r^{2}+a^{2}\cos^{2}\theta,\quad \Delta \equiv r^{2}-2Mr +a^{2}
\end{align}
\]

其具有两个 Killing 矢量：\(k^{\mu},m^{\mu}\) ，对应 \(t,\varphi\) 两个时空对称性，即能量 \(E\) 与角动量 \(L_{z}\) 守恒。

此外还存在一个隐藏的 Killing 矢量，对应 Carter 常数，后续可以在哈密顿-雅可比理论框架下利用分离变量将其给出。

#### 运动方程

考虑一个测试点粒子，我们的目的便是得到其类时测地线方程。哈密顿-雅可比框架是一个简单快速的方法，并且可以直接得到 Carter 常数；接下来我们便采用这一方法导出测地线。


点源的拉格朗日量为（对应为测试粒子的拉氏密度，对应的守恒量也应为密度）：

\[
\begin{aligned}
	\mathcal{L}=\frac{1}{2}g_{\mu\nu}\dot{x}^{\mu}\dot{x}^{\nu}
\end{aligned}
\]

容易发现由于两个 Killing 矢量的存在，对应的 \(t,\varphi\) 是循环坐标，正是对应了两个守恒量：能量密度 \(\mathcal{E}\) 与角动量密度：\(L_{z}\) 。利用 Killing 矢量的性质可以直接给出,其中 \(\mu=0,1,2,3\)：

\[
\begin{align}
    \mathcal{E} = -u^{\mu}k_{\mu} = -u_{t} = -g_{t\mu}u^{\mu} = A\;\dot{t}+ B\; \dot{\varphi}\\
    L_{z} = u^{\mu}m_{\mu} = u_{\varphi}=g_{\varphi\mu}u^{\mu}= -B\;\dot{t}+ C\; \dot{\varphi}
\end{align}
\]

自然可以反解出 \(\dot{t},\;\dot{\varphi}\) :

\[
\begin{align}
	\frac{\mathrm{d}t}{\mathrm{d}\tau}&=\frac{C \mathcal{E} - B L_{z}}{\Delta\sin^{2}\theta}=\frac{1}{\Delta}\left[(r^{2}+a^{2}+\frac{2M r a^{2}}{\Sigma}\sin^{2}\theta)\mathcal{E} - \frac{2 M r a}{\Sigma} L_{z}\right]\\
	\frac{\mathrm{d}\varphi}{\mathrm{d}\tau}&=\frac{B \mathcal{E} + A L_{z}}{\Delta\sin^{2}\theta}=\frac{1}{\Delta}\left[(1-\frac{2M r}{\Sigma})\frac{L_{z}}{\sin^{2}\theta} + \frac{2 M r a}{\Sigma} \mathcal{E} \right]
\end{align}
\]

接着考虑 哈密顿-雅可比方程：

\[
\begin{align}
	\frac{\partial S}{\partial \lambda}+H(x^{\mu},\frac{\partial S}{\partial x^{\mu}})=0
\end{align}
\]

这是个四维方程，具有四个待求变量，因此如果解 \(S\) 依赖于四个独立的常数，那么便构成完全积分，由此便有：

\[
\begin{align}
\frac{\partial S}{\partial x^{\mu}}=p_{\mu}
\end{align}
\]

由欧拉-拉格朗日方程，可以得到粒子的正则动量：

\[
p_{\mu}=\frac{\partial \mathcal{L}}{\partial \dot{x}^{\mu}}=g_{\mu\nu}\dot{x}^{\mu}
\]

因此对应的哈密顿量（密度）可以通过勒让德变换得到：

\[
H(x^{\mu},p_{\nu})=p_{\mu}\dot{x}^{\mu}(p_{\nu})-\mathcal{L}(x^{\mu},\dot{x}^{\mu}(p_{\nu}))=\frac{1}{2}g^{\mu\nu}p_{\mu}p_{\nu}
\]

则容易由四速度的归一性： \(g_{mu\nu}\dot{x}^{\mu}\dot{x}^{\nu}=-1\) 得到三个常数：

\[
\begin{align}
	H(x^{\mu},p_{\nu}) &= \frac{1}{2}\kappa = const\\
	p_{t}&=-\mathcal{E}\\
	p_{\varphi}&=L_{z}
\end{align}
\]

为使得系统可积，接下来寻找第四个常数，将作用量泛函设为：
\[S=-\frac{1}{2}\kappa-\mathcal{E} t+L_{z}\varphi+S^{r\theta}(r,\theta)\]
容易发现是符合已有守恒量要求的，接着假设可以进行分离变量（以此分离常数便是需要寻找的第四个常数）：

\[
\begin{align}
	S^{r\theta}(r,\theta)=S^{r}(r)+S^{\theta}(\theta)
\end{align}
\]

因此可以由：

\[
\begin{align}
	\frac{\partial S}{\partial \lambda}+H(x^{\mu},\frac{\partial S}{\partial x^{\mu}})=-\frac{1}{2}\kappa + \frac{1}{2} g^{\mu\nu}\frac{\partial S}{\partial x^{\mu}}\frac{\partial S}{\partial x^{\nu}}=0
\end{align}
\]

得到方程：

\[
\begin{align}
	-\kappa + \frac{\Delta}{\Sigma}(\frac{\mathrm{d}S^{r}}{\mathrm{d}r})^{2} + \frac{1}{\Sigma}(\frac{\mathrm{d}S^{\theta}}{\mathrm{d}\theta})^{2} - \frac{1}{\Delta}\left[r^{2}+a^{2}+\frac{2Mra^{2}}{\Sigma}\sin^{2}\theta\right]\mathcal{E}^{2} + \frac{4Mra}{\Sigma}\mathcal{E}L_{z} + \frac{\Delta-a^{2}\sin^{2}\theta}{\Sigma \Delta \sin^{2}\theta}L_{z}=0
\end{align}
\]

这是可分离的，化简得到：

\[
\begin{align}
	&\Delta(\frac{\mathrm{d}S^{r}}{\mathrm{d}r})^{2}-\kappa r^{2} - \frac{1}{\Delta}\left[\mathcal{E}(r^{2}+a^{2})-aL_{z}\right]^{2}+(L_{z}-a\mathcal{E})^{2}=-Q\\
	&(\frac{\mathrm{d}S^{\theta}}{\mathrm{d}\theta})^{2}-\cos^{2}\theta\left[\mathcal{E}^{2}a^{2}+\kappa a^{2}-\frac{L_{z}^{2}}{1-\cos^{2}\theta}\right] = Q
\end{align}
\]

其中 \(Q\) 称之为 Carter 常数（也可以定义：\(q=Q+(L_{z}-a\mathcal{E})^{2}\) 为 Carter 常数）。由此可以进一步利用：\(\frac{\partial S}{\partial x^{\mu}}=p_{\mu}=g_{\mu\nu}\dot{x}^{\nu}\) 得到径向与角向的运动方程：

\[
\begin{align}
	(g_{rr}\frac{\mathrm{d}r}{\mathrm{d}\tau})^{2}&=\left(\frac{\Sigma}{\Delta}\frac{\mathrm{d}r}{\mathrm{d}\tau}\right)^{2}=\frac{1}{\Delta^{2}}\left[\mathcal{E}(r^{2}+a^{2})-aL_{z}\right]^{2} + \frac{\kappa r^{2} + -(L_{z}-a\mathcal{E})^{2}-Q}{\Delta}\\
	(g_{\theta\theta}\frac{\mathrm{d}\theta}{\mathrm{d}\tau})^{2}&=\left(\Sigma\frac{\mathrm{d}\theta}{\mathrm{d}\tau}\right)^{2}= \cos^{2}\theta\left[\mathcal{E}^{2}a^{2}+\kappa a^{2}-\frac{L_{z}^{2}}{1-\cos^{2}\theta}\right] + Q
\end{align}
\]

我们还希望把运动方程中对角向与径向的依赖函数分离开来，而引入 Mino time: \(\lambda\) 便可以做到：

\[
\begin{align}
	\mathrm{d}\tau=\Sigma\mathrm{d}\lambda
\end{align}
\]

由此运动方程为：

\[
\begin{align}
	&\left(\frac{\mathrm{d}r}{\mathrm{d}\lambda}\right)^{2}=\left[\mathcal{E}(r^{2}+a^{2})-aL_{z}\right]^{2} + \Delta[\kappa r^{2} + -(L_{z}-a\mathcal{E})^{2}-Q]\\
	&\left(\frac{\mathrm{d}\theta}{\mathrm{d}\lambda}\right)^{2}= \cos^{2}\theta\left[\mathcal{E}^{2}a^{2}+\kappa a^{2}-\frac{L_{z}^{2}}{1-\cos^{2}\theta}\right] + Q
\end{align}
\]

为了方便后续展开，对角向方程做如下变换：\(\mathrm{d}\cos\theta = -\sin\theta\mathrm{d}\theta\) ，则：

\[
\begin{align}
	\left(\frac{\mathrm{d}\cos\theta}{\mathrm{d}\lambda}\right)^{2}&= Q - \left[Q-a^{2}(\mathcal{E}^{2}+\kappa) + L_{z}^{2}\right]\cos^{2}\theta - a^{2}(\mathcal{E}^{2}+\kappa)\cos^{4}\theta\\
	&=\Theta(\cos\theta)\\
	\left(\frac{\mathrm{d}r}{\mathrm{d}\lambda}\right)^{2}& = R(r)
\end{align}
\]

已知讨论的是束缚椭圆轨道，因此存在运动转折点，径向体现为近心点 \(r_{p}\) 与远心点 \(r_{a}\) ，角向体现为升降点： \(\theta_{min},\pi-\theta_{min}\) ，由于径向与角向的分离性，可以独立对待两个运动，因此在各自的转折点均应该有：

\[
\begin{align}
	&\left.\frac{\mathrm{d}\cos\theta}{\mathrm{d}\lambda}\right|_{\theta_{min},\pi-\theta_{min}} = 0\\
	&\left.\frac{\mathrm{d}r}{\mathrm{d}\lambda}\right|_{r_{a},r_{p}} = 0
\end{align}
\]

这是转折点，运动方程在此发生变号，这也正是体现在平方号上。转折点的存在使得运动方程被迫分段，通常只能在半个周期内进行计算，需要手动改变正负号才能够计算整个周期的运动。

对于 \(t,\varphi\) 方向的运动，由于径向与角向的可分离性因此也可以写为径向部分与角向部分，进行这般处理后便可以得到点源的测地线方程：

\[
\begin{align}
	&\left(\frac{\mathrm{d}\cos\theta}{\mathrm{d}\lambda}\right)^{2} =\Theta(\cos\theta)=Q - \left[Q-a^{2}(\mathcal{E}^{2}+\kappa) + L_{z}^{2}\right]\cos^{2}\theta - a^{2}(\mathcal{E}^{2}+\kappa)\cos^{4}\theta\\
	&\left(\frac{\mathrm{d}r}{\mathrm{d}\lambda}\right)^{2} = R(r)=\left[\mathcal{E}(r^{2}+a^{2})-aL_{z}\right]^{2} + \Delta[\kappa r^{2} + -(L_{z}-a\mathcal{E})^{2}-Q]\\
	&\frac{\mathrm{d}t}{\mathrm{d}\lambda}= T_{r}+T_{\theta}=\frac{\mathcal{E}(r^{2}+a^{2})^{2}}{\Delta}-\frac{(r^{2}+a^{2})aL_{z}}{\Delta}+aL_{z}-a^{2}\mathcal{E}(1-\cos^{2}\theta)\\
	&\frac{\mathrm{d}\varphi}{\mathrm{d}\lambda}= \Phi_{r}+\Phi_{\theta}=\frac{a\mathcal{E}(r^{2}+a^{2})-a^{2}L_{z}}{\Delta}-a\mathcal{E}+\frac{L_{z}}{1-\cos^{2}\theta}
\end{align}
\]

束缚轨道的Kerr类时测地运动的数值计算的办法由考虑对 \(\theta,r\) 进行变量替换消除远点与近点的变号问题而解决，而解析的办法直至2012年才由 Fujita & hikita 给出，为不失一般性，接下来将从数值方法开始介绍。

#### 转折点的处理

不管是数值计算还是解析计算，对于转折点的处理本质上是相同的，均是将束缚轨道的振荡特性等价转换为周期函数的增加。
对 \(\theta\) 方向，由于 \(\theta\in[\theta_{min},\pi-\theta_{min}]\) ，因此 \(\cos\theta\) 仅能在 \(\cos\theta_{min}-0-\cos(\pi-\theta_{min})-0-\cos\theta_{min}\) 来回振荡，尽管变量是周期的，但是由于 \(\cos\theta_{min}<1\) 使得函数只能在半个周期的区间内来回振荡，变好正是对应了”折返“；而考虑变换：\(\cos\theta_{min}\cos\chi=\cos\theta\) ，则可以解出限制，新变量 \(\chi\in[0,2\pi]\) 且对应有 \(\cos\chi\) : \(1\rightarrow0\rightarrow-1\rightarrow0\rightarrow1\rightarrow\cdots\) ，自然发现对于 N 个周期的运动，对应着 \(\chi\in[0,N\pi]\) 。至于运动方向，有：

\[
\begin{align}
	&\mathrm{d}\chi/\mathrm{d}\theta=\frac{\sin\theta}{\cos\theta_{min}\sin\chi}\\\\
	&\frac{\mathrm{d}\theta}{\mathrm{d}\lambda}=\frac{\mathrm{d}\theta}{\mathrm{d}\chi}\frac{\mathrm{d}\chi}{\mathrm{d}\lambda}=\pm\sqrt{\Theta}
\end{align}
\]

变换前，由于运动方向会发生改变，而始终有 \(\sqrt{\Theta}>0\) ，因此需要进行变号来保证运动方程，变换后可以发现：
+ 当  \(\theta:\theta_{min}\rightarrow \pi-\theta_{min}\) ，\(\mathrm{d}\theta/\mathrm{d}\lambda>0,\mathrm{d}\theta/\mathrm{d}\chi>0\)  
+ 当  \(\theta:\pi-\theta_{min}\rightarrow \theta_{min}\) ，\(\mathrm{d}\theta/\mathrm{d}\lambda<0,\mathrm{d}\theta/\mathrm{d}\chi<0\)   
可以看到 \(\mathrm{d}\theta/\mathrm{d}\chi\) 将使得运动方程始终可以保持符号不发生改变（或说已经自动编码了变号）。
对于 \(r\) 方向，可以用椭圆轨道常见的形式来代换：
\[r=\frac{pM}{1-e\cos\psi}\]
可见  \((r_{p},r_{a})\rightarrow(\psi=0,\psi=\pi)\) ，同样有：
\[
\begin{align}
	\mathrm{d}r/\mathrm{d}\psi=\frac{pM}{(1-e\cos\psi)^{2}}e\sin\psi
\end{align}
\]
可见变号过程已经被编码在了 \(\sin\psi\) ，对于一个周期： \(\psi\in[0,2\pi]\) 。
这两个变换在数值计算使用，而对于解析表达，则使用的是具有周期性的椭圆函数及其反函数来编码此类 “往返” 周期性运动，其中的道理是相同的，在后面将揭示这一点。
#### 运动频率
##### 傅里叶展开
由于径向与角向的分离性，且束缚轨道的周期特性，只需要考虑半个周期的运动便可以得到轨道的频率信息，因此直接有：
\[
\begin{align}
	\Lambda_{r}&=2\int^{r_{a}}_{r_{p}}\frac{\mathrm{d}r}{\sqrt{R}}=\frac{2\pi}{\Upsilon_{r}}\\
	\Lambda_{\theta}&=4\int^{\cos\theta_{min}}_{0=\cos\frac{\pi}{2}}\frac{\mathrm{d}\cos\theta}{\sqrt{\Theta}}=\frac{2\pi}{\Upsilon_{\theta}}
\end{align}
\]
对于 \(t,\varphi\) 运动，由于 \(r,\theta\) 的独立性，可以进行如下傅里叶展开：
\[
\begin{align}
	\frac{\mathrm{d}t}{\mathrm{d}\lambda}&=T_{r}+T_{\theta}+aL_{z}=T(r,\theta)=\sum_{n,k}T_{nk}e^{-\mathrm{i}(n\Upsilon_{r}+k\Upsilon_{\theta})\lambda}\\
	T_{nk}&=\frac{1}{(2\pi)^{2}}\int^{2\pi}_{0}\mathrm{d}\omega_{r}\int^{2\pi}_{0}\mathrm{d}\omega_{\theta}T(r,\theta)e^{\mathrm{i}(n\omega_{r}+k\omega_{\theta})}
\end{align}
\]
其中相位变量 \(\omega_{·}=\Upsilon_{·}\lambda\) ，展开系数 \(T_{nk}\) 是一个确定的实数（有意义），由傅里叶基的正交归一性：
\[
\begin{align}
	\int^{2\pi}_{0}e^{\mathrm{i}n\omega}\mathrm{d}\omega=\left\{\begin{matrix}0\;,\quad n\neq0 \\\\ 2\pi\;,\quad n=0\end{matrix}\right.
\end{align}
\]
展开系数写为：
\[
\begin{align}
	T_{nk}=\frac{1}{(2\pi)^{2}}\left[\int^{2\pi}_{0}T_{r}e^{\mathrm{i}n\omega_{r}}\mathrm{d}\omega_{r}\int^{2\pi}_{0}e^{\mathrm{i}k\omega_{\theta}}\mathrm{d}\omega_{\theta} + \int^{2\pi}_{0}e^{\mathrm{i}n\omega_{r}}\mathrm{d}\omega_{r}\int^{2\pi}_{0}T_{\theta}e^{\mathrm{i}k\omega_{\theta}}\mathrm{d}\omega_{\theta} + aL_{z}\int^{2\pi}_{0}e^{\mathrm{i}n\omega_{r}}\mathrm{d}\omega_{r}\int^{2\pi}_{0}e^{\mathrm{i}k\omega_{\theta}}\mathrm{d}\omega_{\theta} \right]
\end{align}
\]
因此系数具有对称性：
\[
\begin{align}
	T_{-n,-k}=\overline{T}_{nk}=T_{nk},\quad T_{00}\neq0,\quad T_{0k},T_{n0}\neq0
\end{align}
\]
注意由于周期性，因此自然有 \(T_{-n,-k}=T_{nk}\) ，这直接等价于取 \(-\omega\) ，由此容易得到：
\[
\begin{align}
	\sum_{n,k}^{\pm\infty}T_{nk}e^{-\mathrm{i}(n\omega_{r}+k\omega_{\theta})}=\sum_{n,k=0}^{\infty}\left[T_{nk}e^{-\mathrm{i}(n\omega_{r}+k\omega_{\theta})}+\overline{T}_{nk}e^{\mathrm{i}(n\omega_{r}+k\omega_{\theta})}\right]
\end{align}
\]
以及非零项表达式：
\[
\begin{align}
	T_{00}&=\frac{1}{2\pi}\int^{2\pi}_{0}T_{r}[r(\omega_{r})]\mathrm{d}\omega_{r}+\frac{1}{2\pi}\int^{2\pi}_{0}T_{\theta}[\theta(\omega_{\theta})]\mathrm{d}\omega_{\theta}+aL_{z}\\
	T_{n0}&=\frac{1}{2\pi}\int^{2\pi}_{0}T_{r}[r(\omega_{r})]e^{\mathrm{i}\omega_{r}n}\mathrm{d}\omega_{r}\quad,\quad T_{0k}=\frac{1}{2\pi}\int^{2\pi}_{0}T_{\theta}[\theta(\omega_{\theta})]e^{\mathrm{i}\omega_{\theta}k}\mathrm{d}\omega_{\theta}
\end{align}
\]
因此运动方程化为：
\[
\begin{align}
	\mathrm{d}t&=T_{00}+\left[\sum^{+\infty}_{n=1}T_{n0}e^{-\mathrm{i}\omega_{r}n}+\sum^{+\infty}_{k=1}T_{0k}e^{-\mathrm{i}\omega_{\theta}k}+c.c.\right]\mathrm{d}\lambda\\
	t&=T_{00}\lambda+\left[\sum^{+\infty}_{n=1}\frac{\mathrm{i}T_{n0}}{n\Upsilon_{r}}e^{-\mathrm{i}n\Upsilon_{r}}+c.c.\right]+\left[\sum^{+\infty}_{k=1}\frac{\mathrm{i}T_{0k}}{k\Upsilon_{\theta}}e^{-\mathrm{i}k\Upsilon_{\theta}}+c.c.\right]\\
	&=\Gamma\lambda+\Delta t=\Gamma\lambda+\Delta t^{(r)}+\Delta t^{(\theta)}
\end{align}
\]
其中 \(\Gamma\) 代表一个周期增长的相位，是时间频率，\(\Delta t\) 是周期运动过程中的小振动产生的相位差；最后注意到：
\[
\begin{align}
	\frac{1}{2\pi}\int^{2\pi}_{0}T_{r}[r(\omega_{r})]\mathrm{d}\omega_{r}=\frac{\Upsilon_r}{2\pi}\int^{\Lambda_{r}}_{0}\frac{T_{r}}{\sqrt{R}}\mathrm{d}\lambda=\left<T_{r}\right>_{\lambda}=\frac{2}{\Lambda_{r}}\int^{\Lambda_{r}/2}_{0}\frac{T_{r}}{\sqrt{R}}\mathrm{d}\lambda
\end{align}
\]
因此也有表达式：
\[
\begin{align}
\Gamma=T_{00}=\left<T_{r}\right>_{\lambda}+\left<T_{\theta}\right>_{\lambda}+aL_{z}=aL_{z}+\Upsilon_{t^{(r)}}+\Upsilon_{t^{(\theta)}}
\end{align}
\]
可令：
\[
\begin{align}
	\frac{\mathrm{d}t}{\mathrm{d}\lambda}&=\left<T_{r}\right>_{\lambda}+\left<T_{\theta}\right>_{\lambda}+aL_{z}+\frac{\mathrm{d}t^{(r)}}{\mathrm{d}\lambda}+\frac{\mathrm{d}t^{(\theta)}}{\mathrm{d}\lambda}\\
	\frac{\mathrm{d}t^{(r)}}{\mathrm{d}\lambda}&=\frac{\mathrm{d}\Delta t^{(r)}}{\mathrm{d}\lambda}=T_{r}-\left<T_{r}\right>_{\lambda}\\
	t^{(r)}&=\int^{r}\frac{T_{r}-\left<T_{r}\right>_{\lambda}}{\sqrt{R(r')}}\mathrm{d}r'
\end{align}
\]
对于  \(\varphi\)  方向完全一样。至此就可以开始数值计算了，但距离得到解析解还需要用椭圆积分来表示前面出现的几个积分。
##### 角向与径向频率解析表达
对于解析计算，首先解析求解角向频率，令 \(z=\cos\theta\) ：
\[
\begin{align}
	\frac{\mathrm{d}\cos\theta}{\mathrm{d}\lambda}&=\pm\sqrt{\Theta}=(z^{2}-z^{2}_{-})\left[a^{2}(1-\mathcal{E}^{2})z^{2}-a^{2}(1-\mathcal{E}^{2})z^{2}_{+}\right]\\&=(z^{2}-z^{2}_{m})\left[\beta z^{2}-z_{p}^{2}\right]=(z^{2}-z^{2}_{m})(\beta z^{2}-z^{2}_{p})
\end{align}
\]
其中 \(z^{2}_{m}=\cos\theta_{min},z^{2}_{p}=\beta z^{2}_{+}=\frac{Q}{z^{2}_{-}}=\beta+\frac{L^{2}_{z}}{1-z^{2}_{m}}\) ，需要注意，转折点仅对应 \(z=z_{-}=\cos^{2}\theta_{min}\) 这两个角度，另外两个根 \(z=z_{+}\) 没有物理意义，一般而言是虚数或者大于0； 再令 \(z=z_{m}y_{\theta}\) ，则角向周期可表为：
\[
\begin{align}
	\Lambda_{\theta}&=4\int^{z_{m}}_{0}\frac{\mathrm{d}z}{z_{m}z_{p}\sqrt{(1-\frac{z^{2}}{z^{2}_{m}})(1-\beta\frac{z^{2}}{z^{2}_{p}})}}=\frac{4}{z_{m}z_{p}}\int^{1}_{0}\frac{z_{m}\mathrm{d}y_{\theta}}{z_{m}z_{p}\sqrt{(1-y^{2})(1-\beta\frac{z_{m}^{2}}{z^{2}_{p}}y^{2}_{\theta})}}\\
	&=\frac{4}{z_{p}}\int^{\frac{\pi}{2}}_{0}\frac{\mathrm{d}\psi}{\sqrt{1-k^{2}_{\theta}\sin^{2}\phi}}=\frac{4}{z_{p}}K(k^{2}_{\theta})
\end{align}
\]
其中 \(K(m)\) 为第一类完全椭圆积分（其中 \(m_{r}=k^{2}_{r}\) ，后面均用 \(m_{r/\theta}\) 表示），倒数第二个等号使用了：\(y_{\theta}=\sin\phi\)  ，\(m=k^{2}_{\theta}=\beta\frac{z^{2}_{m}}{z^{2}_{p}}\) 。 
对于径向同理，考虑如下代换：
\[
\begin{align}
	y_{r}=\sqrt{\frac{r-r_{2}}{r-r_{3}}\frac{r_{1}-r_{3}}{r_{1}-r_{2}}}\;,r_{1}&=\frac{pM}{1-e}\;,r_{2}=\frac{pM}{1-e}\;,r_{3}=\frac{A+B+\sqrt{(A+B)^{2}-4AB}}{2}\;, r_{4}=\frac{AB}{r_{3}}\\
	A+B&=\frac{2M}{1-\mathcal{E}^{2}}-(r_{1}+r_{2})\;,AB=\frac{a^{2}Q}{(1-\mathcal{E}^{2})r_{1}r_{2}}
\end{align}
\]
以及将运动方程改写为（\(r_{1}>r_{2}>r_{3}>r_{4}\)）：
\[
\begin{align}
	\frac{\mathrm{d}r}{\mathrm{d}\lambda}=\sqrt{R}=(1-\mathcal{E}^{2})(r_{1}-r)(r-r_{2})(r-r_{2})(r-r_{4})
\end{align}
\]
同样需要注意，按这个写法实际上有 \((r_{2},r_{1}),(r_{4},r_{3})\) 两个束缚轨道区间，但是一般而言 \(r_{4}\) 在视界内部，因此并不是有意义的物理束缚轨道；与角向积分相同，径向频率可以写为：
\[
\begin{align}
	\Lambda_{r}=2\int^{r_{1}}_{r_{2}}\frac{\mathrm{d}r}{\sqrt{R}}&=2\frac{2}{\sqrt{(1-\mathcal{E}^{2})(r_{1}-r_{3})(r_{2}-r_{4})}}\int^{1}_{0}\frac{\mathrm{d}y_{r}}{\sqrt{(1-y_{r}^{2})(1-k^{2}_{r}y_{r}^{2})}}\;,\quad k^{2}_{r}=\frac{r_{3}-r_{4}}{r_{2}-r_{4}}\frac{r_{1}-r_{2}}{r_{1}-r_{3}}\\
	&=\frac{2}{\sqrt{(1-\mathcal{E}^{2})(r_{1}-r_{3})(r_{2}-r_{4})}}\int^{\frac{\pi}{2}}_{0}\frac{\mathrm{d}\psi}{\sqrt{1-k^{2}_{r}\sin^{2}\phi}}\;,\quad y_{r}=\sin\phi\\
	&=\frac{2}{\sqrt{(1-\mathcal{E}^{2})(r_{1}-r_{3})(r_{2}-r_{4})}}K(m_{r})
\end{align}
\]
##### 时间与轴向方向频率的解析表达
而 \(t,\varphi\) 方向的频率为：
\[
\begin{align}
	\Gamma &= \frac{2}{\Lambda_{r}}\int^{r_{1}}_{r_{2}}\frac{T_{r}}{\sqrt{R}}\mathrm{d}r + \frac{4}{\Lambda}\int^{\cos\theta_{min}}_{0}\frac{T_{\theta}}{\sqrt{\Theta}}\mathrm{d}\cos\theta +  aL_{z}=\Upsilon_{t^{(r)}}+\Upsilon_{t^{(\theta)}}+aL_{z}\\
	\Upsilon_{\varphi}&=\frac{2}{\Lambda_{r}}\int^{r_{1}}_{r_{2}}\frac{\Phi_{r}}{\sqrt{R}}\mathrm{d}r + \frac{4}{\Lambda}\int^{\cos\theta_{min}}_{0}\frac{\Phi_{\theta}}{\sqrt{\Theta}}\mathrm{d}\cos\theta - a\mathcal{E}=\Upsilon_{\varphi^{(r)}}+\Upsilon_{\varphi^{(\theta)}}-a\mathcal{E}
\end{align}
\]
角向部分只存在 \(z\) 的一次项，容易积分，直接计算即可，结果为：
\[
\begin{align}
	\Upsilon_{t^{(\theta)}}&=\frac{2a^{2}\mathcal{E}\Upsilon_{\theta}}{\pi\beta z_{p}}\left[(z^{2}_{p}-\beta)K(m_{\theta})-z^{2}_{p}E(m_{\theta})\right]\\
	\Upsilon_{\varphi^{(\theta)}}&=\frac{2L_{z}\Upsilon_{\theta}}{\pi z_{p}}\Pi(\frac{\pi}{2},-z^{2}_{m},m_{\theta})
\end{align}
\]
其中 \(E(m),\Pi(\frac{\pi}{2},c,m)\) 依次为第二、三类完全椭圆积分；对于径向部分则由于二次项的存在而复杂得多，因此需要利用多项式展开将 \(r^{2}\) 降阶为 \(r\) 或 \(r^{-1}\) ，具体的可以展开为：
\[
\begin{align}
	T_{r}&=\mathcal{E}r^{2}+2Mr\mathcal{E}+(a^{2}+4M^{2})\mathcal{E}-aL_{z}+\frac{2M}{r_{+}-r_{-}}\left\{\frac{(4M^{2}\mathcal{E}-aL_{z})r_{+}-2M^{2}a^{2}\mathcal{E}}{r-r_{+}}-(+\leftrightarrow-)\right\},\quad a<M\\
	&=4M^{2}\mathcal{E}+\frac{2M(4M^{2}\mathcal{E}-aL_{z})}{r-M}+\frac{2M^{2}(2M^{2}\mathcal{E}-aL_{z})}{(r-M)^{2}},\quad a=M\\\\
	\Phi_{r}&=a\mathcal{E}+\frac{a}{r_{+}-r_{-}}\left(\frac{2Ma\mathcal{E}r_{+}-aL_{z}}{r-r_{+}}-\frac{2Ma\mathcal{E}r_{-}-aL_{z}}{r-r_{-}}\right),\quad a<M\\
	&=a\mathcal{E}+\frac{2Ma\mathcal{E}}{r-M}+\frac{2M^{2}a\mathcal{E}-aL_{z}}{(r-M)^{2}},\quad a=M
\end{align}
\]
由此径向积分由以下几个积分组成：
\[
\begin{align}
	\int^{r_{1}}_{r_{2}}\frac{1}{\sqrt{R}}\mathrm{d}r\quad,\quad\int^{r_{1}}_{r_{2}}\frac{r}{\sqrt{R}}\mathrm{d}r\quad,\quad\int^{r_{1}}_{r_{2}}\frac{r^{2}}{\sqrt{R}}\mathrm{d}r\quad,\quad\int^{r_{1}}_{r_{2}}\frac{1}{(r-r_{\pm})\sqrt{R}}\mathrm{d}r\quad,\quad\int^{r_{1}}_{r_{2}}\frac{1}{(r-r_{\pm})^{2}\sqrt{R}}\mathrm{d}r
\end{align}
\]
第一项就是径向周期，第二项与第四项为一阶项，也可以直接计算得到：
\[
\begin{align}
	&\int^{r_{1}}_{r_{2}}\frac{r}{\sqrt{R}}\mathrm{d}r = \frac{2(r_{2}-r_{3})}{\sqrt{(1-\mathcal{E}^{2})(r_{1}-r_{3})(r_{2}-r_{4})}}\Pi(\frac{\pi}{2},-h_{r},k^{2}_{r})+\frac{2r_{3}}{\sqrt{(1-\mathcal{E}^{2})(r_{1}-r_{3})(r_{2}-r_{4})}}K(k^{2}_{}r)\\
	&\int^{r_{1}}_{r_{2}}\frac{1}{(r-r_{\pm})\sqrt{R}}\mathrm{d}r=\frac{2}{\sqrt{(1-\mathcal{E}^{2})(r_{1}-r_{3})(r_{2}-r_{4})}}\left[K(k^{2}_{r})-\frac{r_{2}-r_{3}}{r_{2}-r_{\pm}}\Pi(\frac{\pi}{2},-h_{\pm},k^{2}_{r})\right]\\
	&h_{r}=\frac{r_{1}-r_{2}}{r_{1}-r_{3}},h_{\pm}=h_{r}\frac{r_{3}-r_{\pm}}{r_{2}-r_{\pm}}
\end{align}
\]
对于二阶项，已知通过变量替换有：
\[
\begin{align}
	\sqrt{R}\sim \sqrt{\psi}=\sqrt{(1-y_{r}^{2})(1-k_{r}^{2}y_{r}^{2})}
\end{align}
\]
我们希望用三椭圆积分来表示目标积分，而第三类椭圆积分在特殊情况下可以转化为另外两类，而第三类椭圆积分为：
\[
\begin{align}
	\Pi(\phi,-h,m)=\int^{\phi}_{0}\frac{\mathrm{d}y}{(1-hy^{2})\sqrt{\psi}}
\end{align}
\]
由于面对的被积函数是二阶的，可以预计会出现二次项的分母 \(\sim (1-hy^{2})^{2}\) ，事实上正是如此，我们可以展开 \(r^{2}\) ：
\[
\begin{align}
	&r^{2}=r_{3}^{2}+\frac{2r_{3}(r_{2}-r_{3})}{1-h^{r}y^{2}_{r}}+\frac{(r_{2}-r_{3})^{2}}{(1-h_{r}y^{2}_{r})^{2}}
\end{align}
\]
注意到其中可以进一步展开：
\[
\begin{align}
	\frac{1}{(1-h_{r}y^{2}_{r})^{2}}=\frac{A}{(y_{r}-h^{-\frac{1}{2}}_{r})^{2}}+\frac{B}{(y_{r}+h^{-\frac{1}{2}}_{r})^{2}}
\end{align}
\]
这样分母便被降阶为了：\(y^{2}_{r}\) 项，已经接近第三类椭圆积分了；最终便有：
\[
\begin{align}
	r^{2}=r_{3}^{2}+\frac{(r_{2}-r_{3})(r_{2}+3r_{3})}{2}\frac{1}{1-h^{r}y^{2}_{r}}+\frac{(r_{2}-r_{3})^{2}}{4h_{r}}\left[\frac{1}{(y_{r}-h^{-\frac{1}{2}}_{r})^{2}}+\frac{1}{(y_{r}+h^{-\frac{1}{2}}_{r})^{2}}\right]
\end{align}
\]
对于 \((r-M)^{-2}\) 同理：
\[
\begin{align}
	&\left(\frac{r_{3}-M}{r-M}\right)^{2}=\notag\\
	&1+\frac{1}{2}\left(4-\frac{r_{2}-M}{r_{2}-M}\right)\left(\frac{r_{2}-r_{3}}{r_{2}-M}\right)\frac{1}{1-h^{r}y^{2}_{r}} + \frac{(\frac{r_{2}-r_{3}}{r_{2}-M})^{2}}{4h_{r}}\left[\frac{1}{(y_{r}-h^{-\frac{1}{2}}_{r})^{2}}+\frac{1}{(y_{r}+h^{-\frac{1}{2}}_{r})^{2}}\right]
\end{align}
\]
接下来的问题就是对展开式的最后一项积分，恰恰发现在椭圆积分的递推关系有一个一般表达式：
\[
\begin{align}
	J_{n}(c)=\int\frac{\mathrm{d}y}{(y-c)^{n}\sqrt{\psi(y)}}
\end{align}
\]
因此我们需要求的是：\(J_{2}[c]+J_{2}[-c]\) ，这实际上是求：\(\int\frac{\mathrm{d}y}{(y^{2}-c^{2})^{2}\sqrt{\psi}}\) ，而这可以通过构造：
\[
\begin{align}
	\frac{\mathrm{d}}{\mathrm{d}y}\left(\frac{y\sqrt{\psi}}{(y^{2}-c^{2})}\right)=\frac{\mathrm{d}}{\mathrm{d}y}\left(\frac{y}{(y^{2}-c^{2})}\sqrt{\psi}\right)=\frac{\mathrm{d}}{\mathrm{d}y}\left(fg\right)
\end{align}
\]
来解决，经过分部积分便可以导出等式：
\[
\begin{align}
	J_{2}[c]+J_{2}[-c]&=\\
	&	\frac{2}{\sqrt{\psi(c)}}\left\{[k^{2}_{r}(2c^{2}-1)-1]\Pi(\phi,-c^{-2},m)+(1-c^{2}k^{2}_{r})F(\phi,m)-E(\phi,m)-\left.\frac{y\sqrt{\psi}}{y^{2}-c^{2}}\right|^{y}_{0}\right\}
\end{align}
\]
最终借此可以给出 \(\Upsilon_{t^{(r)}},\Upsilon_{\varphi^{(r)}}\) ，由此最终给出（\(a<M\)）频率的解析表达式：
\[
\begin{align}
	\Gamma&=4M^{2}\mathcal{E}+\frac{2a^{2}\mathcal{E}\Upsilon_{\theta}z_{p}}{\pi\beta }\left[K(m_{\theta})-E(m_{\theta})\right]\notag\\
	&+\frac{2\Upsilon_{r}}{\pi\sqrt{(1-\mathcal{E}^{2})(r_{1}-r_{3})(r_{2}-r_{4})}}\Big\{2M\mathcal{E}[r_{3}K(m_{r})+(r_{2}-r_{3})\Pi(\frac{\pi}{2},-h_{r},m_{r})] \notag\\
	&+ \frac{\mathcal{E}}{2}\Big[\Big(r_{3}(r_{1}+r_{2}+r_{3})-r_{1}r_{2}\Big)K(m_{r})+(r_{1}-r_{3})(r_{2}-r_{4})E(m_{r})\notag\\
	&+(r_{2}-r_{3})(r_{1}+r_{2}+r_{3}+r_{4})\Pi(\frac{\pi}{2},-h_{r},m_{r})\Big]\notag\\
	&+ \Big[\frac{2M}{(r_{+}-r_{-})}\Big((4M^{2}\mathcal{E}-aL_{z})r_{+}-2M^{2}a^{2}\mathcal{E}\Big)\Big(\frac{K(m_{r})}{r_{3}-r_{-}}-\frac{(r_{2}-r_{3})\Pi(\frac{\pi}{2},-h_{+},m_{r})}{(r_{2}-r_{+})(r_{3}-r_{+})}\Big) - (+\leftrightarrow-)\Big] \Big\}\\\notag\\
	\Upsilon_{\varphi}&=\frac{2L_{z}\Upsilon_{\theta}}{\pi z_{p}}\Pi(\frac{\pi}{2},-z^{2}_{m},m_{\theta})\notag\\
	&+ \frac{2\Upsilon_{r}}{\pi\sqrt{(1-\mathcal{E}^{2})(r_{1}-r_{3})(r_{2}-r_{4})}}\frac{a}{r_{+}-r_{-}}\Big[\frac{2Ma\mathcal{E}r_{+}-aL_{z}}{r_{3}-r_{+}}\Big(K(m_{r})-\frac{r_{2}-r_{3}}{r_{2}-r_{+}}\Pi(\frac{\pi}{2},-h_{+},m_{r})\Big)\notag\\
	&-(+\leftrightarrow -)\Big]\\\notag\\
	\Upsilon_{r}&=\frac{\pi\sqrt{(1-\mathcal{E}^{2})(r_{1}-r_{3})(r_{2}-r_{4})}}{2K(m_{r})}\\\notag\\
	\Upsilon_{\theta}&=\frac{\pi z_{p}}{2K(m_{\theta})}
\end{align}
\]
由于篇幅太长暂时就不考虑 \(a=M\) 情况，日后再做补充。
#### 运动轨迹
接下来可以求解运动轨迹的解析表达式，首先需要的是选定一个演化起始点，存在两个选择：上行区与下行区，由两个转折点分开，对应为：\(\frac{\mathrm{d}r}{\mathrm{d}\lambda}>0,\frac{\mathrm{d}r}{\mathrm{d}\lambda}<0\) 。我们暂时只展示起始点位于上行区的情况，因为下行区与上行区本质上只存在一个相位差。此外由于周期性，我们只需要讨论最后一个周期内的运动就可以完成整个系统全时间运动的描述。
##### 径向运动
选取初始点：\(r_{0}^{(1)}\)  ，其中 \((1)\) 代表选择的上行区的初值点。对于上行区，有  \(sgn(L_{z})>0,\left.\frac{\mathrm{d}r}{\mathrm{d}\lambda}\right|_{\lambda=0}\) ，从运动方程可以得到累计的Mino时间为：
\[
\begin{align}
	\lambda=\lambda^{(r)}=\int^{r}_{r_{0}^{(1)}}\frac{\mathrm{d}r'}{\sqrt{R}}
\end{align}
\]
其中 \(\lambda^{(r)}\) 用于标记这是由径向运动标记的时间；考虑运动的顺序为：\(r_{0}^{(1)}\rightarrow r_{1}\rightarrow r_{2}\rightarrow r_{0}^{(1)}\) ，因此分三段来表示上述积分： 
\[
\begin{align}
	\lambda^{(r)}=\left\{\begin{matrix}(\int^{r}_{r_{2}}-\int^{r_{0}^{(1)}}_{r_{2}})\frac{\mathrm{d}r'}{\sqrt{R}}\quad,\quad\quad\quad\quad r_{0}^{(1)}\rightarrow r_{1}\\(-\int^{r}_{r_{2}}+2\int^{r_{1}}_{r_{2}}-\int^{r_{0}^{(1)}}_{r_{2}})\frac{\mathrm{d}r'}{\sqrt{R}}\quad,\quad r_{1}\rightarrow r_{2}\\
	(\int^{r}_{r_{2}}+2\int^{r_{1}}_{r_{2}}-\int^{r_{0}^{(1)}}_{r_{2}})\frac{\mathrm{d}r'}{\sqrt{R}} \quad,\quad r_{2}\rightarrow r_{0}^{(1)}\end{matrix}\right.
\end{align}
\]
现在令：
\[
\begin{align}
	\lambda^{(r)}_{0}(r)&=\frac{2}{\sqrt{(1-\mathcal{E}^{2})(r_{1}-r_{3})(r_{2}-r_{4})}}F(\phi,m_{r})=\frac{2}{\sqrt{(1-\mathcal{E}^{2})(r_{1}-r_{3})(r_{2}-r_{4})}} u_{r}(r)=\int^{r}_{r_{2}}\frac{\mathrm{d}r'}{\sqrt{R}}\\
	\Lambda^{(1)}_{0}&=\int^{r^{(1)}_{0}}_{r_{2}}\frac{\mathrm{d}r'}{\sqrt{R}}\quad,\quad \alpha=\frac{2}{\sqrt{(1-\mathcal{E}^{2})(r_{1}-r_{3})(r_{2}-r_{4})}}
\end{align}
\]
则可以将Mino时间用一系列周期函数表示为：
\[
\begin{align}
	\lambda^{(r)}=\left\{\begin{matrix}\lambda^{(r)}_{0}-\Lambda^{(1)}_{r}\quad,\quad\quad\quad\quad r_{0}^{(1)}\rightarrow r_{1}\\
	-\lambda^{(r)}_{0}+\Lambda_{r}-\Lambda^{(1)}_{r}\quad,\quad r_{1}\rightarrow r_{2}\\
	\lambda^{(r)}_{0}+\Lambda_{r}-\Lambda^{(1)}_{r} \quad,\quad r_{2}\rightarrow r_{0}^{(1)}\end{matrix}\right.
\end{align}
\]
又从径向频率可以有：
\[
\begin{align}
	\Lambda_{r}=2\alpha K(m_{r})\Rightarrow \alpha=\frac{\Lambda_{r}}{2K}
\end{align}
\]
因此可以定义一个新函数：
\[
\begin{align}
	u_{r}=F(\phi,m_{r})=\frac{2K}{\Lambda_{r}}\lambda^{(r)}_{0}
\end{align}
\]
当然 \(u_{r}\) 也是分段表出的，将时间的分段表述反解出来即可，在此就先不多赘述。引入 \(u_{r}\) 最重要的一个作用是得到坐标 \(r\) 的解析表达，首先可以发现，\(u_{r}\) 表征了从初始点开始积累的相位：
\[
\begin{align}
	u_{r}=\frac{K}{\pi}\Upsilon_{r}\lambda^{(r)}_{0}=\frac{K}{\pi}q_{r}
\end{align}
\]
可以通过椭圆函数的反函数来提取坐标表达式：
\[
\begin{align}
	&u_{r}=F(\arctan y_{r},m_{r})
	\quad \Rightarrow\quad \arctan y_{r}=\mathrm{am}[u_{r},m_{r}]\\\notag\\
	&\Rightarrow \quad y_{r}=\sqrt{\frac{r-r_{2}}{r-r_{3}}\frac{r_{1}-r_{3}}{r_{1}-r_{2}}}=\mathrm{sn}(u_{r},m_{r})=\mathrm{sn}(\frac{K}{\pi}q_{r},m_{r})\\\notag\\
	&\quad r(\lambda)=\frac{r_{3}(r_{1}-r_{2})\mathrm{sn}^{2}-r_{2}(r_{1}-r_{3})}{(r_{1}-r_{2})\mathrm{sn}^{2}-(r_{1}-r_{3})}
\end{align}
\]
其中 \(q_{r}(\lambda)\) 便是相位，我们可以直接以此为新变量来计算运动轨迹。这里也可以看到，\(\mathrm{sn}\) 函数是周期性的，其完整编码了径向的周期性往返运动！
##### 角向运动
角向运动与径向运动类似，同样考虑上行区的起始点：\(\theta_{0}^{(1)}\) ，\(\left.\frac{\mathrm{d}\cos\theta}{\mathrm{d}\lambda}\right|_{\lambda=0}>0\) ；
运动轨迹为：\(\theta_{0}^{(1)}\rightarrow 0\rightarrow \theta_{min}\rightarrow 0\rightarrow\pi-\theta_{min}\rightarrow\theta^{(1)}_{0}\) ，同样由角向运动计量的Mino时间为：
\[
\begin{align}
	\lambda^{(\theta)}=\left\{\begin{matrix}(\int^{\cos\theta}_{0}-\int^{\cos\theta_{0}^{(1)}}_{0})\frac{\mathrm{d}\cos\theta'}{\sqrt{\Theta}}\quad,\quad\quad\quad\quad \theta_{0}^{(1)}\rightarrow \theta_{min}\\
	(-\int^{\cos\theta}_{0}+2\int^{\cos\theta_{min}}_{0}-\int^{\cos\theta^{(1)}_{0}}_{0})\frac{\mathrm{d}\cos\theta'}{\sqrt{\Theta}}\quad,\quad \theta_{min}\rightarrow \pi-\theta_{min}\\
	(\int^{\cos\theta}_{0}+4\int^{\cos\theta_{min}}_{0}-\int^{\cos\theta^{(1)}_{0}}_{0})\frac{\mathrm{d}\cos\theta'}{\sqrt{\Theta}} \quad,\quad \pi-\theta_{min}\rightarrow \theta_{0}^{(1)}\end{matrix}\right.
\end{align}
\]
同样令：
\[
\begin{align}
 & \lambda^{(\theta)}_{0}=\int^{\cos\theta}_{0}\frac{\mathrm{d}\cos\theta'}{\sqrt{\Theta}}=\frac{F(\arcsin y_{\theta},m_{\theta})}{z_{p}}\quad,\quad \Lambda^{(1)}_{\theta}=\int^{\cos\theta^{(1)}_{0}}_{0}\frac{\mathrm{d}\cos\theta'}{\sqrt{\Theta}}
\end{align}
\]
因此也有：
\[
\begin{align}
	\lambda^{(\theta)}=\left\{\begin{matrix}\lambda^{(\theta)}_{0}-\Lambda^{(1)}_{\theta}\quad,\quad\quad\quad\quad \theta_{0}^{(1)}\rightarrow \theta_{min}\\
	-\lambda^{(\theta)}_{0}+\frac{\Lambda_{\theta}}{2}-\Lambda^{(1)}_{\theta}\quad,\quad \theta_{min}\rightarrow \pi-\theta_{min}\\
	\lambda^{(\theta)}_{0}+\Lambda_{\theta}-\Lambda^{(1)}_{\theta} \quad,\quad \pi-\theta_{min}\rightarrow \theta_{0}^{(1)}\end{matrix}\right.
\end{align}
\]
同样的可以通过引入函数 \(u_{\theta}\) 来推导 \(\cos\theta\) 的表达式：
\[
\begin{align}
  & u_{\theta}=F(\arcsin y_{\theta},m_{\theta})=z_{p}\lambda^{(\theta)}_{0}=\frac{2K}{\pi}\Upsilon_{\theta}\lambda^{(\theta)}_{0}=\frac{2K}{\pi}q_{\theta} \\ \notag\\
  & y_{\theta}=\frac{\cos\theta}{\cos\theta_{min}}=\mathrm{sn}(u_{\theta},m_{\theta})\quad\Rightarrow\quad \cos\theta=\cos\theta_{min}\mathrm{sn}(\frac{2K}{\pi}q_{\theta},m_{\theta})
\end{align}
\]
##### 初值选取
最后是关于初值的选取，首先需要说明的是，椭圆积分是允许相位超过 \(\frac{\pi}{2}\) 的，对于一个周期而言，需要椭圆积分的积分相位为：\(r:(0,\pi),\theta:(0,2\pi)\) ，又因为椭圆积分本身的被积函数是大于等于0且周期为 \(\pi\) 的且关于极值点对称的函数，因此椭圆积分本身具有 \(\frac{\pi}{2}\) 周期。因此恰好可以描述所需要的轨道相位演化。
其次，因为坐标时间是统一的，因此有：
\[
\begin{align}
 & \lambda=\lambda^{(r)}=\lambda^{(\theta)}
\end{align}
\]
为了简单起见，选取转折点 \(r_{2},\theta_{min}\) 为初始点，由此便不必刻意区分上行或者下行，进一步取径向时间作为基准，则有：
\[
\begin{align}
 & \lambda=\lambda^{(r)}=\lambda^{(r)}_{0}\quad\Rightarrow\quad q_{r}=\Upsilon_{r}\lambda^{(r)}_{0}=\Upsilon_{r}\lambda\\ \\
 & \lambda=\lambda^{(\theta)}=\lambda^{(\theta)}_{0} - \frac{\Lambda_{\theta}}{4}\\ \\
 & \Rightarrow\quad q_{\theta}=\Upsilon_{\theta}\lambda^{(\theta)}_{0}=\Upsilon_{\theta}\lambda + \frac{\pi}{2}
\end{align}
\]
由此确定了数值计算时径向与角向的相位变量。如果初始点不在转折点，那么便是从 \((r_{0},\theta_{0})\) 开始计时，变换到上述使用转折点开始只需要减去一个沿着轨道方向的相对 \((r_{1},\theta_{min})\) 的相位差即可。
##### 时间与轴向运动
从前面的傅里叶展开出发：
\[
\begin{align}
	t&=T_{00}\lambda+\left[\sum^{+\infty}_{n=1}\frac{\mathrm{i}T_{n0}}{n\Upsilon_{r}}e^{-\mathrm{i}n\Upsilon_{r}}+c.c.\right]+\left[\sum^{+\infty}_{k=1}\frac{\mathrm{i}T_{0k}}{k\Upsilon_{\theta}}e^{-\mathrm{i}k\Upsilon_{\theta}}+c.c.\right]\\
	&=\Gamma\lambda+\Delta t=\Gamma\lambda+\Delta t^{(r)}+\Delta t^{(\theta)}=\Gamma\lambda + t^{(r)}+ t^{(\theta)}\\ \\
	\varphi&=\Phi_{00}\lambda+\left[\sum^{+\infty}_{n=1}\frac{\mathrm{i}\Phi_{n0}}{n\Upsilon_{r}}e^{-\mathrm{i}n\Upsilon_{r}}+c.c.\right]+\left[\sum^{+\infty}_{k=1}\frac{\mathrm{i}\Phi_{0k}}{k\Upsilon_{\theta}}e^{-\mathrm{i}k\Upsilon_{\theta}}+c.c.\right]\\
	&=\Upsilon_{\varphi}\lambda+\Delta \varphi=\Upsilon\lambda+\Delta \varphi^{(r)}+\Delta \varphi^{(\theta)}=\Upsilon\lambda+ \varphi^{(r)}+ \varphi^{(\theta)}
\end{align}
\]
结合上一节对 \(r,\cos\theta\) 解析表达式的求解，显然发现
\[
\begin{align}
 & \omega_{r}= q_{r} \quad,\quad \omega_{\theta}=q_{\theta}+\frac{\pi}{2}
\end{align}
\]
因此至此就可以对 \(t,\varphi\) 运动数值求解了，但是为了得到解析表达，还需要进一步处理
\[
\begin{align}
 & t^{(r)}=\int^{r}_{r_{0}}\frac{T_{r}-\Upsilon_{t^{(r)}}}{\sqrt{R(r')}}\mathrm{d}r'=\int^{r}_{r_{0}}\frac{T_{r}}{\sqrt{R(r')}}\mathrm{d}r'-\frac{2}{\Lambda_{r_{1}}}\int^{r}_{r_{2}}\frac{T_{r}}{\sqrt{R(r')}}\mathrm{d}r'\int^{r}_{r_{0}}\frac{\mathrm{d}r'}{\sqrt{R(r')}} \\
 & t^{(r)}=\int^{r}_{r_{0}}\frac{T_{r}-\Upsilon_{t^{(r)}}}{\sqrt{R(r')}}\mathrm{d}r'=\int^{r}_{r_{0}}\frac{T_{r}}{\sqrt{R(r')}}\mathrm{d}r'-\frac{2}{\Lambda_{r_{1}}}\int^{r}_{r_{2}}\frac{T_{r}}{\sqrt{R(r')}}\mathrm{d}r'\int^{r}_{r_{0}}\frac{\mathrm{d}r'}{\sqrt{R(r')}}
\end{align}
\]
对于 \(\varphi\) 同理，经过复杂运算可以给出：
\[
\begin{align}
	t^{(r)}&=\frac{\alpha\mathcal{E}}{2}(r_{1}-r_{3})(r_{2}-r_{4})\widetilde{E}_{r}(\phi,-h_{r},m_{r})+\frac{\alpha\mathcal{E}}{2}(r_{2}-r_{3})\left[4M+(r_{1}+r_{2}+r_{3}+r_{4})\right]\widetilde{\Pi}_{r}(\phi,-h_{r},m_{r}) \notag\\
	&-\alpha\left[\frac{2M(2Mr^{2}_{+}\mathcal{E}-aL_{z}r_{+})(r_{2}-r_{3})}{(r_{+}-r_{-})(r_{3}-r_{+})(r_{2}-r_{+})}\widetilde{\Pi}_{r}(\phi,-h_{+},m_{r})-(+\leftrightarrow-)\right] \\
	\varphi^{(r)}&=-\frac{a\alpha}{r_{+}-r_{-}}\left[\frac{2Ma\mathcal{E}r_{+}-aL_{z}}{r_{3}-r_{+}}\frac{r_{2}-r_{3}}{r_{2}-r_{+}}\widetilde{\Pi}_{r}(\phi,-h_{+},m_{r})-(+\leftrightarrow-)\right] \\
	t^{(\theta)}&=-\frac{a^{2}\mathcal{E}z_{p}}{\beta}\widetilde{E}_{\theta}(\phi,m_{\theta})\quad,\quad \varphi^{(\theta)}=\frac{L_{z}}{z_{p}}\widetilde(\phi,-z^{2}_{m},m_{\theta})
\end{align}
\]
其中（以上行区的分段积分为例子，对于数值计算时取转折点为起始点则可以直接套用）：
\[
\begin{align}
 & \widetilde{E}_{r}(\phi,-h_{r},m_{r})=E_{r}(\phi,-h_{r},m_{r})-\frac{\Upsilon_{r}\lambda^{(r)}}{\pi}E(m_{r}) \\
 & E_{r}(\phi,-h_{r},m_{r})=\int^{\phi}_{\phi_{0}}\sqrt{1-m_{r}\sin^{2}\phi'}\mathrm{d}\phi' + \left.\frac{y_{r}\sqrt{\psi}}{y^{2}_{r}-h^{-1}_{r}}\right|^{y_{r}}_{y_{0}} \\
 & =\left(E(\phi,m_{r})+\left.\frac{y_{r}\sqrt{\psi}}{y^{2}_{r}-h^{-1}_{r}}\right|^{y_{r}}_{0}\right)-\left(E(\phi_{0},m_{r})+\left.\frac{y_{r}\sqrt{\psi}}{y^{2}_{r}-h^{-1}_{r}}\right|^{y_{0}}_{0}\right) \\
 & = E^{(0)}_{r}(\phi,-h_{r},m_{r})-E^{(0)}_{r}(\phi_{0},-h_{r},m_{r})\quad\quad \quad\quad\quad\quad\quad\quad\quad\quad,\quad r_{0}\to r_{1} \\
 & =2E^{(0)}_{r}(\frac{\pi}{2},-h_{r},m_{r})-E^{(0)}_{r}(\phi,-h_{r},m_{r})-E^{(0)}_{r}(\phi_{0},-h_{r},m_{r})\quad,\quad r_{1}\to r_{2} \\
 & =2E^{(0)}_{r}(\frac{\pi}{2},-h_{r},m_{r})-E^{(0)}_{r}(\phi,-h_{r},m_{r})+E^{(0)}_{r}(\phi_{0},-h_{r},m_{r})\quad,\quad r_{2}\to r_{0} \\
 &  \notag\\
 & \widetilde{\Pi}_{r}(\phi,-h_{r/\pm},m_{r})=\Pi_{r}(\phi,-h_{r/\pm},m_{r})-\frac{\Upsilon_{r}\lambda^{(r)}}{\pi}\Pi_{r}(\phi,-h_{r/\pm},m_{r}) \\
 & \Pi_{r}(\phi,-h_{r/\pm},m_{r})=\Pi(\phi,-h_{r/\pm},m_{r})-\Pi(\phi_{0},-h_{r/\pm},m_{r})\quad,\quad r_{0}\to r_{1} \\
 & =2\Pi(\frac{\pi}{2},-h_{r/\pm},m_{r})-\Pi(\phi,-h_{r/\pm},m_{r})-\Pi(\phi_{0},-h_{r/\pm},m_{r})\quad,\quad r_{1}\to r_{2} \\
 & =2\Pi(\frac{\pi}{2},-h_{r/\pm},m_{r})-\Pi(\phi,-h_{r/\pm},m_{r})+\Pi(\phi_{0},-h_{r/\pm},m_{r})\quad,\quad r_{2}\to r_{0} \\
 &  \notag\\
 & \widetilde{E}_{\theta}(\phi,m_{\theta})=E_{\theta}(\phi,m_{\theta})-\frac{2\Upsilon_{\theta}}{\pi}\lambda^{(\theta)}E(\frac{\pi}{2},m_{\theta}) \\
 & E_{\theta}(\phi,m_{\theta})=E(\phi,m_{\theta})-E(\phi_{0},m_{\theta}) \quad,\quad \theta_{0}\to \theta_{min}\\
 & =-E(\phi,m_{\theta})+2E(\frac{\pi}{2},m_{\theta})-E_(\phi_{0},m_{\theta})\quad,\quad \theta_{min}\to \pi-\theta_{min} \\
 & = E(\phi,m_{\theta})+4E(\frac{\pi}{2},m_{\theta})-E_(\phi_{0},m_{\theta})\quad,\quad \pi-\theta_{min}\to \theta_{0}  \\
 & \widetilde{\Pi}_{\theta}(\phi,-z^{2}_{m},m_{\theta})=\Pi_{\theta}(\phi,-z^{2}_{m},m_{\theta})-\frac{2\Upsilon_{\theta}}{\pi}\lambda^{(\theta)}\Pi(\frac{\pi}{2},-z^{2}_{m},m_{\theta}) \\
 & \Pi_{\theta}(\phi,-z^{2}_{m},m_{\theta})=\Pi(\phi,-z^{2}_{m},m_{\theta})-\Pi(\phi_{0},-z^{2}_{m},m_{\theta}) \quad,\quad \theta_{0}\to \theta_{min}\\
 & =-\Pi(\phi,-z^{2}_{m},m_{\theta})+2\Pi(\frac{\pi}{2},-z^{2}_{m},m_{\theta})-\Pi_(\phi_{0},-z^{2}_{m},m_{\theta})\quad,\quad \theta_{min}\to \pi-\theta_{min} \\
 & = \Pi(\phi,-z^{2}_{m},m_{\theta})+4\Pi(\frac{\pi}{2},-z^{2}_{m},m_{\theta})-\Pi_(\phi_{0},-z^{2}_{m},m_{\theta})\quad,\quad \pi-\theta_{min}\to \theta_{0} 
\end{align}
\]
至此已经完成对于类时束缚轨道测地线的运动解析求解（以及提供了数值求解的理论基础）。
对于数值计算法，理论上有：直接四阶龙格库塔求解四个线性方程或者更粗暴直接数值积分，以及采取本文所讲述的计算傅里叶级数的半解析积分。
#### 从观测量到守恒量
直接从运动方程出发，以及转折点的构造给出观测量 \((p,e,\cos\iota)\) 与 守恒量 \((\hat{E},\hat{L}_{Z},\hat{Q})\) 之间的映射。
其中
\[
\begin{align}
 & a=\frac{\hat{a}}{M}\quad,\quad \mathcal{E}=\frac{\hat{E}}{\mu}\quad,\quad L_{z}=\frac{\hat{L_{z}}}{\mu M}\quad,\quad Q=\frac{\hat{Q}}{\mu^{2}M^{2}}
\end{align}
\]
对于倾斜角 \(\iota\) 定义为在赤道为 0，沿着转动方向时为 九十度 ，大于 九十度 时标志着运动方向与黑洞自旋方向相反，即逆行；因此有：
\[
\begin{align}
 & \iota=\frac{\pi}{2}-sgn(L_{z})\theta_{min}
\end{align}
\]
且有 :
\[
\begin{align}
 & \Theta=0 \quad\Rightarrow\quad Q=\cos^{2}\theta_{min}\left(\beta+\frac{L^{2}_{z}}{1-\cos^{2}\theta_{min}}\right)
\end{align}
\]
再带入径向方程，考虑转折点处的方程以及圆轨道情况，可以解出：
\[
\begin{align}
 & \mathcal{E}^{2}=\frac{\kappa\rho+2\epsilon\sigma+2(sgnL_{z})\sqrt{\sigma(\sigma\epsilon^{2}+\rho\kappa\epsilon-\eta\kappa^{2})}}{\rho^{2}+4\eta\sigma} \\
 & L_{z}=-\frac{g_{1}\mathcal{E}}{h_{1}}+(sgnL_{z})\sqrt{\frac{g^{2}_{1}\mathcal{E}^{2}}{h^{2}_{1}}+\frac{f_{1}\mathcal{E}^{2}-d_{1}}{h_{1}}}
\end{align}
\]
具体系数暂时懒得写了，反正考虑两个等式： \(\frac{\mathrm{d}r}{\mathrm{d}\lambda}=0,\frac{\mathrm{d}^{2}r}{\mathrm{d}^{2}\lambda}=0\) 联立求解即可，对于偏心轨道，采用第一个等式分别在近远点的情况；对于用圆轨道，两个等式联立求解，详情见参考文献。
### snapshot 的绘制（采取BlackHoleToolKit）
数值计算是EMRI除运动轨迹建模外另一个难点，计算的步骤为：
+ 计算测地线方程，首先给出轨道四个方向的频率，其次给出径向与角向运动 \(r,\theta\) 表达式，最终给出 \(\Delta t\)，\(\Delta \varphi\) 。
+ 构造Teukolsky方程齐次解，获取无穷远/视界的渐近振幅
+ 构造源项表达式，计算二重卷积积分，获得振幅
+ 多模态叠加得到EMRI波形
测地线的难度主要在理论建模，数值计算上并没有太大阻碍，真正困难的在于二重积分的计算，该积分的被积函数是振荡的，由运动轨迹和 Teukolsy 方程齐次解构成，因此齐次解的计算也直接决定了二重积分的计算，例如，如果采取MST方法，那么二重积分的计算速度将远慢于采取数值直接求解Teukolsky方程（并行计算下至少慢十倍）；
计算设置：
- BlackHoleToolKit程序包实现，调用了 `Teukolsky` 、 `KerrGeodesic` 、 `SpinWeightedSpheroidalHarmonics` 。
- 轨道信息调用 
- ```mathematica
  orbit = KerrGeodesicOrbit[a,p,e,x,"Method"->"Analytical"]
	```
	即采取前面讲述过的解析法，尽管计算轨道的时候更慢，但是在给出给定轨道的各类参数时计算要更快。
- 调用 Teukolsky 包其中内置了 对于二重积分的计算，采取的是 ``` Tracy ``` 数值积分法；且集成了径向齐次方程与自旋加权椭谐函数的调用，官方程序包默认调用的齐次解为采用 Heun 方程的数值求解法： ``` Numerical  ``` ，不过我们修改了程序，调用了 ```MST``` 进行了同样的计算；注意，径向齐次解均已经对振幅进行了归一化。由此可以获得单模态振幅：
- ```mathematica
  \[Psi] = TeukolskyPointParticleMode[s, l, m, n, k, orbit]]
  Z = \[Psi]["Amplitudes"]["\[ScriptCapitalI]"]
  S = \[Psi]["AngularFunction"][\[Pi]/3, 0]
  \[Omega] = \[Psi]["\[Omega]"]
  h_{+}-ih_{\times} = 2/D Z/\[Omega]^2 S Exp[-I \[Omega] (t-r_{*})]
  ```
- 多模态叠加：对 ``` l,n,k ``` 进行截断，通常截断在当固定其他参数，目标参数连续几位的振幅都保持衰减且均低于设置阈值，其中 ``` l ``` 衰减的比较快，一般采取截断 ``` l=10,11 ``` ，对于 `n` 其是径向运动产生的指标，对于偏心率越大的轨道，需要将其截断到更高的数值，因为高偏心率导致 远点与近点相距更远，辐射强度量级差距大，二重积分容易发散，这可以追溯到径向方程的格林函数解。对于 `k` 是角向运动的指标，一般对其截断要求较低，因为角向频率较低且角向运动产生的引力波辐射并不占主导。
	本文对于低偏心率轨道（`e=0.1,0.3`）采取 `l=5,n=10,k=5`  截断，高偏心率轨道（`e=0.5`）采取 `l=5,n=15,k=5` 截断，需要注意，理论上需要计算的模态数有：	\((2l+1)(l+1)(2n+1)(2k+1)-1\) ，减 1 是因为量子数均为 0 时频率也为 0 ，没有意义。因此我们的参数配置理论上有 `7388(低偏心);10908(高偏心)` 个模态参与叠加。
-  结果展示（基于 ai agent 协助构建代码，EMRI波形的计算成本很大，即便采取并行计算以及计算速度最快的数值齐次解方法，在个人电脑上全部的16个核并行，以下结果也一共需要六小时左右，如果采取 MST 法那单个低偏心率也起码算几个小时）：
![不同偏心率与观测角下的EMRI波形](waveform1.png)
![不同偏心率与观测角下的EMRI波形](waveform3.png)
![不同偏心率与观测角下的EMRI波形](waveform5.png)
Note：目前最新进展有JH方法（`Rust`）、Heun数值计算法以及广义SN方法（`Julia`），下一步势必需要自己从BlackHoleToolKit中重构出一个专属的整合了不同齐次解解法的计算程序包。此外二重积分计算的收敛性问题也需要解决，BlackHoleToolKit程序内置的二重积分反而在计算角向振荡积分时难以达到设定收敛精度。
## 下一步
### 辐射反应——轨道衰减
##### 能流计算
##### 测地线修正
##### SFG 理论
##### Effective-One-Body 
##### Lense
##### 考虑到的几个潜在小项目：
+ 天体物理环境下的 EMRI ：
	超大质量黑洞最常见的出现位置是星系中心，而ANG是一个得到天文观测证明的天体，吸积盘的物质密度比较高，具有丰富的恒星形成与演化过程，是天然的EMRI产生场所。
	尽管越靠近克尔视界，物质密度会降低，直到越过最小稳定轨道；因此直接把吸积盘作为一个摩擦流体考虑的话其对EMRI的影响较小（尤其是对于旋近后期），但是这没有考虑引力；如果考虑吸积盘对背景时空的形变，情况或许会不一样。
	因此可以考虑一个有吸积盘形变的时空度规，以便全相对论的考虑吸积盘对EMRI的影响，更近一步的或许可以研究其对双星并和信号的影响，以此来估计目前事件发生在吸积盘上的比例。
	同理暗物质尖峰/晕。
	对于非真空轴对称 Type-D 假设下，采取 NP 形式具有度规解：
\[
\begin{align}
&\mathrm{d}s^{2}=A(r,\theta)\Bigg[
\frac{L(r)}{\Sigma}\left(\mathrm{d}t-a\sin^{2}\theta\,\mathrm{d}\varphi\right)^{2}
-\frac{\Sigma}{L(r)}\mathrm{d}r^{2}
\notag\\
&\qquad
-\Sigma\,\mathrm{d}\theta^{2}
-\frac{(r^{2}+a^{2})^{2}\sin^{2}\theta}{\Sigma}
\left(
\mathrm{d}\varphi-\frac{a}{r^{2}+a^{2}}\mathrm{d}t
\right)^{2}
\Bigg]
\end{align}
\]
对应的引力扰动的 Teukolsky 方程为：
\[
\begin{align}
&\left[\frac{(r^{2}+a^{2})^{2}}{L(r)}-a^{2}\sin^{2}\theta\right]\frac{\partial^{2}{}_{-2}\Psi}{\partial t^{2}}
+\left[\frac{2(r^{2}+a^{2})}{L(r)}\frac{\partial L(r)}{\partial r}-8r-4\mathrm{i}a\cos\theta\right]\frac{\partial {}_{-2}\Psi}{\partial t}
\notag\\
&+\left[\frac{a^{2}}{L(r)}-\frac{1}{\sin^{2}\theta}\right]\frac{\partial^{2}{}_{-2}\Psi}{\partial \varphi^{2}}
-L(r)^{2}\frac{\partial}{\partial r}\left(L(r)^{-1}\frac{\partial {}_{-2}\Psi}{\partial r}\right)
-\frac{1}{\sin\theta}\frac{\partial}{\partial\theta}\left(\sin\theta\frac{\partial {}_{-2}\Psi}{\partial \theta}\right)
\notag\\
&+\left[\frac{2aL'(r)}{L(r)}+\frac{4\mathrm{i}\cos\theta}{\sin^{2}\theta}\right]\frac{\partial {}_{-2}\Psi}{\partial\varphi}
+\left[\frac{2a(r^{2}+a^{2})}{L(r)}-2a\right]\frac{\partial^{2}{}_{-2}\Psi}{\partial t\,\partial\varphi}
\notag\\
&+(4\cot^{2}\theta+2)\,{}_{-2}\Psi
=4\pi \Sigma\,\hat{T}
\end{align}
\]
Note：该方程的导出需要进行分离规范选择，而Kerr情况的度规并不需要。
近期  `Meng Kun el.ta 2026`  研究的DM EMRI 所用的形变度规正是此形式。
评估：主要难点在于需要从度规开始全流程计算，实施成本较大（预估三个月），且物质对时空的形变较小，公式复杂，数值上对精度的需求可能比较大。
对于吸积盘情况就更为复杂，目前主要还在球对称黑洞薄盘模型，例如 `Che-Yu Chen 2023`:
\[
	\begin{align}
	 & \mathrm{d}s^{2}=-f(r)e^{2\nu_{disk}}\mathrm{d}t^{2}+e^{2\lambda_{ext}-2\nu_{disk}}\frac{\mathrm{d}r^{2}}{f(r)}+r^{2}e^{-2\nu_{disk}}(e^{2\lambda_{ext}}\mathrm{d}\theta^{2}+\sin^{2}\theta\mathrm{d}\varphi^{2})
	\end{align}
\]
不过对于球对称度规的话比较好处理，但是考虑到吸积盘结构的复杂性，预估与考虑DM的难度会差不多。
+ 有效单体描述 b-EMRI
	目前关于相对论建模的 b-EMRI 采取的 Dixon 形式来描述作为扰动源项的内双星，通过一系列坐标变换从内双星平坦时空的运动映射到背景的BL时空，以此获得可以计算的源项能动张量，但是Dixon形式描述的是具有内部结构的扩展体运动，需要假设时空渐近平坦，且需要展开到足够高阶（八极矩）以揭露内双星运动的特性。
	但是可以发现有一个共性：先描述内双星轨道运动，通过坐标变换到背景时空BL系，以此获取内双星单个天体的轨迹，问题就在于如何更好的描述内双星运动。
	近期有EOB理论可以描述具有极端质量比的双星系统的 IMR 过程，因此可以考虑使用Kerr背景下的EOB运动描述内双星，采取质心在BL系做测地运动，再通过坐标变换得到内双星天体的运动轨迹（四速度，时空坐标），以此代替点粒子能动张量中的运动量来给出单个天体的源项，而内双星的源项由两个黑洞的线性叠加即可。这样还有一个好处，可能可以用EOB来描述内双星的 IMR 过程。
	评估：四月份太极数据训练营听韩文标老师的报告，他们也在想办法去弄可以 IMR 的b-EMRI，EOB 国内有谁能做的过韩老师组啊
+ 多信使
	1. 辐射反应会导致测地偏运动，假设是带有磁场（中子星？）或者电（气体云？）的天体旋近，在辐射反应加剧的时候可能产生可以观测到的电磁信号。
	 不过看到的几篇文献都是解 爱因斯坦-麦克斯韦方程来考虑电荷的。
	2. 黑洞-中子星旋近会发出X射线，bnb-EMRI 组合或许会有一些结果?
+ 记忆效应在EMRI中的长期叠加效应
	引力波辐射中会伴随一种非振荡的长期/永久的应变，在BBH时间中或许难以观测，但是如果考虑EMRI的长期轨道演化，是否会对其相位与波形产生足以观测到的永久性偏离？
	补充：侯绍齐老师说效应太弱，对测试粒子轨迹的修正是个很小的量，基本不考虑。
