# 代理模型 Surrogate Model

> **一句话理解**：用一个 **训练成本 + 推理成本 + 内存** 都远低于真实物理模拟的 **可学习函数 $\hat{f}_\theta$** 来近似真实 (但昂贵的) 函数 $f^\star$，使得后续的优化、采样、推断在 $\hat{f}_\theta$ 上完成。

`#核心概念`

---

## 1. 形式化定义

### 1.1 真实模型 vs 代理模型

$$f^\star: \mathcal{X} \to \mathcal{Y}, \quad y = f^\star(x)$$

**变量含义**：
- $f^\star$ — 真实物理 / 实验过程；视为黑盒函数。
- $\mathcal{X}$ — 输入空间，例如设计参数 $\in \mathbb{R}^d$、点云、图像等。
- $\mathcal{Y}$ — 输出空间，例如任务表现 $\in \mathbb{R}$、状态向量等。
- $x \in \mathcal{X}$ — 一次输入 (一组设计参数 / 一个状态)。
- $y \in \mathcal{Y}$ — 真实模型对此输入的输出。

**公式解读**：把"做一次仿真 / 实验" 抽象为函数调用 $y = f^\star(x)$。**问题**：$f^\star$ 慢，每次求值要 $T_\text{eval}^\star$ (秒到小时)。

我们用代理 $\hat{f}_\theta: \mathcal{X} \to \mathcal{Y}$ 替代之，**$\theta$ 是代理的参数 (NN 权重 / GP 超参)**。

**理想性质**：
1. **快**：$T_\text{eval}^{\hat f} \ll T_\text{eval}^\star$ (差 1000× 以上)。
2. **准**：$\mathbb{E}_{x \sim p(x)}[\,\| \hat{f}_\theta(x) - f^\star(x)\|^2\,] \le \epsilon$
   - **变量**：$p(x)$ 是输入分布；$\epsilon$ 是允许误差上限。
   - **解读**：在期望意义下，代理的预测与真实输出的均方误差不大。
3. **可微**：$\nabla_x \hat{f}_\theta$ 存在 → 可对输入做梯度优化。

---

## 2. 高斯过程代理 (Gaussian Process, GP)

### 2.1 GP 先验

$$f \sim \mathcal{GP}(m(x), k(x, x'))$$

**变量含义**：
- $f$ — 待学的目标函数。
- $\mathcal{GP}$ — 高斯过程，本质是 "**任意有限点上联合服从多元正态分布**"。
- $m(x)$ — **均值函数**，即先验认为 $f$ 大致围绕的中心；常取 $m(x) = 0$。
- $k(x, x')$ — **协方差核函数**，描述输入 $x$ 与 $x'$ 上 $f$ 值的相关程度；越近的 $x$ 期望 $f$ 值越像。

**公式解读**：在没看到任何数据前，对函数 $f$ 的猜测是"均值 $m$ + 协方差 $k$ 决定的随机函数族"。$k$ 是 GP 的"**尺子**"，决定函数光滑性。

### 2.2 GP 后验 (核心)

观测数据 $D = \{(x_i, y_i)\}_{i=1}^N$ 后，GP 给出对新点 $x_*$ 的预测分布：

$$\mu_n(x_*) = \mathbf{k}_*^\top (\mathbf{K} + \sigma_n^2\mathbf{I})^{-1}\mathbf{y}$$

$$\sigma_n^2(x_*) = k(x_*, x_*) - \mathbf{k}_*^\top (\mathbf{K} + \sigma_n^2\mathbf{I})^{-1}\mathbf{k}_*$$

**变量含义**：
- $\mu_n(x_*)$ — 在新点 $x_*$ 处 $f$ 的 **后验均值** (最佳点估计)。
- $\sigma_n^2(x_*)$ — 后验 **方差** (不确定性)；越大表示越不知道。
- $\mathbf{y} = (y_1, ..., y_N)^\top \in \mathbb{R}^N$ — 已观测值列向量。
- $\mathbf{K} \in \mathbb{R}^{N \times N}$ — 训练点之间的协方差矩阵，$\mathbf{K}_{ij} = k(x_i, x_j)$。
- $\mathbf{k}_* \in \mathbb{R}^N$ — 新点 $x_*$ 与训练点的协方差，$(\mathbf{k}_*)_i = k(x_i, x_*)$。
- $\sigma_n^2$ — 观测噪声方差 (假设 $y_i = f(x_i) + \epsilon_i, \epsilon \sim \mathcal{N}(0, \sigma_n^2)$)。
- $\mathbf{I}$ — $N \times N$ 单位矩阵；加 $\sigma_n^2 \mathbf{I}$ 避免奇异。

**公式解读**：
- 均值 $\mu_n(x_*)$ 是 **训练数据 $\mathbf{y}$ 的加权和**，权重由 "**新点与训练点的相似度**" 决定 (核函数算)。即"哪个训练点离 $x_*$ 近，它的 $y$ 就权重大"。
- 方差 $\sigma_n^2(x_*)$ 反映"**离训练点远 = 越不确定**"。在训练点重合时方差很小，远离时趋近先验方差 $k(x_*, x_*)$。

**直觉**：GP 是"**有不确定度量化的智能插值**"。

### 2.3 Matérn 5/2 核 (常用核函数)

$$k_{\nu=5/2}(r) = \sigma_f^2 \left(1 + \sqrt{5}\,r + \tfrac{5 r^2}{3}\right)\exp(-\sqrt{5}\,r)$$

$$r = \sqrt{\textstyle \sum_{d=1}^D (x_d - x'_d)^2 / \ell_d^2}$$

**变量含义**：
- $r$ — 两输入点 $x, x'$ 的 **归一化距离** (无量纲)。
- $D$ — 输入维度。
- $\ell_d$ — 第 $d$ 维的 **长度尺度**；越大表示该维变化越平缓。
- $\sigma_f^2$ — 信号方差 (函数整体幅度)。

**公式解读**：
- 当 $r = 0$ ($x = x'$): $k = \sigma_f^2$ — 自己与自己最相似。
- 当 $r \to \infty$: $k \to 0$ — 相距远的点几乎不相关。
- $\nu = 5/2$ 表示函数 **二阶可微**，比 RBF 核 (无限可微) 更现实。

**直觉**：核函数是"**相似度尺子**"，距离近 → 相似度高。

### 2.4 GP 超参数学习

最大化 **边缘对数似然**：

$$\log p(\mathbf{y}|X) = -\tfrac{1}{2}\mathbf{y}^\top(\mathbf{K} + \sigma_n^2\mathbf{I})^{-1}\mathbf{y} - \tfrac{1}{2}\log|\mathbf{K} + \sigma_n^2\mathbf{I}| - \tfrac{N}{2}\log 2\pi$$

**变量含义**：
- $X = (x_1, ..., x_N)$ — 训练输入集合。
- $|\cdot|$ — 行列式。
- 三项分别是：**数据拟合项** + **复杂度惩罚项** + **常数**。

**公式解读**：选 $\theta = (\ell_d, \sigma_f, \sigma_n)$ 让训练数据出现概率最大。第二项 $\log|\mathbf{K}|$ 自动惩罚过复杂的核 (避免过拟合)。

**计算代价**：训练 $O(N^3)$；推理 $O(N^2)$。维度 $D \le 20$ 实用。

---

## 3. 神经网络代理 (本论文路线)

### 3.1 训练损失

$$L(\theta) = \frac{1}{N}\sum_{i=1}^N \|\hat{f}_\theta(x_i) - y_i\|_p^p + \lambda \|\theta\|_2^2$$

**变量含义**：
- $\theta$ — NN 全部权重和偏置。
- $\hat{f}_\theta(x_i)$ — 网络对输入 $x_i$ 的预测 (一次前向传播)。
- $y_i$ — 真实输出 (来自昂贵仿真)。
- $\|\cdot\|_p$ — $L^p$ 范数；$p = 1$ (绝对值, 抗离群点) 或 $p = 2$ (平方, 平滑)。
- $\lambda \|\theta\|_2^2$ — **L2 权重衰减正则**，防止 $\theta$ 过大导致过拟合。
- $\lambda \ge 0$ — 正则强度超参。

**公式解读**：让 NN 在训练集上预测 $y$，且参数不要过大。本论文 $p = 1$ (L1)，对偶尔的离群仿真值更鲁棒。

### 3.2 SGD/Adam 更新

$$\theta_{t+1} = \theta_t - \eta\,\nabla_\theta L(\theta_t)$$

**变量含义**：
- $\theta_t$ — 第 $t$ 步迭代时的参数。
- $\eta$ — 学习率 (通常 $10^{-3}$)。
- $\nabla_\theta L$ — 损失对参数的梯度，由自动微分给出。

**公式解读**：每步沿损失下降最快方向 (负梯度) 更新一点。Adam 在此基础上加自适应步长。

### 3.3 NN 代理的特点

| 性质 | 描述 |
|---|---|
| 维度无上限 | 输入可几千维 (vs GP $\le 20$) |
| 训练 | $O(N \cdot E \cdot |\theta|)$，$E$ 是 epoch 数 |
| 推理 | $O(|\theta|)$，毫秒级 |
| 不确定度 | 不直接给出 (除非 Bayesian NN / ensemble) |

---

## 4. 代理 + 优化的完整工作流

代理本身不是目的——**最终要用代理替代昂贵 $f^\star$ 完成下游任务**。

### 4.1 贝叶斯优化 (BO) 工作流

```
初始化: 随机抽 n_0 = 5d 个点 X_0, 评估得 D_0 = {(x_i, y_i)}
重复 t = 1, 2, ..., T:
    1) 用 D_{t-1} 训练 GP 代理: 得 (μ_t, σ_t)
    2) 选下一个采样点:
         x_t = argmax_x  α(x; μ_t, σ_t)
    3) 真实评估: y_t = f*(x_t)            # 昂贵, 只这一步慢
    4) D_t = D_{t-1} ∪ {(x_t, y_t)}
返回 D_T 中最优 (x*, y*)
```

### 4.2 EI 采集函数 (用得最多)

$$\text{EI}(x) = \mathbb{E}_{Y \sim p(f(x)|D)}\left[\max(y^* - Y, 0)\right]$$

由于 $Y \sim \mathcal{N}(\mu(x), \sigma^2(x))$，可解析积分：

$$\boxed{\text{EI}(x) = (y^* - \mu(x))\Phi(z) + \sigma(x)\phi(z), \quad z = \frac{y^* - \mu(x)}{\sigma(x)}}$$

**变量含义**：
- $y^* = \min_i y_i$ — 当前已知最优值 (假设最小化问题)。
- $\mu(x), \sigma(x)$ — GP 在点 $x$ 的后验均值和标准差。
- $z$ — 标准化的"改进量"；表示"如果均值是真值，相对当前最优提升了几个标准差"。
- $\Phi(z)$ — 标准正态 CDF (累积分布函数)，即 $z \sim \mathcal{N}(0,1)$ 时 $P(Z \le z)$。
- $\phi(z)$ — 标准正态 PDF (密度函数)，$\phi(z) = \frac{1}{\sqrt{2\pi}} e^{-z^2/2}$。

**公式解读**：
- **第 1 项 $(y^*-\mu(x))\Phi(z)$**：均值改进 × 改进概率 = **利用 (exploitation)**。
- **第 2 项 $\sigma(x)\phi(z)$**：不确定性放大利用项的"惊喜空间" = **探索 (exploration)**。
- 二者自动平衡：$\sigma$ 大处重探索；$\mu$ 远低于 $y^*$ 处重利用。

**直觉**：EI 选 "**最有可能产生大改进的点**"。

### 4.3 神经代理 + 梯度反向设计 (本论文 Yi'25 路线)

```
─── 离线阶段 ─────────────────────────────
1) 在设计空间 LHS 采样 N=80,000 个 (x_i, k_i)
2) 跑 FEM 仿真得 y_i = f*(x_i, k_i)         # 数日, 一次性
3) 训神经网络代理 f̂_θ:
     L_train(θ) = Σ_i ‖f̂_θ(x_i, k_i) - y_i‖_1
   用 Adam 训 1 hr (1 GPU)

─── 在线优化阶段 ─────────────────────────
设损失 L_opt(p, k) = w1·‖f̂_θ(p, k)‖ + w2·penalty(...)
重复 t = 1..T:
    g = ∇_k L_opt(p, k_t)                   # PyTorch autograd 通过 NN
    k_{t+1} = k_t - η · g                   # 梯度下降
返回 k* = k_T
```

**关键**：因为 $\hat{f}_\theta$ 是 NN，$\nabla_k$ 由 PyTorch 自动微分提供，**绕过原始 FEM 不可微的梯度坑**。

### 4.4 主动学习 (Active Learning)

当数据生成贵到无法预生成 80k 时：

```
D ← 初始小集合 (n_0 个点)
重复:
    1) 训 f̂_θ on D
    2) 选最有价值的下一个点:
         x_next = argmax_x  σ_θ(x)        # 最不确定处
    3) y_next = f*(x_next)                # 真实评估
    4) D ← D ∪ {(x_next, y_next)}
直到代理足够准
```

---

## 5. 代理梯度 vs 物理梯度 (本论文核心动机)

为何不直接对真实模型 $f^\star$ 求梯度？

**真实物理梯度问题**：
- **接触不连续**：$\partial f^\star / \partial x$ 在接触切换处跳变，定义不存在。
- **塑性 / 锐边**：数值梯度被放大或归零。
- **混沌**：长 rollout 下梯度爆炸。

**代理梯度优势**：
- **平滑性**：NN 的 ReLU/Tanh 激活让 $\nabla_x \hat{f}_\theta$ 处处定义且有限。
- **数据驱动正则**：训练数据已"平均掉"高频噪声。
- **快速**：每步反传 ~1 ms。

**实验对比** (本论文 Fig 4d):
- 直接 FEM 可微梯度：每迭代 ~10 s，梯度噪声大。
- NN 代理梯度：每迭代 ~10 ms (1000× 加速)，梯度平滑。

---

## 6. 验证：代理 ≠ 真实

代理上线后必须回头验证：

1. **Held-out test set**：留 20% 数据不训，测 $\|\hat{f}_\theta - f^\star\|$。
2. **Optimization-target validation**：把代理找到的 $x^*$ 拿回真实模型重测。
3. **OOD 检测**：若 $x^*$ 远离训练分布，警惕代理失真。
4. **不确定度过滤**：在 GP/Bayesian 代理中拒绝高 $\sigma(x)$ 的预测。

本论文 (Yi'25) 在 Fig 4 用真实 FEM 验证 $L(k^*)$，并报告 OOD (KIT/EGAD) 性能——这就是上述第 2、3 步。

---

## 7. 关联概念

- [[神经物理模拟器 Neural Physics]]：物理专用代理。
- [[可微分模拟 Differentiable Simulation]]：原生物理 + AD 的另一路线。
- [[../02-方法分类/GNN 图神经网络模拟器]]：图结构代理。
- [[../02-方法分类/采样优化方法]]：BO / CMA-ES。
- [[双层优化 Bi-level Optimization]]：代理常加速外层。

## 8. 关联文献

- [[../03-参考文献/Ref-22-Calandra-2016-BO]]：GP 代理 + BO 步态调优。
- [[../03-参考文献/Ref-23-Snoek-2012-PracticalBO]]：GP + Matérn 5/2 + EI。
- [[../03-参考文献/Ref-28-Pfaff-2021-MeshGraphNets]]：GNN 代理。
- [[../03-参考文献/Ref-20-Allen-2022-InverseDesignGNS]]：代理 + 反设计。
- [[../03-参考文献/Ref-45-Son-2023-CollisionNet]]：碰撞检测 NN 代理。
- [[../03-参考文献/Ref-07-He-2024-MORPH]]：硬件代理用于 RL。

---

*← 返回 [[../00-总览/主索引 MOC]]*
