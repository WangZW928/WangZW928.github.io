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

## 2. 正向扩散：单步转移

正向扩散过程定义为一个固定的马尔可夫链：

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
