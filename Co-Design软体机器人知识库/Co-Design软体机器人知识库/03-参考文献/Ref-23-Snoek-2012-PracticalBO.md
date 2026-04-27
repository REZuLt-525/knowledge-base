# [23] Snoek, Larochelle & Adams 2012 · Practical Bayesian Optimization of ML Algorithms

> **原文**: J. Snoek, H. Larochelle, R. P. Adams. *Practical Bayesian optimization of machine learning algorithms.* **NIPS 2012**. arXiv:1206.2944
> **译名**: 《机器学习算法的实用贝叶斯优化》 — 神经信息处理系统大会 (NIPS), 2012.

`#文献-方法` `#贝叶斯优化` `#里程碑`

---

## 1 · 研究问题

机器学习模型有许多超参 (学习率、正则化系数、网络宽度等), 调优长期靠 grid search / 随机搜索 / 经验。Toronto 大学 Snoek + Larochelle + Adams 提出:

> **如何把贝叶斯优化 (BO) 真正做"实用"** —— 解决 (a) 核函数选择, (b) 并行评估, (c) 整数/类别变量, (d) 计算时间作为约束 等四大瓶颈, 让 BO 比手调超参更快。

测试场景: 让 BO 找到 CIFAR-10 上 CNN 的最佳超参。

---

## 2 · 贝叶斯优化形式化

### 2.1 问题设定

最小化黑盒函数:

$$x^* = \arg\min_{x \in \mathcal{X}} f(x)$$

**变量含义**：
- $x$ — 超参向量 (例如学习率、正则化系数等)。
- $\mathcal{X}$ — 超参空间。
- $f$ — 黑盒函数 (例如训完模型测得的验证错误率)。
- $x^*$ — 最优超参。

**关键约束**：
- $f$ 评估贵 (训一个 CNN 需小时)。
- $f$ 不可微 (只能输入 → 输出)。
- 目标: 在有限预算 $T$ 下逼近 $x^*$。

### 2.2 高斯过程代理

假设 $f$ 是 GP 实现:

$$f \sim \mathcal{GP}(m(x), k(x, x'))$$

**变量含义**：
- $\mathcal{GP}$ — 高斯过程: "任何有限点上 $f$ 服从联合高斯分布"。
- $m(x)$ — 均值函数 (常取 $m \equiv 0$)。
- $k(x, x')$ — 核函数 (协方差函数), 度量 $f(x), f(x')$ 之间相关程度。

**公式解读**：把"对函数 $f$ 的不确定性"用概率分布刻画。GP 提供 **后验均值** + **后验方差**。

观测 $D_n = \{(x_i, y_i)\}_{i=1}^n$ (假设 $y_i = f(x_i) + \epsilon_i$, $\epsilon_i \sim \mathcal{N}(0, \sigma_n^2)$)。

### 2.3 GP 后验

$$\mu_n(x) = \mathbf{k}(x)^\top \mathbf{K}_n^{-1} \mathbf{y}$$

$$\sigma_n^2(x) = k(x, x) - \mathbf{k}(x)^\top \mathbf{K}_n^{-1} \mathbf{k}(x)$$

**变量含义**：
- $\mu_n(x)$ — 在新点 $x$ 的 **后验均值** (最佳点估计)。
- $\sigma_n^2(x)$ — 后验 **方差** (不确定度)。
- $\mathbf{y} = (y_1, ..., y_n)^\top$ — 观测值列向量。
- $\mathbf{K}_n \in \mathbb{R}^{n \times n}$, $(\mathbf{K}_n)_{ij} = k(x_i, x_j) + \sigma_n^2 \delta_{ij}$ — 训练点协方差矩阵 + 噪声项。
- $\mathbf{k}(x) \in \mathbb{R}^n$, $(\mathbf{k}(x))_i = k(x_i, x)$ — 新点 $x$ 与训练点的协方差。

**公式解读**：
- 均值是观测值 $\mathbf{y}$ 的加权和, 权重由"$x$ 与各 $x_i$ 的相似度"决定。
- 方差反映: 离训练点越远, 方差越大 (越不确定)。

---

## 3 · 三大实用技术

### 3.1 Matérn 5/2 核 (替代 RBF)

**RBF 核** $k(r) = \exp(-r^2/2\ell^2)$ 假设 $f$ **无限光滑**, 不适合 ML 超参调优 (实际函数往往只是几次可微)。

**Matérn 5/2** (二阶可微):
$$\boxed{k_{\nu=5/2}(r) = \sigma_f^2 \left(1 + \sqrt{5}\,r + \tfrac{5 r^2}{3}\right) \exp\!\left(-\sqrt{5}\,r\right)}$$

**变量含义**：
- $r$ — 归一化距离 (无量纲), 见下。
- $\sigma_f^2$ — **信号方差**: 函数整体幅度 (越大, 后验方差越大)。
- $\nu = 5/2$ — Matérn 核的"光滑参数"; $\nu = \infty$ 退化为 RBF, $\nu = 1/2$ 退化为指数核。

归一化距离 (ARD, Automatic Relevance Determination):
$$r = \sqrt{\textstyle \sum_{d=1}^D (x_d - x'_d)^2 / \ell_d^2}$$

**变量含义**：
- $D$ — 输入维度。
- $\ell_d$ — 第 $d$ 维的 **长度尺度**: 该维上"多大变化才显著影响 $f$"。
- $\ell_d$ 大 → 该维不重要 (ARD 自动学得)。

**公式解读**：核函数表示 "**两点相似度**":
- $r = 0$ ($x = x'$): $k = \sigma_f^2$ (自己最相似)。
- $r \to \infty$: $k \to 0$ (远点不相关)。
- $\nu = 5/2$ → 函数二阶可微 → 适合"半光滑"目标。

### 3.2 EI 采集函数 + 闭式

EI 定义:
$$\text{EI}(x) = \mathbb{E}_{Y \sim p(f(x)|D)}\left[\max(y^* - Y, 0)\right]$$

**变量含义**：
- $Y$ — $f(x)$ 的随机变量 (依据 GP 后验)。
- $p(f(x)|D)$ — GP 给出的 $Y$ 分布: $\mathcal{N}(\mu(x), \sigma^2(x))$。
- $y^*$ — 当前最佳 (假设最小化, $y^* = \min_i y_i$)。
- $\max(y^* - Y, 0)$ — "**改进量**": $Y$ 比 $y^*$ 好多少 (不好则为 0)。

**公式解读**：EI 是 "**改进量的期望**" — 衡量在 $x$ 评估能带来多大平均提升。

由于 $Y \sim \mathcal{N}(\mu(x), \sigma^2(x))$, 解析积分:

$$\boxed{\text{EI}(x) = (y^* - \mu(x))\Phi(z) + \sigma(x)\phi(z),\quad z = \frac{y^* - \mu(x)}{\sigma(x)}}$$

**变量含义**：
- $z$ — **标准化改进量**: 当前最佳超过预测均值 $z$ 个标准差。
- $\Phi(z)$ — 标准正态 **累积分布函数 (CDF)**: $P(Z \le z), Z \sim \mathcal{N}(0,1)$。
- $\phi(z)$ — 标准正态 **概率密度函数 (PDF)**: $\phi(z) = \frac{1}{\sqrt{2\pi}} e^{-z^2/2}$。

**公式解读**：
- **第 1 项** $(y^*-\mu)\Phi(z)$: 均值改进 × 改进概率 = **利用 (exploitation)**。
  - 直觉: 如果均值低于当前最佳, 而且高概率成立 → 重点利用。
- **第 2 项** $\sigma(x)\phi(z)$: 不确定度放大利用项的"惊喜空间" = **探索 (exploration)**。
  - 直觉: 即使均值不那么好, 如果方差大也值得一试 (可能有大改进)。
- **平衡**: $\sigma$ 大处重探索; $\mu \ll y^*$ 处重利用。

**直觉**: EI 自动选 "**最有可能产生大改进的点**"。

### 3.3 并行 BO via Fantasy

要在 batch 中选 $q$ 个点同时评估 (利用并行硬件):
- 已选 $\{x_1, ..., x_p\}$ 但未评估。
- 假设其值 $\tilde{y}_i \sim \mathcal{GP}\text{-posterior}$ ("fantasy")。
- 把 $(x_i, \tilde{y}_i)$ 加入数据 → 更新 GP → 选下一个。
- Monte Carlo 多次 fantasy 后平均。

期望批次 EI:
$$q\text{-EI}(x_1, ..., x_q) = \mathbb{E}_{\tilde{y}_{1:p}}\left[\text{EI}(x_{p+1}; D \cup \tilde{y})\right]$$

**变量含义**：
- $q$ — batch 大小 (并行评估数)。
- $\tilde{y}_{1:p}$ — 假想观测 (从 GP 后验采样)。
- $D \cup \tilde{y}$ — 加入假想观测后的扩展数据集。

**公式解读**：通过模拟"如果选了 $q$ 个点会发生什么", 期望意义下选最佳 batch。

### 3.4 整数 / 类别处理

- **整数**: 拟合时连续, 评估时四舍五入。
- **类别**: 独热编码 + ARD (会自动学到该维度无关)。

### 3.5 计算时间约束

把"训练时间" 也建模为 GP $g(x)$。优化目标:
$$x_\text{next} = \arg\max_x \frac{\text{EI}(x)}{g(x)}$$

**变量含义**：
- $g(x)$ — 在超参 $x$ 下完成评估所需时间 (由 GP 估计)。
- 比值 = "**单位时间期望提升**"。

**公式解读**：偏好 "**提升大且评估快**" 的点。例如先试小网络 (快) 再试大网络。

---

## 4 · 完整 BO 工作流

```
─── Algorithm: Practical BO ──────────────────
初始: 随机抽 n_0 = 5d 个点, 评估得 D_0
重复 t = 1..T:
    1) 训 GP 超参 ℓ, σ_f, σ_n (最大边缘似然)
       ℓ* = argmax_ℓ log p(y | X, ℓ)
    2) 优化采集函数:
       x_t = argmax_x EI(x; D_{t-1})
       (用 L-BFGS-B 多 restart, 因 EI 非凸)
    3) 真实评估: y_t = f(x_t)
    4) D_t = D_{t-1} ∪ {(x_t, y_t)}
返回 (x*, y*) ∈ D_T
```

---

## 5 · 实验

### 5.1 测试场景
- 论文用 CIFAR-10 CNN 的 9 个超参:
  - learning rate, batch size, layer 1 channels, layer 2 channels, dropout, ...
- 评估 = 训完整 CNN (~30 min) → 测试集错误率。

### 5.2 量化结果

| 方法 | 最佳 CIFAR-10 错误率 | 评估次数 |
|---|---|---|
| 人工调参 (论文) | 14.98% | 不计 |
| 随机搜索 100 点 | 13.43% | 100 |
| TPE | 12.95% | 100 |
| **本文 BO** | **9.50%** | **100** |

**BO 显著超越人类专家调参**。

---

## 6 · 关键贡献

1. **Matérn 5/2 核** 普及到 ML / 工程 BO 实践。
2. **并行 BO + Fantasy MC**, 让 BO 利用并行硬件。
3. **计算时间作为约束**, 现实超参搜索的关键。
4. **代码开源 (Spearmint)**, 推动 BO 工具化 (后继 GPyOpt, BoTorch)。

---

## 7 · 局限

1. **维度 $\le 20$**: GP 拟合 $O(n^3)$, 高维不可行。
2. **超参敏感**: $\ell, \sigma_f, \sigma_n$ 选不好则失败。
3. **不确定度可能过冷**: 真实函数比 GP 假设更平滑时, EI 过早终止探索。

---

## 8 · 历史影响

- 现代 BO 工具 (Spearmint, GPyOpt, BoTorch) 设计蓝本。
- 推广到机器人步态 ([[Ref-22-Calandra-2016-BO]])、神经架构搜索、化学合成、A/B 测试。
- 是 ML 自动调参的奠基论文。

---

## 9 · 与本论文 (Yi'25) 的关系

本论文综述把 BO 列入 **"采样优化方法"** 类别。本论文 28 维设计空间正逼近 BO 维度上限, 加上多物体联合 + FEM 评估慢, 转用神经代理 + 梯度更高效。

---

*← [[_参考文献索引]]*
