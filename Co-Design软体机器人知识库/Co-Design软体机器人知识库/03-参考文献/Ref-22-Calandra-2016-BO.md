# [22] Calandra et al. 2016 · 双足步态贝叶斯优化实验比较

> **原文**: R. Calandra, A. Seyfarth, J. Peters, M. P. Deisenroth. *Bayesian optimization for learning gaits under uncertainty: An experimental comparison on a dynamic bipedal walker.* **Annals of Mathematics and AI**, 76:5-23, 2016.
> **译名**: 《不确定性下学习步态的贝叶斯优化：动态双足行走器实验比较》 — *数学与人工智能年刊*, 2016.

`#文献-方法` `#贝叶斯优化` `#机器人学习`

---

## 1 · 研究问题

双足机器人步态调参一直是 **经验活**: 频率、摆幅、踝部角度等十几个参数, 每改一组都要做一次真实实验, 又费时又危险 (机器人摔倒可能损坏)。Calandra 等的目标:

> **系统比较多种调参方法** (网格搜索 / 随机搜索 / 进化策略 / 贝叶斯优化) 在 **同一动态双足行走器** 上的样本效率, 量化哪种方法最适合"评估贵且有噪声" 的机器人学习场景。

---

## 2 · 形式化

### 2.1 步态参数空间

$$\mathbf{x} \in \mathbb{R}^d, \quad d = 8 \sim 12$$

**变量含义**：
- $\mathbf{x}$ — 步态参数向量。
- 各分量例:
  - $f_\text{step}$ (Hz) — 步频, $\in [0.5, 3]$。
  - $A_\text{hip}$ (rad) — 髋摆幅, $\in [0.1, 0.5]$。
  - $\alpha_\text{knee}$ (rad) — 膝弯曲角度。
  - $\tau_\text{ankle}$ (N·m) — 踝扭矩偏置。
  - $\phi_\text{phase}$ (rad) — 相位差。

### 2.2 评估函数 (含噪声)

$$y = \bar{f}(\mathbf{x}) + \epsilon, \quad \epsilon \sim \mathcal{N}(0, \sigma^2)$$

**变量含义**：
- $\bar{f}(\mathbf{x})$ — 真实平均表现 (无噪声)。
- $\epsilon$ — 实验测量噪声 (重复性误差)。
- $\sigma^2$ — 噪声方差。

每次评估实际值 $y$:
1. 让机器人按 $\mathbf{x}$ 走 5 米。
2. 记录 (前进速度, 能量消耗, 是否摔倒)。

综合奖励:
$$y = w_1 v - w_2 e - w_3 \mathbb{1}[\text{fall}]$$

**变量含义**：
- $v$ (m/s) — 前进速度。
- $e$ (J) — 能耗。
- $\mathbb{1}[\text{fall}]$ — 摔倒指示函数 (1=摔倒, 0=未摔)。
- $w_1, w_2, w_3$ — 权重 (例 $1, 0.1, 100$)。

**公式解读**：奖励是"速度高 - 能耗低 - 不摔倒" 的加权组合, 后两项为惩罚。

### 2.3 优化目标

$$\mathbf{x}^* = \arg\max_{\mathbf{x} \in \mathcal{X}} \mathbb{E}\left[y(\mathbf{x})\right]$$

**变量含义**：
- $\mathcal{X}$ — 步态参数可行域 (各参数上下界)。
- $\mathbb{E}$ — 对实验噪声的期望 (即"重复多次平均")。

每次评估 ~30 秒 + 人工设置 ~ 几分钟。

---

## 3 · 比较的方法

### 3.1 网格搜索 (Grid)

每参数 5 个点, 全枚举: $5^{10} = 10^7$ 不可行 → 仅低维实测。

### 3.2 随机搜索 (Random)

$$\mathbf{x}_i \sim \text{Uniform}(\mathcal{X})$$

每次从可行域均匀抽样, 取最佳。

### 3.3 进化策略 (CMA-ES, [[Ref-24-Hansen-2003-CMA-ES]])

维护协方差 + 种群迭代。详见 [[Ref-24-Hansen-2003-CMA-ES]]。

### 3.4 贝叶斯优化 (BO)

GP 代理 + Matérn 5/2 核 (Snoek 2012, [[Ref-23-Snoek-2012-PracticalBO]])。

EI 采集函数:
$$\text{EI}(\mathbf{x}) = (y^* - \mu(\mathbf{x}))\Phi(z) + \sigma(\mathbf{x}) \phi(z),\quad z = \frac{y^* - \mu(\mathbf{x})}{\sigma(\mathbf{x})}$$

**变量含义** (与 [[Ref-23-Snoek-2012-PracticalBO]] 同):
- $y^*$ — 当前最佳观测。
- $\mu(\mathbf{x}), \sigma(\mathbf{x})$ — GP 后验均值/标准差。
- $z$ — 标准化改进量。
- $\Phi, \phi$ — 标准正态 CDF / PDF。

每次选 $\arg\max \text{EI}$ 评估。

### 3.5 BO 处理噪声

观测噪声 $\sigma_n^2$ 通过 GP 的 likelihood 项内置:
$$\mathbf{K}_n = \mathbf{K} + \sigma_n^2 \mathbf{I}$$

**变量含义**：
- $\mathbf{K}_n$ — 含噪协方差矩阵。
- $\mathbf{K}$ — 训练点协方差。
- $\sigma_n^2$ — 噪声方差 (从训练数据估计)。
- $\mathbf{I}$ — 单位矩阵。

**公式解读**：在协方差矩阵对角加一项噪声方差, 让 GP 不要"硬拟合"噪声数据。

GP 后验在噪声下仍 well-defined。

---

## 4 · 实验设置

### 4.1 平台
- 真实双足机器人 (Daedalus, Calandra 自制)。
- 8-12 个步态参数。
- 每参数组评估 ~30 s。

### 4.2 计算预算

固定预算 100 次评估, 比较各方法找到的最优 $\mathbf{x}^*$。

### 4.3 重复 + 噪声

每方法重复 10 次随机种子, 报告平均 ± 方差。

---

## 5 · 量化结果

| 方法 | 最优速度 (m/s) | 找到所需评估次数 |
|---|---|---|
| 网格搜索 (低维) | 0.45 | $5^d$ 太多 |
| 随机搜索 | 0.55 ± 0.10 | 100 |
| CMA-ES | 0.65 ± 0.08 | 80 |
| **BO (GP + EI)** | **0.78 ± 0.04** | **40** |

**关键发现**:
- BO 比随机快 2.5 倍 (40 vs 100 评估)。
- BO 比 CMA-ES 快 2 倍。
- BO 方差更小 (鲁棒性高)。

---

## 6 · 关键贡献

1. **真实机器人 BO**: 第一篇在真实双足机器人上系统比较 BO 与传统方法。
2. **量化 BO 优势**: 样本效率 2-5× 提升。
3. **噪声处理**: GP 内置噪声处理被验证有效。
4. **方法选择指南**: 为机器人调参领域提供决策依据。

---

## 7 · 局限

1. **维度受限**: 测试 $d \le 12$, 高维 BO 退化。
2. **预算固定**: 没有 anytime 性能曲线。
3. **未与现代深度方法对比** (RL, evolutionary deep)。

---

## 8 · 历史影响

- 推广 BO 在机器人学的应用。
- 启发后续 "**机器人步态 BO**" 系列工作。
- 是机器人学习中 BO 的奠基应用之一。

---

## 9 · 与本论文 (Yi'25) 的关系

本论文综述把 BO 列入 **"采样优化方法"** 类。本论文 28 维设计空间 + 多物体联合 → 已超出 BO 甜区 (≤20 维), 转用神经代理 + 梯度。

---

*← [[_参考文献索引]]*
