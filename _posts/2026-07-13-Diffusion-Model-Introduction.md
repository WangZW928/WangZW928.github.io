---
layout: article
title: Diffusion Model：DDPM 正向与反向过程的数学推导
mathjax: true
pageview: true
---

DDPM（Denoising Diffusion Probabilistic Models）的核心思想是：先定义一个固定的正向加噪过程，把真实数据逐步变成接近标准高斯噪声的变量；再训练一个神经网络近似反向去噪过程，从噪声逐步生成样本。

本文用 DDPM 的常见记号推导以下内容：

1. 为什么正向过程是马尔可夫链；
2. 单步加噪公式如何得到任意步长的闭式表达；
3. 为什么终点可以近似为标准高斯噪声；
4. 训练目标为什么通常写成预测噪声；
5. 反向采样公式如何由贝叶斯后验推导出来。

---

## 0. 图片、随机变量与概率分布

一张图像可以看成一个高维向量 $x \in \mathbb{R}^d$，其中 $d = H \times W \times C$。单张图片是这个空间中的一个点，真实图片数据集则来自某个未知分布 $q(x_0)$。

生成模型的目标不是记住某一张图片，而是学习这个数据分布。训练完成后，模型可以从一个简单分布（DDPM 中通常是标准高斯分布）出发，逐步生成落在高概率区域的新样本。

在扩散模型中，$x_t$ 表示第 $t$ 步的整张图像，而不是某一个像素。公式中的乘法和加法都按张量逐元素执行。对于第 $i$ 个维度，有：

$$q(x_t^i \vert{} x_{t-1}^i) = \mathcal{N}(x_t^i; \sqrt{1 - \beta_t}\,x_{t-1}^i, \beta_t)$$

由于每一步都会加入随机噪声，给定 $x_{t-1}$ 后，$x_t$ 不是确定值，而是一个随机变量。一次代码运行得到的是样本；概率分布描述的是这些样本在重复实验中的统计规律。

---

## 1. 为什么选择高斯噪声？

DDPM 使用高斯噪声主要有三个原因。

**第一，高斯分布对线性组合封闭。** 如果 $\epsilon_1,\epsilon_2$ 是独立标准高斯噪声，那么 $a\epsilon_1 + b\epsilon_2$ 仍然是高斯噪声，方差为 $a^2 + b^2$。这使得多步加噪可以合并成一个闭式公式：

$$x_t = \sqrt{\bar{\alpha}_t}\,x_0 + \sqrt{1 - \bar{\alpha}_t}\,\epsilon$$

训练时因此不需要真的循环执行 $t$ 次加噪。

**第二，高斯分布是方便的终点分布。** 标准高斯 $\mathcal{N}(0,\mathbf{I})$ 易采样、各维独立、尺度固定，适合作为反向生成的起点。

**第三，高斯分布有解析后验。** 在给定 $x_0$ 的条件下，$q(x_{t-1}\vert{}x_t,x_0)$ 仍然是高斯分布，可以通过配平方得到闭式均值和方差。这是 DDPM 训练和采样公式的基础。

---

## 1.5. SGLD 与扩散模型的关系

在理解 DDPM 的反向过程之前，可以先看一个更一般的采样方法：Stochastic Gradient Langevin Dynamics（SGLD）。它来自 Langevin dynamics，连续时间形式可以写成：

$$dx = \nabla_x \log p(x)\,dt + \sqrt{2}\,dW$$

其中 $W$ 是标准 Wiener process，$dW$ 表示随机布朗运动的增量。第一项 $\nabla_x \log p(x)$ 称为 score function，它指向概率密度上升最快的方向；第二项 $\sqrt{2}\,dW$ 注入随机噪声，使采样不会只停在某个局部高概率点，而是能够覆盖整个目标分布。

把上式离散化，得到类似下面的更新：

$$x_{k+1} = x_k + \eta \nabla_x \log p(x_k) + \sqrt{2\eta}\,z_k,\qquad z_k\sim\mathcal{N}(0,\mathbf{I})$$

这就是 Langevin sampling 的基本形式。SGLD 的思想是：如果无法直接从 $p(x)$ 采样，但可以估计或学习它的 score $\nabla_x\log p(x)$，就可以用“沿 score 做梯度上升 + 注入高斯噪声”的方式，把样本逐渐推向高概率区域。

扩散模型的反向 SDE 可以看成一种随时间变化的 score-based Langevin sampling。正向过程不断把数据分布 $q(x_0)$ 平滑成噪声分布；反向过程则需要在每个噪声水平 $t$ 上知道当前边缘分布 $q(x_t)$ 的 score：

$$\nabla_{x_t}\log q(x_t)$$

然后用这个 score 把 $x_t$ 推回更接近数据流形的区域。Song et al. 的 score-based generative modeling 正是从这个角度统一了 score matching、Langevin dynamics 和扩散模型：先学习不同噪声水平下的 score，再用反向 SDE 或相关采样器生成数据。

在 DDPM 中，反向一步写成：

$$x_{t-1} = \mu_\theta(x_t,t) + \sigma_t z,\qquad z\sim\mathcal{N}(0,\mathbf{I})$$

它也可以理解为一个离散的 Langevin-like step：$\mu_\theta(x_t,t)$ 提供向高概率区域移动的确定性方向，$\sigma_t z$ 保留随机采样所需的噪声。后文 Section 7 会推导 $\mu_\theta(x_t,t)$ 的具体形式。

DDPM 通常不直接训练网络输出 score，而是训练网络预测正向加噪公式中的噪声 $\epsilon$。由 Section 3 的直达公式：

$$x_t = \sqrt{\bar{\alpha}_t}\,x_0 + \sqrt{1-\bar{\alpha}_t}\,\epsilon$$

如果固定 $x_0$，则条件分布 $q(x_t\vert{}x_0)$ 的 score 为：

$$\nabla_{x_t}\log q(x_t\vert{}x_0)
= -\frac{x_t-\sqrt{\bar{\alpha}_t}x_0}{1-\bar{\alpha}_t}
= -\frac{\epsilon}{\sqrt{1-\bar{\alpha}_t}}$$

因此在训练好的模型中，噪声预测与 score 之间有近似关系：

$$\epsilon_\theta(x_t,t)\approx -\sqrt{1-\bar{\alpha}_t}\,\nabla_{x_t}\log q(x_t)$$

等价地：

$$\nabla_{x_t}\log q(x_t)\approx -\frac{1}{\sqrt{1-\bar{\alpha}_t}}\epsilon_\theta(x_t,t)$$

关键理解是：DDPM 表面上在学习预测噪声，实质上是在学习不同噪声水平下的 score function。采样时，模型把这个学到的 score 代入反向高斯转移中，于是生成过程就可以看作由学习到的 score 引导的 Langevin dynamics。

---

## 2. 正向扩散：单步转移

正向扩散过程定义为一个固定的马尔可夫链：

![正向高斯加噪过程示意图](/assets/images/diffusion/forward-gaussian-process.jpg)

$$q(x_1,\dots,x_T \vert{} x_0) = \prod_{t=1}^T q(x_t \vert{} x_{t-1})$$

每一步只依赖前一步：

$$q(x_t \vert{} x_{t-1}) = \mathcal{N}(x_t; \sqrt{1 - \beta_t}\,x_{t-1}, \beta_t \mathbf{I})$$

其中 $\beta_t \in (0,1)$ 是预先设定的 variance schedule。通常 $\beta_t$ 随 $t$ 逐渐增大。

令

$$\alpha_t = 1 - \beta_t$$

则单步转移也可写为：

$$q(x_t \vert{} x_{t-1}) = \mathcal{N}(x_t; \sqrt{\alpha_t}\,x_{t-1}, (1-\alpha_t)\mathbf{I})$$

对应的重参数化采样公式是：

$$x_t = \sqrt{\alpha_t}\,x_{t-1} + \sqrt{1-\alpha_t}\,\epsilon_{t-1}, \qquad \epsilon_{t-1}\sim\mathcal{N}(0,\mathbf{I})$$

### 为什么系数要开根号？

如果假设 $x_{t-1}$ 已经归一化到方差约为 $1$，且噪声 $\epsilon_{t-1}$ 与 $x_{t-1}$ 独立，则：

$$
\begin{aligned}
\operatorname{Var}(x_t)
&= \operatorname{Var}\left(\sqrt{\alpha_t}\,x_{t-1}\right)
 + \operatorname{Var}\left(\sqrt{1-\alpha_t}\,\epsilon_{t-1}\right) \\
&= \alpha_t \operatorname{Var}(x_{t-1})
 + (1-\alpha_t)\operatorname{Var}(\epsilon_{t-1}) \\
&= \alpha_t + (1-\alpha_t) \\
&= 1
\end{aligned}
$$

也就是说，根号系数让原图成分和噪声成分的方差份额相加为 $1$。如果改成

$$x_t = \alpha_t x_{t-1} + (1-\alpha_t)\epsilon_{t-1}$$

方差会变成

$$\alpha_t^2 + (1-\alpha_t)^2$$

通常小于 $1$，尺度会随步数漂移。

---

## 3. 任意步长直达公式

从单步重参数化公式出发：

$$x_t = \sqrt{\alpha_t}\,x_{t-1} + \sqrt{1-\alpha_t}\,\epsilon_{t-1}$$

继续展开 $x_{t-1}$：

$$x_{t-1} = \sqrt{\alpha_{t-1}}\,x_{t-2} + \sqrt{1-\alpha_{t-1}}\,\epsilon_{t-2}$$

代入得到：

$$
\begin{aligned}
x_t
&= \sqrt{\alpha_t}\left(\sqrt{\alpha_{t-1}}\,x_{t-2}
 + \sqrt{1-\alpha_{t-1}}\,\epsilon_{t-2}\right)
 + \sqrt{1-\alpha_t}\,\epsilon_{t-1} \\
&= \sqrt{\alpha_t\alpha_{t-1}}\,x_{t-2}
 + \sqrt{\alpha_t(1-\alpha_{t-1})}\,\epsilon_{t-2}
 + \sqrt{1-\alpha_t}\,\epsilon_{t-1}
\end{aligned}
$$

后两项是独立高斯噪声的线性组合。它们的总方差为：

$$
\begin{aligned}
\alpha_t(1-\alpha_{t-1}) + (1-\alpha_t)
&= \alpha_t - \alpha_t\alpha_{t-1} + 1 - \alpha_t \\
&= 1 - \alpha_t\alpha_{t-1}
\end{aligned}
$$

因此可以合并为一个标准高斯噪声 $\epsilon \sim \mathcal{N}(0,\mathbf{I})$：

$$x_t = \sqrt{\alpha_t\alpha_{t-1}}\,x_{t-2}
 + \sqrt{1-\alpha_t\alpha_{t-1}}\,\epsilon$$

令

$$\bar{\alpha}_t = \prod_{i=1}^t \alpha_i$$

归纳可得：

$$\boxed{x_t = \sqrt{\bar{\alpha}_t}\,x_0 + \sqrt{1-\bar{\alpha}_t}\,\epsilon, \qquad \epsilon\sim\mathcal{N}(0,\mathbf{I})}$$

对应的条件分布是：

$$\boxed{q(x_t \vert{} x_0) = \mathcal{N}\big(x_t; \sqrt{\bar{\alpha}_t}\,x_0, (1-\bar{\alpha}_t)\mathbf{I}\big)}$$

这条公式把训练时生成 $x_t$ 的成本从 $O(t)$ 降到 $O(1)$。

---

## 4. 终点为什么接近标准高斯？

由直达公式：

$$x_t = \sqrt{\bar{\alpha}_t}\,x_0 + \sqrt{1-\bar{\alpha}_t}\,\epsilon$$

可见，只要 $\bar{\alpha}_t$ 足够小，$x_t$ 就主要由标准高斯噪声决定。

需要注意：仅有 $\beta_t \in (0,1)$ 并不足以保证无限乘积 $\bar{\alpha}_t = \prod_{i=1}^t(1-\beta_i)$ 收敛到 $0$。严格地说，需要：

$$\sum_{i=1}^{\infty}\beta_i = \infty$$

因为当 $\beta_i$ 较小时，$\log(1-\beta_i) \approx -\beta_i$，于是：

$$
\log \bar{\alpha}_t
= \sum_{i=1}^t \log(1-\beta_i)
\le -\sum_{i=1}^t \beta_i
$$

如果 $\sum_i \beta_i$ 发散，则 $\log \bar{\alpha}_t \to -\infty$，因此 $\bar{\alpha}_t \to 0$。

在实际 DDPM 中，$T$ 是有限的，通常选择 schedule 使 $\bar{\alpha}_T$ 很小。于是：

$$q(x_T\vert{}x_0)
= \mathcal{N}\big(x_T; \sqrt{\bar{\alpha}_T}x_0, (1-\bar{\alpha}_T)\mathbf{I}\big)
\approx \mathcal{N}(0,\mathbf{I})$$

这里的“接近”是有限步近似，不是说任何有限 $T$ 下都精确等于标准高斯。

---

## 5. 逐像素理解与代码实现

协方差矩阵 $\beta_t\mathbf{I}$ 表示每个维度的噪声方差都是 $\beta_t$，不同维度之间协方差为 $0$。在实现中通常直接生成与图像同形状的标准高斯噪声，然后按元素计算。

单步公式：

```python
import math
import torch

beta_t = 0.02
sqrt_alpha = math.sqrt(1.0 - beta_t)
sqrt_beta = math.sqrt(beta_t)

noise = torch.randn_like(x_t_minus_1)
x_t = sqrt_alpha * x_t_minus_1 + sqrt_beta * noise
```

训练时更常用直达公式。下面代码展示批量生成任意时间步的 $x_t$：

```python
import torch
import torch.nn.functional as F

def extract(values, t, x_shape):
    """
    values: [T]
    t: [B]
    返回可广播到 x_shape 的系数张量。
    """
    out = values.gather(0, t)
    return out.view(t.shape[0], *((1,) * (len(x_shape) - 1)))

def forward_diffusion_batch(x_0, alphas_cumprod):
    batch_size = x_0.shape[0]
    t = torch.randint(0, len(alphas_cumprod), (batch_size,), device=x_0.device)

    epsilon = torch.randn_like(x_0)
    sqrt_alpha_bar = torch.sqrt(extract(alphas_cumprod, t, x_0.shape))
    sqrt_one_minus_alpha_bar = torch.sqrt(
        1.0 - extract(alphas_cumprod, t, x_0.shape)
    )

    x_t = sqrt_alpha_bar * x_0 + sqrt_one_minus_alpha_bar * epsilon
    return x_t, t, epsilon

# 训练步骤：
# x_t, t, noise = forward_diffusion_batch(x_0, alphas_cumprod)
# noise_pred = model(x_t, t)
# loss = F.mse_loss(noise_pred, noise)
```

---

## 6. 训练目标：为什么预测噪声？

由直达公式：

$$x_t = \sqrt{\bar{\alpha}_t}\,x_0 + \sqrt{1-\bar{\alpha}_t}\,\epsilon$$

训练时我们自己采样了 $\epsilon$，因此它是已知监督信号。DDPM 常用的简化目标是：

$$\boxed{L_{\text{simple}}(\theta)
= \mathbb{E}_{t,x_0,\epsilon}
\left[\left\Vert \epsilon - \epsilon_\theta(x_t,t) \right\Vert^2\right]}$$

这里 $\epsilon_\theta(x_t,t)$ 是神经网络对噪声的预测。

预测噪声有两个实际好处：

1. $\epsilon$ 始终来自 $\mathcal{N}(0,\mathbf{I})$，目标尺度稳定；
2. 只要知道 $\epsilon$，就能由直达公式反推出 $x_0$：

$$
\hat{x}_0
= \frac{x_t - \sqrt{1-\bar{\alpha}_t}\,\epsilon_\theta(x_t,t)}
{\sqrt{\bar{\alpha}_t}}
$$

所以预测噪声与预测干净图像在代数上可以互相转换，但噪声目标通常更稳定。

---

## 6.1. 从 ELBO 推导训练目标

Section 6 直接给出了常用的简化噪声预测目标。本节补上它和变分下界之间的关系：DDPM 的训练并不是凭空选择 MSE，而是从最大化数据似然的变分下界推导而来，最后再简化成预测噪声。

生成模型的目标是最大化 $\log p_\theta(x_0)$，等价于最小化负对数似然 $-\log p_\theta(x_0)$。由于反向链中存在隐变量 $x_1,\dots,x_T$，直接计算边缘似然很困难，于是引入正向扩散过程 $q(x_{1:T}\vert{}x_0)$ 作为变分分布。负对数似然有如下上界：

$$-\log p_\theta(x_0) \leq \mathbb{E}_q\left[-\log p(x_T) - \sum_{t \geq 1} \log \frac{p_\theta(x_{t-1}\vert{}x_t)}{q(x_t\vert{}x_{t-1})}\right]$$

这个式子就是负 ELBO 的一种写法。进一步利用 Section 2 的马尔可夫结构和 Section 7 中的后验 $q(x_{t-1}\vert{}x_t,x_0)$，可以整理成 KL divergence 的形式：

$$\mathcal{L} = \mathbb{E}_q\left[
\underbrace{D_{KL}(q(x_T\vert{}x_0) \Vert p(x_T))}_{L_T}
+ \sum_{t=2}^T \underbrace{D_{KL}(q(x_{t-1}\vert{}x_t, x_0) \Vert p_\theta(x_{t-1}\vert{}x_t))}_{L_{t-1}}
- \underbrace{\log p_\theta(x_0\vert{}x_1)}_{L_0}
\right]$$

其中 $L_T$ 约束正向终点接近先验 $p(x_T)=\mathcal{N}(0,\mathbf{I})$；$L_0$ 对应最后一步从 $x_1$ 还原 $x_0$；中间项 $L_{t-1}$ 则训练模型反向转移 $p_\theta(x_{t-1}\vert{}x_t)$ 去匹配真实后验 $q(x_{t-1}\vert{}x_t,x_0)$。

对 $t\ge 2$，两边都是高斯分布：

$$q(x_{t-1}\vert{}x_t,x_0)
= \mathcal{N}\big(x_{t-1};\tilde{\mu}_t(x_t,x_0),\tilde{\beta}_t\mathbf{I}\big)$$

$$p_\theta(x_{t-1}\vert{}x_t)
= \mathcal{N}\big(x_{t-1};\mu_\theta(x_t,t),\sigma_t^2\mathbf{I}\big)$$

两个协方差固定的高斯分布之间的 KL divergence，只剩下均值差的二次项和与 $\theta$ 无关的常数。忽略不影响优化的常数后：

$$L_{t-1}
= \mathbb{E}_{x_0,\epsilon}\left[
\frac{1}{2\sigma_t^2}
\left\Vert\tilde{\mu}_t(x_t,x_0)-\mu_\theta(x_t,t)\right\Vert^2
\right]$$

这里的 $x_t$ 由 Section 3 的重参数化公式生成：

$$x_t = \sqrt{\bar{\alpha}_t}\,x_0 + \sqrt{1-\bar{\alpha}_t}\,\epsilon,\qquad \epsilon\sim\mathcal{N}(0,\mathbf{I})$$

Section 7 会推导出真实后验均值可以写成关于真实噪声 $\epsilon$ 的形式：

$$\tilde{\mu}_t(x_t,\epsilon)
= \frac{1}{\sqrt{\alpha_t}}
\left(
x_t-\frac{\beta_t}{\sqrt{1-\bar{\alpha}_t}}\epsilon
\right)$$

模型均值则用预测噪声 $\epsilon_\theta(x_t,t)$ 参数化：

$$\mu_\theta(x_t,t)
= \frac{1}{\sqrt{\alpha_t}}
\left(
x_t-\frac{\beta_t}{\sqrt{1-\bar{\alpha}_t}}\epsilon_\theta(x_t,t)
\right)$$

因此两者的差为：

$$
\begin{aligned}
\tilde{\mu}_t(x_t,\epsilon)-\mu_\theta(x_t,t)
&= \frac{1}{\sqrt{\alpha_t}}
\left(
-\frac{\beta_t}{\sqrt{1-\bar{\alpha}_t}}\epsilon
+\frac{\beta_t}{\sqrt{1-\bar{\alpha}_t}}\epsilon_\theta(x_t,t)
\right) \\
&= \frac{\beta_t}{\sqrt{\alpha_t}\sqrt{1-\bar{\alpha}_t}}
\left(\epsilon_\theta(x_t,t)-\epsilon\right)
\end{aligned}
$$

代回 $L_{t-1}$：

$$L_{t-1}
= \mathbb{E}_{x_0,\epsilon}\left[
\frac{\beta_t^2}{2\sigma_t^2\alpha_t(1-\bar{\alpha}_t)}
\left\Vert \epsilon-\epsilon_\theta(x_t,t)\right\Vert^2
\right]$$

严格的 ELBO 对不同时间步会有一个依赖 $t$ 的权重：

$$w_t = \frac{\beta_t^2}{2\sigma_t^2\alpha_t(1-\bar{\alpha}_t)}$$

Ho et al. 的 DDPM 发现，去掉这个权重、直接均匀采样时间步并优化噪声 MSE，通常会得到更稳定、更好的生成质量。于是得到 Section 6 使用的简化目标：

$$L_{\text{simple}}(\theta)
= \mathbb{E}_{t,x_0,\epsilon}\left[
\left\Vert\epsilon-\epsilon_\theta(x_t,t)\right\Vert^2
\right]$$

这个期望包含三层随机性：

1. $t$ 来自时间步采样，通常在 $\{1,\dots,T\}$ 上均匀采样，用来训练模型处理不同噪声强度；
2. $x_0$ 来自训练数据分布 $q(x_0)$，对应真实图片样本；
3. $\epsilon\sim\mathcal{N}(0,\mathbf{I})$ 是正向加噪时采样的高斯噪声，决定当前训练样本的具体 $x_t$。

所以，$L_{\text{simple}}$ 不是独立于 ELBO 的经验技巧，而是把高斯 KL 中的均值匹配项重参数化为噪声预测，再省略时间相关权重后的简化版本。

---

## 6.2. KL 散度：从定义到直观理解

Section 6.1 中的 ELBO 反复出现 KL divergence。它衡量的是：如果真实数据来自分布 $P$，但我们用另一个分布 $Q$ 来编码或近似它，会额外付出多少信息代价。

### 离散形式

对离散随机变量，KL 散度定义为：

$$D_{KL}(P \Vert Q) = \sum_x P(x) \log \frac{P(x)}{Q(x)}$$

这里的期望是在 $P$ 下计算的，所以 $P(x)$ 大的地方更重要。如果某些 $x$ 在真实分布 $P$ 中经常出现，但在近似分布 $Q$ 中概率很低，那么 $\log \frac{P(x)}{Q(x)}$ 会很大，KL 散度就会明显增加。

KL 散度有几个关键性质：

1. 非负：$D_{KL}(P\Vert Q)\ge 0$；
2. 不对称：通常 $D_{KL}(P\Vert Q)\ne D_{KL}(Q\Vert P)$；
3. 当且仅当 $P=Q$ 时为 $0$。

例如考虑三分类分布：

$$P=(0.5,0.3,0.2),\qquad Q=(0.4,0.4,0.2)$$

则：

$$
\begin{aligned}
D_{KL}(P\Vert Q)
&=0.5\log\frac{0.5}{0.4}
 +0.3\log\frac{0.3}{0.4}
 +0.2\log\frac{0.2}{0.2} \\
&\approx 0.0204
\end{aligned}
$$

反过来：

$$
\begin{aligned}
D_{KL}(Q\Vert P)
&=0.4\log\frac{0.4}{0.5}
 +0.4\log\frac{0.4}{0.3}
 +0.2\log\frac{0.2}{0.2} \\
&\approx 0.0258
\end{aligned}
$$

这说明 KL 散度不是普通距离。它关心的是“用 $Q$ 描述来自 $P$ 的样本”时的额外代价，因此交换 $P$ 和 $Q$ 会改变问题本身。

### 连续形式

对连续随机变量，求和变成积分：

$$D_{KL}(P \Vert Q) = \int p(x) \log \frac{p(x)}{q(x)} dx$$

在 DDPM 中最常见的是两个高斯分布之间的 KL。考虑一维情形：

$$P=\mathcal{N}(\mu_1,\sigma_1^2),\qquad Q=\mathcal{N}(\mu_2,\sigma_2^2)$$

两者的对数密度差为：

$$
\log\frac{p(x)}{q(x)}
= \log\frac{\sigma_2}{\sigma_1}
+ \frac{(x-\mu_2)^2}{2\sigma_2^2}
- \frac{(x-\mu_1)^2}{2\sigma_1^2}
$$

对 $x\sim P$ 取期望。由于：

$$\mathbb{E}_P[(x-\mu_1)^2]=\sigma_1^2$$

以及：

$$
\mathbb{E}_P[(x-\mu_2)^2]
=\mathbb{E}_P[(x-\mu_1+\mu_1-\mu_2)^2]
=\sigma_1^2+(\mu_1-\mu_2)^2
$$

得到闭式公式：

$$\boxed{
D_{KL}\big(\mathcal{N}(\mu_1,\sigma_1^2)\Vert \mathcal{N}(\mu_2,\sigma_2^2)\big)
= \log\frac{\sigma_2}{\sigma_1}
+ \frac{\sigma_1^2 + (\mu_1-\mu_2)^2}{2\sigma_2^2}
- \frac{1}{2}
}$$

多维各向同性高斯或对角高斯的情况可以看成对各维度求和。DDPM 的 Section 6.1 中，每个中间项

$$D_{KL}(q(x_{t-1}\vert{}x_t,x_0)\Vert p_\theta(x_{t-1}\vert{}x_t))$$

本质上就是高斯到高斯的 KL。若方差 $\tilde{\beta}_t\mathbf{I}$ 和 $\sigma_t^2\mathbf{I}$ 固定，公式中与 $\theta$ 相关的部分只剩均值差：

$$\left\Vert\tilde{\mu}_t(x_t,x_0)-\mu_\theta(x_t,t)\right\Vert^2$$

这解释了为什么 DDPM 的训练可以转化为让模型预测正确的反向均值；再通过 Section 7 的重参数化，均值匹配又进一步变成了噪声预测 MSE。方差相关项不是消失了，而是在常见设定下不依赖 $\theta$，因此不会影响均值网络的优化方向。

为了更好地理解 KL 散度的几何意义，你可以在下面的交互式模拟中拖动参数，直观感受两个高斯分布之间 KL 散度的变化。

<iframe src="/kl-simulation.html" width="100%" height="800" style="border: 1px solid #ddd; border-radius: 8px; margin: 1.5em 0;"></iframe>

---

## 7. 反向过程：从后验到采样公式

反向生成希望从 $x_T\sim\mathcal{N}(0,\mathbf{I})$ 出发，逐步采样 $x_{T-1},x_{T-2},\dots,x_0$。真正想要的是：

$$q(x_{t-1}\vert{}x_t)$$

但这个分布依赖未知的数据分布 $q(x_0)$，无法直接解析。DDPM 采用的关键中间量是给定 $x_0$ 时的后验：

$$q(x_{t-1}\vert{}x_t,x_0)$$

它可以由贝叶斯公式写成：

$$
\begin{aligned}
q(x_{t-1}\vert{}x_t,x_0)
&= \frac{q(x_{t-1},x_t,x_0)}{q(x_t,x_0)} \\
&= \frac{q(x_t\vert{}x_{t-1},x_0)q(x_{t-1}\vert{}x_0)q(x_0)}
{q(x_t\vert{}x_0)q(x_0)} \\
&= q(x_t\vert{}x_{t-1})\frac{q(x_{t-1}\vert{}x_0)}{q(x_t\vert{}x_0)}
\end{aligned}
$$

最后一步使用了马尔可夫性：

$$q(x_t\vert{}x_{t-1},x_0)=q(x_t\vert{}x_{t-1})$$

右边三项都是已知高斯分布：

$$q(x_t\vert{}x_{t-1})
= \mathcal{N}(x_t;\sqrt{\alpha_t}x_{t-1},\beta_t\mathbf{I})$$

$$q(x_{t-1}\vert{}x_0)
= \mathcal{N}(x_{t-1};\sqrt{\bar{\alpha}_{t-1}}x_0,(1-\bar{\alpha}_{t-1})\mathbf{I})$$

$$q(x_t\vert{}x_0)
= \mathcal{N}(x_t;\sqrt{\bar{\alpha}_t}x_0,(1-\bar{\alpha}_t)\mathbf{I})$$

因此后验仍是高斯分布：

$$\boxed{q(x_{t-1}\vert{}x_t,x_0)
= \mathcal{N}\big(x_{t-1};\tilde{\mu}_t(x_t,x_0),\tilde{\beta}_t\mathbf{I}\big)}$$

其中：

$$\boxed{\tilde{\beta}_t
= \frac{1-\bar{\alpha}_{t-1}}{1-\bar{\alpha}_t}\beta_t}$$

$$\boxed{\tilde{\mu}_t(x_t,x_0)
= \frac{\sqrt{\alpha_t}(1-\bar{\alpha}_{t-1})}{1-\bar{\alpha}_t}x_t
+ \frac{\sqrt{\bar{\alpha}_{t-1}}\beta_t}{1-\bar{\alpha}_t}x_0}$$

### 高斯乘除的代数草图

把与 $x_{t-1}$ 有关的两项写成指数形式。令 $y=x_{t-1}$，忽略与 $y$ 无关的常数：

$$
\begin{aligned}
\log q(y\vert{}x_t,x_0)
&= \log q(x_t\vert{}y) + \log q(y\vert{}x_0) + C \\
&= -\frac{1}{2\beta_t}\Vert x_t-\sqrt{\alpha_t}y\Vert^2
-\frac{1}{2(1-\bar{\alpha}_{t-1})}
\Vert y-\sqrt{\bar{\alpha}_{t-1}}x_0\Vert^2 + C
\end{aligned}
$$

展开关于 $y$ 的二次项和一次项：

$$
\begin{aligned}
\log q(y\vert{}x_t,x_0)
&= -\frac{1}{2}
\left[
\left(\frac{\alpha_t}{\beta_t}
+\frac{1}{1-\bar{\alpha}_{t-1}}\right)\Vert y\Vert^2
-2\left(
\frac{\sqrt{\alpha_t}}{\beta_t}x_t
+\frac{\sqrt{\bar{\alpha}_{t-1}}}{1-\bar{\alpha}_{t-1}}x_0
\right)^\top y
\right] + C
\end{aligned}
$$

所以后验精度（方差倒数）为：

$$
\tilde{\beta}_t^{-1}
= \frac{\alpha_t}{\beta_t}
+\frac{1}{1-\bar{\alpha}_{t-1}}
= \frac{1-\bar{\alpha}_t}{\beta_t(1-\bar{\alpha}_{t-1})}
$$

因此：

$$\tilde{\beta}_t
= \frac{1-\bar{\alpha}_{t-1}}{1-\bar{\alpha}_t}\beta_t$$

均值等于“方差 $\times$ 一次项系数”：

$$
\tilde{\mu}_t
= \tilde{\beta}_t
\left(
\frac{\sqrt{\alpha_t}}{\beta_t}x_t
+\frac{\sqrt{\bar{\alpha}_{t-1}}}{1-\bar{\alpha}_{t-1}}x_0
\right)
$$

代入 $\tilde{\beta}_t$ 得：

$$
\tilde{\mu}_t
= \frac{\sqrt{\alpha_t}(1-\bar{\alpha}_{t-1})}{1-\bar{\alpha}_t}x_t
+ \frac{\sqrt{\bar{\alpha}_{t-1}}\beta_t}{1-\bar{\alpha}_t}x_0
$$

这就是上面的后验均值公式。

### 从真实噪声到网络预测噪声

由直达公式可得：

$$x_0
= \frac{x_t-\sqrt{1-\bar{\alpha}_t}\epsilon}{\sqrt{\bar{\alpha}_t}}$$

把它代入 $\tilde{\mu}_t(x_t,x_0)$：

$$
\begin{aligned}
\tilde{\mu}_t
&= \frac{\sqrt{\alpha_t}(1-\bar{\alpha}_{t-1})}{1-\bar{\alpha}_t}x_t
+ \frac{\sqrt{\bar{\alpha}_{t-1}}\beta_t}{1-\bar{\alpha}_t}
\cdot
\frac{x_t-\sqrt{1-\bar{\alpha}_t}\epsilon}{\sqrt{\bar{\alpha}_t}} \\
&= \left[
\frac{\sqrt{\alpha_t}(1-\bar{\alpha}_{t-1})}{1-\bar{\alpha}_t}
+ \frac{\beta_t}{\sqrt{\alpha_t}(1-\bar{\alpha}_t)}
\right]x_t
- \frac{\beta_t}{\sqrt{\alpha_t}\sqrt{1-\bar{\alpha}_t}}\epsilon
\end{aligned}
$$

其中使用了 $\bar{\alpha}_t=\alpha_t\bar{\alpha}_{t-1}$。括号中的 $x_t$ 系数化简为：

$$
\begin{aligned}
\frac{\sqrt{\alpha_t}(1-\bar{\alpha}_{t-1})}{1-\bar{\alpha}_t}
+ \frac{\beta_t}{\sqrt{\alpha_t}(1-\bar{\alpha}_t)}
&= \frac{\alpha_t(1-\bar{\alpha}_{t-1})+\beta_t}
{\sqrt{\alpha_t}(1-\bar{\alpha}_t)} \\
&= \frac{\alpha_t-\bar{\alpha}_t+1-\alpha_t}
{\sqrt{\alpha_t}(1-\bar{\alpha}_t)} \\
&= \frac{1-\bar{\alpha}_t}{\sqrt{\alpha_t}(1-\bar{\alpha}_t)} \\
&= \frac{1}{\sqrt{\alpha_t}}
\end{aligned}
$$

因此：

$$\boxed{
\tilde{\mu}_t(x_t,\epsilon)
= \frac{1}{\sqrt{\alpha_t}}
\left(
x_t - \frac{\beta_t}{\sqrt{1-\bar{\alpha}_t}}\epsilon
\right)
}$$

推理时真实 $\epsilon$ 不可得，所以用神经网络预测值 $\epsilon_\theta(x_t,t)$ 代替：

$$\boxed{
\mu_\theta(x_t,t)
= \frac{1}{\sqrt{\alpha_t}}
\left(
x_t - \frac{\beta_t}{\sqrt{1-\bar{\alpha}_t}}\epsilon_\theta(x_t,t)
\right)
}$$

反向转移被参数化为：

$$p_\theta(x_{t-1}\vert{}x_t)
= \mathcal{N}\big(x_{t-1};\mu_\theta(x_t,t),\sigma_t^2\mathbf{I}\big)$$

$\sigma_t^2$ 可以取 $\tilde{\beta}_t$，也有实现直接取 $\beta_t$ 或使用可学习方差。经典 DDPM 的采样一步为：

$$\boxed{
x_{t-1}
= \frac{1}{\sqrt{\alpha_t}}
\left(
x_t - \frac{\beta_t}{\sqrt{1-\bar{\alpha}_t}}\epsilon_\theta(x_t,t)
\right)
+ \sigma_t z
}$$

其中 $z\sim\mathcal{N}(0,\mathbf{I})$，当 $t=1$ 或实现中的最后一步时通常令 $z=0$。

这条公式解释了训练目标和采样步骤的连接：训练让 $\epsilon_\theta(x_t,t)$ 逼近真实噪声 $\epsilon$；采样时把这个预测噪声代入后验均值，从而得到 $x_{t-1}$ 的采样分布中心。

---

## 8. 反向采样代码

下面是与上式对应的简化 PyTorch 采样框架：

```python
import torch

@torch.no_grad()
def sample(model, shape, betas, alphas, alphas_cumprod, sigmas):
    device = next(model.parameters()).device
    x = torch.randn(shape, device=device)

    for t in reversed(range(len(betas))):
        t_tensor = torch.full((shape[0],), t, device=device, dtype=torch.long)

        beta_t = betas[t]
        alpha_t = alphas[t]
        alpha_bar_t = alphas_cumprod[t]
        sigma_t = sigmas[t]

        eps_pred = model(x, t_tensor)
        mean = (1.0 / torch.sqrt(alpha_t)) * (
            x - beta_t / torch.sqrt(1.0 - alpha_bar_t) * eps_pred
        )

        if t > 0:
            z = torch.randn_like(x)
            x = mean + sigma_t * z
        else:
            x = mean

    return x
```

实际项目会额外处理条件输入、时间嵌入、图像归一化、方差策略和采样器替换（如 DDIM、DPM-Solver），但 DDPM 的基础逻辑就是上面的均值预测和高斯采样。

---

## 9. 总结

DDPM 的数学结构可以压缩为三条主线：

1. 正向过程固定为高斯马尔可夫链：

$$q(x_t\vert{}x_{t-1})=\mathcal{N}(x_t;\sqrt{\alpha_t}x_{t-1},\beta_t\mathbf{I})$$

2. 高斯线性组合给出任意步长闭式公式：

$$q(x_t\vert{}x_0)=\mathcal{N}(x_t;\sqrt{\bar{\alpha}_t}x_0,(1-\bar{\alpha}_t)\mathbf{I})$$

3. 给定 $x_0$ 的反向后验可解析，训练网络预测噪声并代入后验均值：

$$
\mu_\theta(x_t,t)
= \frac{1}{\sqrt{\alpha_t}}
\left(
x_t-\frac{\beta_t}{\sqrt{1-\bar{\alpha}_t}}\epsilon_\theta(x_t,t)
\right)
$$

正向过程负责构造可监督的噪声预测任务；反向过程把噪声预测转化为逐步采样。两者由同一个高斯后验推导连接起来。
