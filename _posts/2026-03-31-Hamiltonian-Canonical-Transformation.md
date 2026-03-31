---
layout: article
title: Hamiltonian Canonical Transformations
mathjax: true
pageview: true
show_subscribe: true
chart: true
mermaid: true
lightbox: true
theme: dark
sharing: false
---

# Hamilton 正则变换笔记

这篇笔记想回答一个问题：**什么样的变量变换，不会改变 Hamilton 本身的动力学结构？**  
答案就是：**正则变换**。  
它可以从变分原理来理解，也可以从辛几何来理解，还可以从 Poisson 括号代数来理解。这几种说法其实是同一件事的不同侧面。

---

## 1. 从作用量原理出发推导拉格朗日方程

设系统的拉格朗日量为

$$
L(q,\dot q,t),
$$

作用量定义为

$$
S[q] = \int_{t_1}^{t_2} L(q,\dot q,t)\,dt.
$$

在固定端点条件

$$
\delta q(t_1)=\delta q(t_2)=0
$$

下，最小作用量原理要求

$$
\delta S=0.
$$

对轨道作无穷小变分 $q(t)\mapsto q(t)+\delta q(t)$，则

$$\delta S= \int_{t_1}^{t_2}
\left(
\frac{\partial L}{\partial q}\delta q
+\frac{\partial L}{\partial \dot q}\delta \dot q
\right)dt.
$$

对第二项分部积分，

$$
\int_{t_1}^{t_2}\frac{\partial L}{\partial \dot q}\delta \dot q\,dt
= \left[
\frac{\partial L}{\partial \dot q}\delta q
\right]_{t_1}^{t_2}
-\int_{t_1}^{t_2}\frac{d}{dt}\left(\frac{\partial L}{\partial \dot q}\right)\delta q\,dt.
$$

由于端点固定，边界项为零，因此

$$
\delta S
= \int_{t_1}^{t_2}
\left[
\frac{\partial L}{\partial q}
-\frac{d}{dt}\left(\frac{\partial L}{\partial \dot q}\right)
\right]\delta q\,dt.
$$

变分 $\delta q$ 在区间内部任意，所以必有

$$
\boxed{
\frac{d}{dt}\left(\frac{\partial L}{\partial \dot q^i}\right)
-\frac{\partial L}{\partial q^i}=0
}
$$

这就是 Euler-Lagrange 方程。

---

## 2. 从对称性的角度理解广义动量

定义广义动量

$$
\boxed{
p_i := \frac{\partial L}{\partial \dot q^i}
}
$$

这一定义看起来像是“对速度求导”，但它的结构意义更深。  
若某个连续参数 $\epsilon$ 生成配置空间中的变换

$$
q^i \mapsto q^i + \epsilon X^i(q),
$$

其中 $X=X^i\partial_{q^i}$ 是一个无穷小生成向量场，那么对应的速度变化为

$$
\delta \dot q^i = \epsilon \frac{dX^i}{dt}.
$$

Noether 分析告诉我们：如果拉格朗日量在这个连续变换下不变，或者只差一个全导数，那么存在守恒量。最基本的情形下，这个守恒量就是

$$
J_X = p_i X^i.
$$

这说明：

- 广义动量 $p_i$ 是与坐标方向 $q^i$ 配对的“共轭量”
- 更一般地，连续变换方向 $X$ 的生成元是 $p_i X^i$
- 若某个坐标 $q^i$ 是循环坐标，即 $\partial L/\partial q^i=0$，则

$$
\dot p_i = 0
$$

也就是说，该方向上的广义动量守恒

$$
p_i = \text{const.}
$$

所以“广义动量是什么”可以这样理解：  
**它不是单纯的机械动量，而是连续对称方向在相空间中的生成元系数。**

---

## 3. 从拉格朗日形式到 Hamilton 方程，再到辛形式

若拉格朗日量关于速度是非退化的，即 Hessian

$$
\det\left(\frac{\partial^2 L}{\partial \dot q^i \partial \dot q^j}\right)\neq 0,
$$

则可以通过 Legendre 变换把 $(q,\dot q)$ 改写成 $(q,p)$。定义 Hamilton 量

$$
\boxed{
H(q,p,t)=p_i\dot q^i - L(q,\dot q,t)
}
$$

其中 $\dot q$ 要理解为由 $p_i=\partial L/\partial \dot q^i$ 反解得到的函数 $\dot q(q,p,t)$。

对 $H$ 求全微分：

$$ \begin{aligned}
dH &= \dot q^i dp_i + p_i d\dot q^i - \frac{\partial L}{\partial q^i}dq^i - \frac{\partial L}{\partial \dot q^i}d\dot q^i - \frac{\partial L}{\partial t}dt \\
&= \dot q^i dp_i - \frac{\partial L}{\partial q^i}dq^i - \frac{\partial L}{\partial t}dt
\end{aligned}
$$

利用 Euler-Lagrange 方程

$$
\frac{\partial L}{\partial q^i} = \dot p_i,
$$

得到

$$
dH = \dot q^i dp_i - \dot p_i dq^i - \frac{\partial L}{\partial t}dt.
$$

另一方面，

$$
dH = \frac{\partial H}{\partial q^i}dq^i + \frac{\partial H}{\partial p_i}dp_i + \frac{\partial H}{\partial t}dt.
$$

比较系数可得 Hamilton 方程

$$
\boxed{
\dot q^i = \frac{\partial H}{\partial p_i},\qquad
\dot p_i = -\frac{\partial H}{\partial q^i}
}
$$

把它写成矩阵形式更紧凑。设

$$
z =
\begin{pmatrix}
q \\
p
\end{pmatrix},
\qquad
J=
\begin{pmatrix}
0 & I \\
-I & 0
\end{pmatrix}.
$$

则 Hamilton 方程可写成

$$
\boxed{
\dot z = J \nabla H
}
$$

这就是 Hamilton 方程的辛形式。

与之对应的辛 2-形式是

$$
\boxed{
\omega = \sum_i dq^i \wedge dp_i
}
$$

它编码了相空间的基本几何结构，而 Hamilton 流恰好保持这个结构不变。

---

## 4. 正则变量与正则变换：从 Hamilton 作用量理解

Hamilton 形式下的作用量通常写为

$$
\boxed{
S[q,p] = \int_{t_1}^{t_2}\left(p_i\dot q^i - H(q,p,t)\right)dt
}
$$

其中被积函数

$$
p_i\dot q^i - H
$$

是一阶形式的动力学拉格朗日量。

现在考虑相空间变量变换

$$
(q,p) \mapsto (Q,P).
$$

如果变换后作用量仍保持同样的 Hamilton 结构，即

$$
\boxed{
p_i dq^i - H\,dt
=P_i dQ^i - K\,dt + dF
}
$$

其中 $F$ 是某个函数，$dF$ 只会带来边界项，不改变变分方程，那么新变量 $(Q,P)$ 仍满足标准 Hamilton 方程

$$
\dot Q^i = \frac{\partial K}{\partial P_i},\qquad
\dot P_i = -\frac{\partial K}{\partial Q^i}.
$$

因此：

$$
\boxed{
\text{正则变换就是保持 Hamilton 变分原理形式不变的变换}
}
$$

这也是“正则变量”的含义：  
它们是把动力学写成标准一阶 Hamilton 形式的一组变量。

更具体地说，若只关心时间无关的变换，则正则变换等价于

$$
p_i dq^i - P_i dQ^i = dF.
$$

对两边再取外微分，立刻得到

$$
dp_i \wedge dq^i = dP_i \wedge dQ^i.
$$

这就把变分意义上的“正则”自然引向了辛几何意义上的“正则”。

---

## 5. 从微分几何角度看：正则变换就是保辛变换

相空间可以看成一个辛流形 $(M,\omega)$，其中

$$
\omega = dq^i \wedge dp_i.
$$

一个微分同胚

$$
\Phi : M \to M
$$

若满足

$$
\boxed{
\Phi^\ast \omega = \omega
}
$$

则称 $\Phi$ 为保辛变换，或者辛同胚。

在标准正则坐标中，这正是正则变换。

所以：

$$
\boxed{
\text{正则变换} \iff \text{保持辛结构不变}
}
$$

这句话的含义非常重要。它说明正则变换并不要求“坐标长得差不多”，也不要求变换必须线性；它真正保持的是：

- 相空间中的面积元推广
- Hamilton 方程的结构
- 可观测量之间的 Poisson 代数

如果写成 Jacobian 矩阵 $M=\partial(Q,P)/\partial(q,p)$，则正则变换满足

$$
\boxed{
M^T J M = J
}
$$

这就是有限维线性情形中的辛条件。

---

## 6. Poisson 括号结构不变与正则变换等价

对任意两个相空间函数 $f(q,p),g(q,p)$，Poisson 括号定义为

$$
\boxed{
\{f,g\}
= \sum_i
\left(
\frac{\partial f}{\partial q^i}\frac{\partial g}{\partial p_i}
-\frac{\partial f}{\partial p_i}\frac{\partial g}{\partial q^i}
\right)
}
$$

在标准正则坐标里，

$$
\{q^i,q^j\}=0,\qquad
\{p_i,p_j\}=0,\qquad
\{q^i,p_j\}=\delta^i_j.
$$

若变量变换 $(q,p)\mapsto(Q,P)$ 满足

$$
\boxed{
\{Q^i,Q^j\}=0,\qquad
\{P_i,P_j\}=0,\qquad
\{Q^i,P_j\}=\delta^i_j
}
$$

则新变量仍是正则变量，因此该变换是正则变换。

更强地说，对任意可观测量 $f,g$ 都有

$$
\boxed{
\{f,g\}_{q,p} = \{f,g\}_{Q,P}
}
$$

这与保持辛形式是等价的。

因此还可以说：

$$
\boxed{
\text{正则变换} \iff \text{Poisson 括号结构不变}
}
$$

从“可观测量代数”的角度看，Poisson 括号给出了经典力学的李代数结构。  
正则变换不只是“换了变量”，而是**保持可观测量之间的代数关系不变**。  
所以它是动力学结构的同构，而不是普通坐标重命名。

---

## 7. 几个例子：非线性变换也完全可以是正则的

下面用“辛形式是否保持不变”来判断。

### 例 1：平移是正则变换

设

$$
Q=q,\qquad P=p+c
$$

其中 $c$ 是常数，则

$$
dQ\wedge dP = dq\wedge dp.
$$

所以这是正则变换。

---

### 例 2：缩放变换可以是正则的

设

$$
Q=\lambda q,\qquad P=\frac{p}{\lambda},\qquad \lambda\neq 0.
$$

则

$$
dQ\wedge dP = \lambda\,dq \wedge \frac{1}{\lambda}dp = dq\wedge dp.
$$

所以这也是正则变换。

这里很清楚地看到：  
**坐标怎么变，动量不能随便变，它必须用互补方式联动，才能保持辛结构。**

---

### 例 3：只改坐标、不改动量，通常不是正则变换

设

$$
Q=2q,\qquad P=p.
$$

则

$$
dQ\wedge dP = 2\,dq\wedge dp \neq dq\wedge dp.
$$

因此它不是正则变换。

这说明：  
“变量替换”不等于“正则变换”。

---

### 例 4：非线性变换也可以是正则的

设一维情形

$$
Q=q^3,\qquad P=\frac{p}{3q^2}
$$

定义在 $q\neq 0$ 的区域。

计算：

$$
dQ = 3q^2\,dq,
$$

而

$$
dP = \frac{1}{3q^2}dp - \frac{2p}{3q^3}dq.
$$

于是

$$
dQ\wedge dP
=3q^2 dq \wedge
\left(
\frac{1}{3q^2}dp - \frac{2p}{3q^3}dq
\right)
= dq\wedge dp.
$$

因为 $dq\wedge dq=0$，所以这是一个正则变换。

这正是一个很好的例子：  
**正则变换完全可以是非线性的，但动量的变换必须精确补偿坐标变换带来的 Jacobian。**

---

### 例 5：点变换诱导的正则变换

若配置空间中做点变换

$$
Q^i = Q^i(q),
$$

那么对应的正则动量必须取为

$$
\boxed{
P_i = \sum_j p_j \frac{\partial q^j}{\partial Q^i}
}
$$

这样才有

$$
p_i dq^i = P_i dQ^i.
$$

因此任意可逆点变换都能提升为相空间中的正则变换，但前提是动量按照余切向量的方式一起变。

---

## 8. 正则变换与 Noether 守恒量：守恒量就是生成元

在 Hamilton 力学中，任意相空间函数 $G(q,p,t)$ 都可以生成一个无穷小变换：

$$
\boxed{
\delta q^i = \epsilon \{q^i,G\} = \epsilon \frac{\partial G}{\partial p_i},
\qquad
\delta p_i = \epsilon \{p_i,G\} = -\epsilon \frac{\partial G}{\partial q^i}
}
$$

这正是一个无穷小正则变换。

如果 $G$ 还是守恒量，即

$$
\frac{dG}{dt}
= \frac{\partial G}{\partial t}+\{G,H\}=0,
$$

那么由它生成的正则变换与系统动力学相容，它对应的正是一个连续对称性。

这就是 Hamilton 形式下 Noether 定理的语言：

$$
\boxed{
\text{守恒量} \longleftrightarrow \text{连续对称性的生成元}
}
$$

例如：

- 空间平移的生成元是线动量
- 空间旋转的生成元是角动量
- 时间平移对应 Hamilton 量本身

所以“守恒量为什么重要”还有另一层意思：  
它不仅在演化中保持不变，而且还能主动生成一族正则变换。

---

## 9. 为什么无穷小正则变换由 Poisson 括号生成？

本节想解释：

$$
\boxed{
\text{为什么任意无穷小正则变换都可以写成： } \delta f=\epsilon\{f,G\}\ ?
}
$$

也就是说，为什么一定存在某个函数 $G(q,p,t)$，使得

$$
\delta q^i = \epsilon \frac{\partial G}{\partial p_i},
\qquad
\delta p_i = -\epsilon \frac{\partial G}{\partial q^i}.
$$

一旦这两个式子成立，那么对任意相空间函数 $f(q,p)$，链式法则立刻给出

$$
\begin{aligned}
\delta f
&=
\sum_i \frac{\partial f}{\partial q^i}\delta q^i
+
\sum_i \frac{\partial f}{\partial p_i}\delta p_i \\
&=
\epsilon \sum_i
\left(
\frac{\partial f}{\partial q^i}\frac{\partial G}{\partial p_i}
-\frac{\partial f}{\partial p_i}\frac{\partial G}{\partial q^i}
\right) \\
&= \epsilon \{f,G\}.
\end{aligned}
$$


下面用三种互相呼应的方式来说明。

### 9.1 从“保持辛形式”直接推出

先不要假设它由某个函数生成，而是写成最一般的无穷小形式

$$
Q^i = q^i + \epsilon \xi^i(q,p),
\qquad
P_i = p_i + \epsilon \eta_i(q,p).
$$

这里 $\xi^i,\eta_i$ 是待定函数。

因为是正则变换，所以它必须保持辛形式

$$
\omega = \sum_i dq^i \wedge dp_i
$$

不变，也就是

$$
\sum_i dQ^i \wedge dP_i
=\sum_i dq^i \wedge dp_i
\qquad.
$$

展开到一阶：

$$
dQ^i = dq^i + \epsilon\, d\xi^i,
\qquad
dP_i = dp_i + \epsilon\, d\eta_i.
$$

因此

$$
dQ^i \wedge dP_i=
dq^i \wedge dp_i
+
\epsilon
\left(
d\xi^i \wedge dp_i
+
dq^i \wedge d\eta_i
\right)
+
O(\epsilon^2).
$$

对所有 $i$ 求和，正则性要求一阶项为零：

$$
\sum_i
\left(
d\xi^i \wedge dp_i
+
dq^i \wedge d\eta_i
\right)
=0.
$$

把它改写一下：

$$
d\left(
\sum_i \xi^i\, dp_i - \eta_i\, dq^i
\right)=0.
$$

所以 1-形式

$$
\alpha := \sum_i \xi^i\, dp_i - \eta_i\, dq^i
$$

是闭的。局部上闭形式就是恰当形式，因此存在某个函数 $G$，使得

$$
\boxed{
\sum_i \xi^i\, dp_i - \eta_i\, dq^i = dG
}
$$

比较 $dq^i$ 与 $dp_i$ 的系数，就得到

$$
\xi^i = \frac{\partial G}{\partial p_i},
\qquad
\eta_i = -\frac{\partial G}{\partial q^i}.
$$

于是

$$
\boxed{
\delta q^i = \epsilon \xi^i = \epsilon \frac{\partial G}{\partial p_i},
\qquad
\delta p_i = \epsilon \eta_i = -\epsilon \frac{\partial G}{\partial q^i}
}
$$

再用链式法则，就得到

$$
\boxed{
\delta f = \epsilon \{f,G\}
}
$$

这就是最直接的推导。

### 9.2 用微分几何语言重述


设无穷小变换对应的向量场为

$$
X=
\sum_i
\left(
\xi^i \frac{\partial}{\partial q^i}
+
\eta_i \frac{\partial}{\partial p_i}
\right).
$$

由于每个 $\epsilon$ 都保持辛结构，因此这个无穷小流必须满足

$$
\mathcal{L}_X \omega = 0.
$$

利用 Cartan 公式

$$
\mathcal{L}_X \omega = d(i_X\omega) + i_X(d\omega),
$$

而辛形式满足 $d\omega=0$，故

$$
d(i_X\omega)=0.
$$

所以局部上存在函数 $G$，使得

$$
\boxed{
i_X\omega = dG
}
$$

这说明任意无穷小保辛变换在局部上都是某个函数 $G$ 的 Hamilton 向量场。

在标准坐标中，

$$
X_G
= \frac{\partial G}{\partial p_i}\frac{\partial}{\partial q^i}
-\frac{\partial G}{\partial q^i}\frac{\partial}{\partial p_i}.
$$

因此

$$
\delta q^i = \epsilon X_G(q^i) = \epsilon \frac{\partial G}{\partial p_i},
\qquad
\delta p_i = \epsilon X_G(p_i) = -\epsilon \frac{\partial G}{\partial q^i}.
$$

更一般地，对任意可观测量 $f$，

$$
\boxed{
\delta f = \epsilon X_G(f) = \epsilon \{f,G\}
}
$$

这说明 Poisson 括号本质上就是 Hamilton 向量场作用在函数上的坐标表达。

### 9.3 从生成函数角度直接推出

对无穷小正则变换，最方便使用第二类生成函数：

$$
F_2(q,P,t)=
\sum_i q^i P_i + \epsilon G(q,P,t).
$$

第一项对应恒等变换，$\epsilon G$ 表示在恒等变换附近做一个小扰动。

第二类生成函数对应的正则变换公式为

$$
p_i = \frac{\partial F_2}{\partial q^i},
\qquad
Q^i = \frac{\partial F_2}{\partial P_i}.
$$

代入上式：

$$
p_i = P_i + \epsilon \frac{\partial G}{\partial q^i},
\qquad
Q^i = q^i + \epsilon \frac{\partial G}{\partial P_i}.
$$

在一阶近似下，$P_i$ 和 $p_i$ 的差本身已经是 $O(\epsilon)$，所以可以把$\frac{\partial G}{\partial P_i}$替换成$
\frac{\partial G}{\partial p_i}.
$

于是

$$
\boxed{
Q^i = q^i + \epsilon \frac{\partial G}{\partial p_i},
\qquad
P_i = p_i - \epsilon \frac{\partial G}{\partial q^i}
}
$$

即

$$
\delta q^i = \epsilon \frac{\partial G}{\partial p_i},
\qquad
\delta p_i = -\epsilon \frac{\partial G}{\partial q^i}.
$$

所以从生成函数角度看，这个公式就是无穷小正则变换的一阶展开。

### 9.4 与 Hamilton 方程的类比：

因为它和 Hamilton 方程完全同型。

Hamilton 方程是

$$
\dot q^i = \frac{\partial H}{\partial p_i} = \{q^i,H\},
\qquad
\dot p_i = -\frac{\partial H}{\partial q^i} = \{p_i,H\}.
$$

因此任意不显含时的函数 $f(q,p)$ 的演化为

$$
\frac{df}{dt} = \{f,H\}.
$$

这说明时间推进本身就是一个由 $H$ 生成的无穷小正则变换。在小时间 $dt$ 内，

$$
\delta f = dt\,\{f,H\}.
$$

如果不是沿“时间方向”推进，而是沿某个抽象参数 $\epsilon$ 方向推进，那么自然就写成

$$
\frac{df}{d\epsilon} = \{f,G\}.
$$

于是无穷小形式就是

$$
\boxed{
\delta f = \epsilon \{f,G\}
}
$$

在这个意义下：

- $H$ 生成时间平移
- 线动量 $p$ 生成空间平移
- 角动量 $L_z$ 生成旋转
- 一般的 $G$ 生成某种连续正则变换

所以这条公式本质上是 Hamilton 方程从“时间演化”推广到“一般连续相空间流”的统一表达。

---

## 总结

这篇笔记的主线可以浓缩为下面几句话：

1. 最小作用量原理给出 Euler-Lagrange 方程。
2. 通过 Legendre 变换，动力学进入 Hamilton 相空间形式。
3. Hamilton 方程的核心结构由辛形式 $\omega=dq^i\wedge dp_i$ 编码。
4. 正则变换就是保持 Hamilton 作用量形式不变的变换。
5. 等价地说，正则变换就是保辛变换。
6. 再等价地说，正则变换就是保持 Poisson 括号代数不变的变换。
7. 守恒量既是 Noether 对称性的结果，也是无穷小正则变换的生成元。

所以，正则变换最本质的意义并不是“换一组更方便的变量”，而是：

$$
\boxed{
\text{它保持了经典动力学的结构本身}
}
$$

这也正是 Hamilton 力学在几何上如此优美的原因。
