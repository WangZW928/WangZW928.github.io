---
layout: article
title: 微分几何：辛几何与哈密顿力学
mathjax: true
pageview: true
show_subscribe: true
chart: true
mermaid: true
lightbox: true
theme: dark
sharing: false
---

# 微分几何：辛几何与哈密顿力学

这篇笔记讨论一个基本问题：**为什么能量函数能够决定相空间中的运动向量场？**

答案藏在相空间的几何结构中：**辛几何**。本文分三层展开：

1. **底层语言**：外代数，处理流形上局部几何量（方向、梯度、面积）的代数系统
2. **几何结构**：辛几何，偶数维流形上的非简并闭 2-形式结构
3. **物理动力学**：哈密顿力学，辛流形上的几何流

---

## 第一部分：底层语言 —— 外代数 (Exterior Algebra)

在弯曲的相空间流形 $M$ 上，我们不能直接"平移"矢量。外代数提供了一套处理局部几何性质（方向、梯度、体积）的代数系统。

### 1.1 切空间与余切空间的对偶性

在流形上的任意一点 $x \in M$，以下两类线性空间反复出现：

**切空间 $T_xM$（Vectors）**：包含描述运动方向和速度的量。

基底为 $\left\{ \dfrac{\partial}{\partial x^\mu} \right\}$，任意矢量可展开为

$$
V = V^\mu \frac{\partial}{\partial x^\mu}.
$$

**余切空间 $T_x^*M$（1-forms）**：包含线性测量切向量的量，例如函数的微分。

基底为 $\{ dx^\mu \}$。它是切空间的对偶空间，满足

$$
dx^\mu \left( \frac{\partial}{\partial x^\nu} \right) = \delta^\mu_{\ \nu}.
$$

直观地说，矢量给出方向，1-形式给出对方向的线性测量。二者配对给出一个数，无需任何度量结构。

### 1.2 核心运算：楔积 (Wedge Product)

楔积 $\wedge$ 是构建高阶微分形式（如面积元、体积元）的工具，其核心是**反对称性**：

$$
dx^\mu \wedge dx^\nu = -\, dx^\nu \wedge dx^\mu,
\qquad
dx^\mu \wedge dx^\mu = 0.
$$

**几何含义：有向面积。**

本文采用标准的交错化约定

$$
(\alpha \wedge \beta)(U,V)=\alpha(U)\beta(V)-\alpha(V)\beta(U),
$$

也就是说，这里没有额外的 $1/2$ 因子。对于两个矢量 $U, V \in T_xM$，2-形式 $dx^\mu \wedge dx^\nu$ 测量它们在 $(x^\mu, x^\nu)$ 平面上投影构成的有向平行四边形面积：

$$
(dx^\mu \wedge dx^\nu)(U, V)
= U^\mu V^\nu - U^\nu V^\mu
= \det \begin{pmatrix} U^\mu & V^\mu \\ U^\nu & V^\nu \end{pmatrix}.
$$

交换 $U, V$ 则结果反号，这正是有向面积的含义。

### 1.3 外微分 (Exterior Derivative)

算子 $d$ 将 $k$-形式映射为 $(k+1)$-形式，是梯度概念的推广。

对标量场 $f$（0-形式），外微分给出 1-形式：

$$
df = \frac{\partial f}{\partial x^\mu}\, dx^\mu.
$$

对 1-形式 $\alpha = \alpha_\mu\, dx^\mu$：

$$
d\alpha = \frac{\partial \alpha_\mu}{\partial x^\nu}\, dx^\nu \wedge dx^\mu.
$$

**庞加莱引理**：

$$
d^2 = 0 \qquad (\text{即 } d(d\alpha) = 0).
$$

这是"梯度的旋度为零"的推广。闭形式局部可写成某个低一阶形式的外微分（局部恰当），这一点会在典范辛形式中出现。

---

## 第二部分：几何结构 —— 辛几何 (Symplectic Geometry)

辛几何是偶数维流形上的一种几何结构。不同于黎曼几何以度规刻画长度和角度，辛几何以非简并闭 2-形式刻画有向面积型配对。

### 2.1 辛形式 (Symplectic Form)

相空间 $M^{2n}$ 上的辛结构由一个 2-形式 $\omega$ 定义，需满足两个条件：

**（1）闭合性 (Closed)**：

$$
d\omega = 0.
$$

这是哈密顿流保持辛形式的关键条件。

**（2）非简并性 (Non-degenerate)**：对于任意非零矢量 $V \in T_xM$，存在 $U \in T_xM$ 使得

$$
\omega(V, U) \neq 0.
$$

这保证了 $\omega$ 诱导的线性映射 $T_xM \to T_x^*M$ 可逆（见 2.3 音乐同构）。

非简并性还强制 $\dim M$ 为偶数：反对称矩阵 $A$ 满足 $\det A = (-1)^{\dim M} \det A$，奇数维时行列式必为零。

### 2.2 达布定理 (Darboux's Theorem)

Darboux 定理说明，辛形式在局部没有类似黎曼曲率的局部不变量。对任意辛流形 $(M,\omega)$ 和任意点 $x \in M$，存在该点附近的局部坐标 $(q^1, \ldots, q^n, p_1, \ldots, p_n)$，使得辛形式呈现标准形式：

$$
\boxed{\;\omega = \sum_{i=1}^n dq^i \wedge dp_i\;}
$$

在此基底下 $\omega$ 的矩阵表示为

$$
\omega_{\mu\nu} =
\begin{pmatrix}
0 & I_n \\
-I_n & 0
\end{pmatrix} \equiv J.
$$

这解释了哈密顿力学中 $(q,p)$ 正则坐标的几何来源：它们是辛结构允许的局部标准坐标。所有同维数辛流形在局部辛同胚；辛几何的非平凡性主要体现在整体结构、子流形和动力学等方面。

**最重要的例子：余切丛 $T^*Q$。** 给定位形空间 $Q$（坐标 $q^i$），其余切丛上有典范 1-形式（Liouville 形式）

$$
\theta = p_i\, dq^i,
$$

辛形式定义为

$$
\omega = -d\theta = dq^i \wedge dp_i.
$$

$\omega$ 自动闭（因为 $d^2=0$），并且在正则坐标中非简并。位形空间 $Q$ 一旦给定，余切丛相空间 $T^*Q$ 上就有典范辛结构。

### 2.3 音乐同构：矢量与梯度的桥梁

由于 $\omega$ 非简并，它定义了一个映射（降指标）

$$
\flat: TM \to T^*M, \qquad V \mapsto \omega(V, \cdot)
$$

及其逆映射（升指标）

$$
\sharp: T^*M \to TM.
$$

**物理意义**：

在黎曼几何中，度规 $g_{\mu\nu}$ 将矢量变为余矢量。在辛几何中，$\omega_{\mu\nu}$ 也给出 $TM$ 与 $T^*M$ 之间的同构；不同之处在于 $\omega$ 是反对称的，因此对应的动力学方向与能量微分之间不是梯度流关系。

$$
\text{给定能量微分 } dH
\;\xrightarrow{\;\sharp\;}\;
\text{产生唯一的哈密顿矢量场 } X_H.
$$

这就是哈密顿力学中能量函数可以决定运动向量场的几何原因：辛形式把能量微分 $dH$ 唯一对应到向量场 $X_H$。

对比：黎曼度规给出的梯度流 $\dot{x} = -\nabla f$ 使 $f$ 单调下降（耗散系统）；辛形式给出的哈密顿流则让 $H$ 严格守恒（保守系统）。差别正来自 $g$ 对称、$\omega$ 反对称。

---

## 第三部分：物理动力学 —— 哈密顿力学

哈密顿形式的经典力学可以表述为辛流形上的几何流。

### 3.1 几何化的运动方程

哈密顿向量场由下式定义：

$$
\boxed{\; \iota_{X_H}\, \omega = dH \;}
\qquad
(\text{本文固定采用这一符号约定})
$$

这里 $\iota$ 是内积算子（interior product / contraction）。对 $k$-形式 $\alpha$，

$$
(\iota_X\alpha)(Y_1,\ldots,Y_{k-1})
=\alpha(X,Y_1,\ldots,Y_{k-1}).
$$

特别地，对 2-形式 $\omega$，$\iota_X\omega=\omega(X,\cdot)$。上式的含义是：寻找一个向量场 $X_H$，使得它插入辛形式第一个槽位后得到 $dH$。有些教材采用 $\iota_{X_H}\omega=-dH$；该选择会同时改变正则方程和泊松括号的相关符号。下文始终使用 $\iota_{X_H}\omega=dH$。

### 3.2 详细推导：从几何到正则方程

让我们在 1 自由度下（$M = \mathbb{R}^2$，$\omega = dq \wedge dp$）完整推导。

**步骤 A：写出哈密顿矢量场的一般形式**

设系统演化速度为

$$
X_H = \dot{q}\, \frac{\partial}{\partial q} + \dot{p}\, \frac{\partial}{\partial p}.
$$

**步骤 B：计算内积 $\iota_{X_H}\omega$**

设 $Y=a\,\partial_q+b\,\partial_p$。由 1.2 的楔积求值约定，

$$
(dq\wedge dp)(X_H,Y)
=dq(X_H)dp(Y)-dq(Y)dp(X_H).
$$

逐项代入

$$
dq(X_H)=\dot q,\qquad dp(X_H)=\dot p,\qquad dq(Y)=a,\qquad dp(Y)=b,
$$

得到

$$
(dq\wedge dp)(X_H,Y)=\dot q\,b-\dot p\,a.
$$

因此作为 1-形式，

$$
\begin{aligned}
\iota_{X_H}(dq \wedge dp)
&= dq(X_H)\, dp - dp(X_H)\, dq \\
&= \dot{q}\, dp - \dot{p}\, dq.
\end{aligned}
$$

**步骤 C：写出能量微分**

$$
dH = \frac{\partial H}{\partial q}\, dq + \frac{\partial H}{\partial p}\, dp.
$$

**步骤 D：比较系数**

令 $\iota_{X_H}\omega = dH$，对比 $dq$、$dp$ 前的系数：

$$
dp \text{ 的系数}: \quad \dot{q} = \frac{\partial H}{\partial p},
$$

$$
dq \text{ 的系数}: \quad -\dot{p} = \frac{\partial H}{\partial q}
\;\;\Longrightarrow\;\;
\dot{p} = -\frac{\partial H}{\partial q}.
$$

$$
\boxed{\;\dot{q} = \frac{\partial H}{\partial p}, \qquad \dot{p} = -\frac{\partial H}{\partial q}\;}
$$

这就得到哈密顿正则方程。第二个方程中的负号来自辛形式的反对称性以及本文把 $X_H$ 插入 $\omega$ 第一个槽位的约定。

### 3.3 守恒定律的几何证明

**（1）能量守恒**

$$
\frac{dH}{dt} = X_H(H) = dH(X_H) = \omega(X_H, X_H) = 0.
$$

其中第三个等号使用了本文约定 $\iota_{X_H}\omega=dH$，即 $dH(Y)=\omega(X_H,Y)$；取 $Y=X_H$ 便得 $dH(X_H)=\omega(X_H,X_H)$。最后一项为零只依赖于辛形式的反对称性：$\omega(V,V)=0$。因此哈密顿流线位于 $H$ 的等值集上（在正则点附近即等能面）。

**（2）相体积守恒（Liouville 定理）**

利用 Cartan 公式：

$$
\mathcal{L}_{X}\, \omega = \iota_X (d\omega) + d(\iota_X \omega),
$$

对 $X = X_H$：

$$
\mathcal{L}_{X_H}\, \omega
= \underbrace{\iota_{X_H}(d\omega)}_{=\,0\ (\text{闭合性})}
+ \underbrace{d(\iota_{X_H}\omega)}_{=\,d(dH)\,=\,0\ (\text{庞加莱引理})}
= 0.
$$

因此 $\mathcal{L}_{X_H}\omega=0$。若 $\phi_t$ 是 $X_H$ 的局部流，则

$$
\frac{d}{dt}\phi_t^*\omega
=\phi_t^*(\mathcal{L}_{X_H}\omega)=0.
$$

由于 $\phi_0^*\omega=\omega$，得到

$$
\phi_t^*\omega=\omega.
$$

相体积形式定义为

$$
\Omega = \frac{1}{n!}\, \omega^{\wedge n} = dq^1 \wedge dp_1 \wedge \cdots \wedge dq^n \wedge dp_n
$$

其中等号右侧对应 $\omega=\sum_i dq^i\wedge dp_i$；展开 $\omega^{\wedge n}$ 时，只有每一对 $dq^i\wedge dp_i$ 各出现一次的项保留下来，共有 $n!$ 个相同符号的项，因此需要系数 $1/n!$。由 pullback 与楔积相容，

$$
\phi_t^*\Omega
=\frac{1}{n!}(\phi_t^*\omega)^{\wedge n}
=\frac{1}{n!}\omega^{\wedge n}
=\Omega.
$$

这就是 Liouville 定理的几何证明：哈密顿流保持辛体积形式不变。注意 $d\omega=0$ 在证明 $\mathcal{L}_{X_H}\omega=0$ 时是关键条件。

### 3.4 泊松括号 (Poisson Bracket)

物理量 $f, g$ 之间的代数关系由泊松括号描述，其几何定义为：

$$
\boxed{\; \{f, g\} = \omega(X_f, X_g) \;}
$$

也就是说，两个函数的泊松括号等于它们对应的哈密顿向量场在辛形式下的配对。这里的 $X_f$ 同样由 $\iota_{X_f}\omega=df$ 定义。

在正则坐标下展开，就回到熟悉的形式：

$$
\{f, g\} = \sum_{i=1}^n \left(
\frac{\partial f}{\partial q^i}\frac{\partial g}{\partial p_i}
- \frac{\partial f}{\partial p_i}\frac{\partial g}{\partial q^i}
\right).
$$

任意物理量沿哈密顿流的时间演化为：

$$
\frac{df}{dt} = \{f, H\}.
$$

更准确地说，映射 $f \mapsto X_f$ 是 Lie 代数反同态：

$$
[X_f, X_g] = -X_{\{f, g\}}.
$$

若改用相反的哈密顿向量场约定（$\iota_{X_f}\omega=-df$），则该映射成为同态。函数空间本身在 $\{\cdot,\cdot\}$ 下构成 Lie 代数，并与普通乘法一起构成 Poisson 代数。

Jacobi 恒等式的几何来源正是 $d\omega=0$。对任意 $f,g,h$，闭合性给出

$$
0=d\omega(X_f,X_g,X_h).
$$

把外微分展开，并利用 $\mathcal{L}_{X_f}\omega=0$（等价于哈密顿流保持 $\omega$）和 $\{f,g\}=\omega(X_f,X_g)$，可化为

$$
\{f,\{g,h\}\}+\{g,\{h,f\}\}+\{h,\{f,g\}\}=0.
$$

Leibniz 法则则来自 $X_f$ 作为向量场对函数乘法满足通常的导子性质。

这正是通往量子力学的桥梁：正则量子化的规则

$$
\{f, g\} \;\longmapsto\; \frac{1}{i\hbar}\,[\hat{f}, \hat{g}]
$$

就是把这个 Poisson 代数形变为 Hilbert 空间上的算符对易子代数。

---

## 总结：三层结构

| 层级 | 数学对象 | 物理对应 |
|------|----------|----------|
| 外代数 | $dx^\mu$、$\wedge$、$d$ | 局部测量：梯度、有向面积 |
| 辛形式 | $\omega = dq^i \wedge dp_i$，$d\omega = 0$ | 相空间的内禀结构 |
| 音乐同构 | $\sharp: dH \mapsto X_H$ | 能量函数确定运动向量场 |
| 哈密顿流 | $\iota_{X_H}\omega = dH$ | 正则方程 |
| $\omega$ 反对称 | $\omega(X, X) = 0$ | 能量守恒 |
| $\omega$ 闭合 | $\mathcal{L}_{X_H}\omega = 0$ | Liouville 定理 |
| Poisson 括号 | $\{f,g\} = \omega(X_f, X_g)$ | 可观测量代数与量子化 |

**核心结论**：哈密顿力学可以看作辛几何的自然动力学表述。辛形式 $\omega$ 给出正则坐标的局部标准形（Darboux 定理）、能量函数到运动向量场的对应（音乐同构）、以及哈密顿流保持相体积的机制（闭合性与反对称性）。改变 $H$ 会改变具体的哈密顿流；固定 $\omega$ 则固定了相空间上的辛几何结构。

---

## 参考文献

- Arnold, V.I. *Mathematical Methods of Classical Mechanics*. Springer, 1989.
- da Silva, A.C. *Lectures on Symplectic Geometry*. Springer, 2001.
- Marsden, J.E. & Ratiu, T.S. *Introduction to Mechanics and Symmetry*. Springer, 1999.
- Nakahara, M. *Geometry, Topology and Physics*. 2nd ed., IOP, 2003.
