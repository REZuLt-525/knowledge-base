# [27] Xu, Ha & Song 2024 · 动力学引导扩散模型

> **原文**: X. Xu, H. Ha, S. Song. *Dynamics-guided diffusion model for robot manipulator design.* arXiv:2402.15038, 2024.
> **译名**: 《用于机器人操作器设计的动力学引导扩散模型》 — arXiv 预印本, 2024.

`#文献-方法` `#生成式设计` `#扩散模型`

---

## 1 · 研究问题

[[Ref-26-Ha-2021-Fit2Form|Fit2Form'21]] 用 GAN 生成夹爪, 但 GAN **多样性差 + 训练不稳**。Columbia Shuran Song 团队跟进:

> 用 **扩散模型 (Diffusion Model)** 替代 GAN, 引入 **动力学引导 (dynamics guidance)**, 实现 **多样化 + 高质量** 的机器人操作器生成。

应用: 不仅夹爪, 更广泛的末端执行器 (拧螺丝、打蛋、剪线等专用工具)。

---

## 2 · 扩散模型基础

### 2.1 正向加噪过程

定义 $T$ 步加噪 schedule $\{\beta_t\}_{t=1}^T$:
$$q(x_t | x_{t-1}) = \mathcal{N}(x_t;\; \sqrt{1-\beta_t}\,x_{t-1},\; \beta_t I)$$

闭式表达 (用 $\bar{\alpha}_t = \prod_{s=1}^t (1 - \beta_s)$):
$$x_t = \sqrt{\bar{\alpha}_t}\,x_0 + \sqrt{1-\bar{\alpha}_t}\,\epsilon,\quad \epsilon \sim \mathcal{N}(0, I)$$

### 2.2 反向去噪过程

学一个网络 $\epsilon_\theta(x_t, t)$ 预测加入的噪声:
$$L_\text{train} = \mathbb{E}_{x_0, \epsilon, t}\;\;\big\|\epsilon - \epsilon_\theta(x_t, t)\big\|^2$$

采样 (反向):
$$x_{t-1} = \frac{1}{\sqrt{\alpha_t}}\Big(x_t - \frac{1-\alpha_t}{\sqrt{1-\bar{\alpha}_t}}\epsilon_\theta(x_t, t)\Big) + \sigma_t z$$
$z \sim \mathcal{N}(0, I)$。

### 2.3 条件生成

用 **classifier-free guidance** (CFG) 让生成模型受任务 $d$ 条件:
$$\epsilon_\text{guided} = \epsilon_\theta(x_t, t, d) + w\big(\epsilon_\theta(x_t, t, d) - \epsilon_\theta(x_t, t)\big)$$

$w > 0$ 增强条件影响。

---

## 3 · 动力学引导 (核心创新)

### 3.1 动力学奖励函数

定义动力学评估 $R_\text{dyn}(x; d)$: 在仿真中测设计 $x$ 完成动作 $d$ 的能力 (例如能否抓起 100g 的杯子)。

### 3.2 引导扩散

在反向去噪过程中, 把动力学奖励的梯度作为额外引导:
$$\hat{x}_{t-1} = \mu_\theta(x_t, t, d) + s \cdot \boldsymbol{\sigma}_t \cdot \nabla_x \log R_\text{dyn}(x_t; d)\bigg|_{x = \hat{x}_0(x_t)}$$

其中:
- $s$: 引导强度。
- $\hat{x}_0(x_t) = \frac{1}{\sqrt{\bar{\alpha}_t}}(x_t - \sqrt{1-\bar{\alpha}_t}\epsilon_\theta)$: 当前去噪估计。
- $\nabla_x \log R_\text{dyn}$: 通过可微仿真或代理给出。

直观: 反向去噪过程不仅追求"长得像好夹爪", 还追求"实际能完成动作"。

### 3.3 引导计算

由于 $R_\text{dyn}$ 通过仿真给出 (本身可能不可微), 用以下方式:
1. **可微仿真** (DiffTaichi-like): 直接 AD。
2. **代理 NN** (类似本论文 Yi'25): 训 NN 预测 $R_\text{dyn}$, 通过 NN 反传。
3. **REINFORCE**: 基于 score function。

---

## 4 · 任务条件

任务 $d$ 编码动力学需求, 例:
- "抓重物 100g"。
- "打开纸箱盖" (推力 + 角度)。
- "插入扁平孔" (姿态精确)。

不同 $d$ 下生成的设计差异巨大 (论文展示)。

---

## 5 · 完整算法

```
─── Algorithm: Dynamics-Guided Diffusion ────
─ 阶段 1: 训练扩散模型 ────────────────────
1) 收集 (设计, 动力学) 数据集
2) 训练 ε_θ:
    L = ‖ε - ε_θ(x_t, t, d)‖²

─ 阶段 2: 训练动力学评估器 ──────────────────
学 NN 代理 R̂_dyn(x; d) 近似仿真奖励

─ 阶段 3: 引导生成 (推理时) ────────────────
输入: 任务 d
x_T ~ N(0, I)
for t = T..1:
    ε_pred = ε_θ(x_t, t, d)
    x̂_0 = (x_t - sqrt(1-ᾱ_t)·ε_pred) / sqrt(ᾱ_t)
    g = ∇_x log R̂_dyn(x̂_0; d)
    μ = pre-calculated mean
    x_{t-1} = μ + s·σ_t·g + σ_t·z
返回 x_0 (生成设计)
```

---

## 6 · 实验

### 6.1 任务多样性
- 物体抓取 (类似 Fit2Form)。
- 工具使用 (开瓶、撬动、剪切)。
- 多步操作 (拧螺丝、装配)。

### 6.2 多样性比较

| 方法 | 单 prompt 生成多样性 (FID) |
|---|---|
| GAN (Fit2Form) | 高 mode collapse |
| **扩散 (本文)** | **多样性显著高** |

### 6.3 任务表现
- 引导前: 50% 任务成功率。
- 引导后: 80% 任务成功率。

---

## 7 · 关键贡献

1. **首篇扩散模型 + 机器人设计**: 引入扩散到机器人形态生成。
2. **动力学引导**: 让生成模型考虑功能而非仅外观。
3. **多任务条件化**: 同一模型服务多种任务。

---

## 8 · 局限

1. **采样慢**: 扩散需 $T \sim 100-1000$ 步去噪。
2. **依赖动力学评估**: 仿真或代理不准则引导失效。
3. **可制造后处理**: 体素 / 隐式表示需转 STL。

---

## 9 · 历史影响

- 把 LLM/Stable Diffusion 时代的生成模型范式引入机器人设计。
- 与 [[Ref-26-Ha-2021-Fit2Form|Fit2Form]] 形成 GAN → Diffusion 的进化。
- 启发后续基础模型 + 机器人设计研究。

---

## 10 · 与本论文 (Yi'25) 的关系

两者都用 NN + 物理感知做 co-design, 但路径完全不同:
- 扩散模型: 一次性生成几何 (生成式, 探索新形态)。
- 本论文: 迭代优化刚度 (优化式, 精调已知形态)。

未来融合: 扩散生成几何骨架 → 本论文优化局部刚度。

---

*← [[_参考文献索引]]*
