# [24] Hansen, Müller & Koumoutsakos 2003 · CMA-ES

> **原文**: N. Hansen, S. D. Müller, P. Koumoutsakos. *Reducing the time complexity of the derandomized evolution strategy with covariance matrix adaptation (CMA-ES).* **Evolutionary Computation**, 11(1):1-18, 2003.
> **译名**: 《降低协方差矩阵自适应去随机化进化策略 (CMA-ES) 的时间复杂度》 — *进化计算*, 2003.

`#文献-方法` `#进化策略` `#里程碑`

---

## 1 · 研究问题

无梯度优化:
$$x^* = \arg\min_{x \in \mathbb{R}^n} f(x)$$

**变量含义**：
- $x \in \mathbb{R}^n$ — $n$ 维向量参数。
- $f$ — 黑盒函数 (可能非凸、可能噪声)。

经典 (1+1)-ES 用各向同性高斯采样, 收敛极慢 (远场到近场尺度不能自适应)。Hansen 等的目标:

> **让 ES 自适应学习问题的局部几何 (尺度 + 旋转)**: 维护一个协方差矩阵 $\mathbf{C} \in \mathbb{R}^{n \times n}$, 在采样过程中更新它来反映"**哪些方向是好方向**", 大幅提高收敛速度。

CMA-ES = **C**ovariance **M**atrix **A**daptation **E**volution **S**trategy。

---

## 2 · 算法核心 — 维护三个量

CMA-ES 维护三个核心量:

- $\mathbf{m} \in \mathbb{R}^n$ — **均值向量**: 当前搜索中心 (类似最优估计)。
- $\sigma > 0$ — **步长 (标量)**: 全局尺度 (越大搜索越广)。
- $\mathbf{C} \in \mathbb{R}^{n \times n}$ — **协方差矩阵**: 采样分布的形状 / 方向 (反映哪些方向是好方向)。

每代 (generation) $g$ 执行 6 步:

### 2.1 采样种群 ($\lambda$ 个候选)

$$\mathbf{x}_k^{(g+1)} = \mathbf{m}^{(g)} + \sigma^{(g)} \cdot \mathbf{B}^{(g)} \mathbf{D}^{(g)} \mathbf{z}_k$$

**变量含义**：
- $k = 1, ..., \lambda$ — 第 $k$ 个候选。
- $\mathbf{x}_k$ — 第 $k$ 个采样点 ($n$ 维)。
- $\mathbf{z}_k \sim \mathcal{N}(\mathbf{0}, \mathbf{I})$ — 标准正态向量 ($n$ 维, 各分量独立)。
- $\mathbf{C} = \mathbf{B}\mathbf{D}^2\mathbf{B}^\top$ — 协方差矩阵的特征分解。
- $\mathbf{B} \in \mathbb{R}^{n \times n}$ — **正交矩阵** (列是 $\mathbf{C}$ 的特征向量)。
- $\mathbf{D} \in \mathbb{R}^{n \times n}$ — **对角矩阵** (对角元是 $\mathbf{C}$ 的特征值平方根)。

**公式解读**：
- $\mathbf{B}\mathbf{D}\mathbf{z}_k$ 把单位高斯旋转 + 拉伸到 $\mathbf{C}$ 形状。
- $\sigma$ 缩放整体尺度。
- $\mathbf{m}$ 平移到当前最优中心。
- 最终 $\mathbf{x}_k \sim \mathcal{N}(\mathbf{m}, \sigma^2 \mathbf{C})$。

### 2.2 评估 + 排序

按 $f(\mathbf{x}_k)$ 升序: $\mathbf{x}_{(1)}, \mathbf{x}_{(2)}, ..., \mathbf{x}_{(\lambda)}$。

下标 $(i)$ 表示第 $i$ 个最好的样本。

### 2.3 选择 $\mu$ 个精英 + 加权均值

权重 $w_1 \ge w_2 \ge ... \ge w_\mu > 0$, $\sum_{i=1}^\mu w_i = 1$:

$$\mathbf{m}^{(g+1)} = \sum_{i=1}^\mu w_i \mathbf{x}_{(i)}^{(g+1)}$$

**变量含义**：
- $\mu \le \lambda$ — 精英数 (典型 $\mu = \lambda/2$)。
- $w_i$ — 第 $i$ 名权重 (好的样本权重高)。

**公式解读**：新均值是 $\mu$ 个最好样本的加权平均, 让搜索中心向好方向移动。

**有效选择数**:
$$\mu_\text{eff} = \frac{1}{\sum_{i=1}^\mu w_i^2}$$

**变量含义**：$\mu_\text{eff}$ 反映 "**有效精英数**": 若所有 $w_i$ 相等则 $\mu_\text{eff} = \mu$; 若集中在前几个则 $\mu_\text{eff} < \mu$。

### 2.4 进化路径更新

#### 步长进化路径 (CSA)

$$\mathbf{p}_\sigma^{(g+1)} = (1 - c_\sigma)\,\mathbf{p}_\sigma^{(g)} + \sqrt{c_\sigma(2 - c_\sigma)\mu_\text{eff}}\;\mathbf{C}^{-1/2}\,\frac{\mathbf{m}^{(g+1)} - \mathbf{m}^{(g)}}{\sigma^{(g)}}$$

**变量含义**：
- $\mathbf{p}_\sigma \in \mathbb{R}^n$ — 步长进化路径 (累积均值移动方向)。
- $c_\sigma$ — 步长路径学习率 (典型 $\sim 0.1-0.3$)。
- $\sqrt{c_\sigma(2-c_\sigma)\mu_\text{eff}}$ — 归一化系数, 让 $\mathbf{p}_\sigma$ 在原假设下服从 $\mathcal{N}(\mathbf{0}, \mathbf{I})$。
- $\mathbf{C}^{-1/2}$ — 协方差矩阵的"逆平方根", 把空间标准化。
- $(\mathbf{m}^{(g+1)} - \mathbf{m}^{(g)})/\sigma$ — 本代均值移动 (标准化)。

**公式解读**：
- 第 1 项 $(1 - c_\sigma)\mathbf{p}_\sigma$ — 旧路径"衰减"。
- 第 2 项 — 加入本代均值移动方向。
- 整体: $\mathbf{p}_\sigma$ 累积"**最近哪些方向反复被选**" 的信息。

#### 协方差进化路径

$$\mathbf{p}_c^{(g+1)} = (1 - c_c)\,\mathbf{p}_c^{(g)} + \sqrt{c_c(2 - c_c)\mu_\text{eff}}\;\frac{\mathbf{m}^{(g+1)} - \mathbf{m}^{(g)}}{\sigma^{(g)}}$$

类似 $\mathbf{p}_\sigma$, 但不做 $\mathbf{C}^{-1/2}$ 标准化, 用于协方差更新。

### 2.5 协方差更新 (Rank-1 + Rank-$\mu$)

$$\boxed{\mathbf{C}^{(g+1)} = (1 - c_1 - c_\mu)\,\mathbf{C}^{(g)} + c_1\,\mathbf{p}_c^{(g+1)}\mathbf{p}_c^{(g+1)\top} + c_\mu \sum_{i=1}^\mu w_i\,\mathbf{y}_{(i)}\mathbf{y}_{(i)}^\top}$$

**变量含义**：
- $c_1$ — Rank-1 学习率 (典型 $\sim 0.01-0.1$)。
- $c_\mu$ — Rank-$\mu$ 学习率。
- $1 - c_1 - c_\mu$ — 旧协方差衰减因子。
- $\mathbf{y}_{(i)} = (\mathbf{x}_{(i)} - \mathbf{m}^{(g)})/\sigma^{(g)}$ — 第 $i$ 名样本相对均值的偏移 (标准化)。
- $\mathbf{p}_c \mathbf{p}_c^\top$ — **rank-1 矩阵** (秩 1)。
- $\sum w_i \mathbf{y}_{(i)} \mathbf{y}_{(i)}^\top$ — **rank-$\mu$ 矩阵** (秩最多 $\mu$)。

**公式解读** (三项加权和):
- **第 1 项** $(1-c_1-c_\mu)\mathbf{C}$ — 保留旧协方差 (历史信息)。
- **第 2 项** $c_1 \mathbf{p}_c\mathbf{p}_c^\top$ (Rank-1) — 利用 **进化路径** 强化"持续好方向"。
- **第 3 项** $c_\mu \sum w_i \mathbf{y}_{(i)}\mathbf{y}_{(i)}^\top$ (Rank-$\mu$) — 利用 **本代精英分布**。

整体: $\mathbf{C}$ 自适应学习问题 **局部 Hessian 信息** (即损失景观的局部形状)。

### 2.6 步长更新 (CSA)

$$\boxed{\sigma^{(g+1)} = \sigma^{(g)} \exp\!\left(\frac{c_\sigma}{d_\sigma}\left(\frac{\|\mathbf{p}_\sigma^{(g+1)}\|}{\mathbb{E}\|\mathcal{N}(\mathbf{0}, \mathbf{I})\|} - 1\right)\right)}$$

**变量含义**：
- $d_\sigma$ — 步长阻尼系数 (典型 $\sim 1$)。
- $\|\mathbf{p}_\sigma\|$ — 步长进化路径长度。
- $\mathbb{E}\|\mathcal{N}(\mathbf{0}, \mathbf{I})\| = \sqrt{n}\,(1 - \tfrac{1}{4n} + \tfrac{1}{21n^2})$ — 标准正态向量长度的期望 ($n$ 维)。

**公式解读**：
- 比较 $\|\mathbf{p}_\sigma\|$ 与"在无信息情况下" 的期望长度。
- $\|\mathbf{p}_\sigma\| > \mathbb{E}\|\mathcal{N}\|$: 进化路径长度异常大 → 走得太一致 → **加大 $\sigma$** (探索更广)。
- $\|\mathbf{p}_\sigma\| < \mathbb{E}\|\mathcal{N}\|$: 走得太抖 → **减小 $\sigma$** (聚焦)。

直觉: 自动平衡探索/利用的尺度。

---

## 3 · 默认参数

Hansen 提供"几乎不需要调"的默认值:
- $\lambda = 4 + \lfloor 3 \ln n \rfloor$
- $\mu = \lfloor \lambda / 2 \rfloor$
- $w_i = \frac{\ln(\mu + 1) - \ln i}{\sum_{j=1}^\mu (\ln(\mu+1) - \ln j)}$
- $c_\sigma = (\mu_\text{eff} + 2) / (n + \mu_\text{eff} + 5)$
- $d_\sigma = 1 + 2 \max(0, \sqrt{(\mu_\text{eff}-1)/(n+1)} - 1) + c_\sigma$
- $c_c = (4 + \mu_\text{eff}/n) / (n + 4 + 2\mu_\text{eff}/n)$
- $c_1 = 2 / ((n+1.3)^2 + \mu_\text{eff})$
- $c_\mu = \min(1 - c_1, 2(\mu_\text{eff} - 2 + 1/\mu_\text{eff})/((n+2)^2 + \mu_\text{eff}))$

这些公式经过大量基准测试得出 "**几乎最优**"。

---

## 4 · 完整算法伪代码

```
─── Algorithm: CMA-ES ─────────────────────
输入: 目标 f, 维度 n, 初值 m_0, 步长 σ_0
初始化: C ← I; p_σ ← 0; p_c ← 0; g ← 0
重复直到收敛:
    # 1. 采样
    特征分解 C = B D² Bᵀ
    for k = 1..λ:
        z_k ~ N(0, I)
        x_k = m + σ·B·D·z_k
    # 2. 评估 & 排序
    f_k = f(x_k);  按 f_k 升序排列
    # 3. 加权均值
    m_new = Σ_i w_i · x_{(i)}
    # 4. 进化路径
    p_σ ← (1-c_σ) p_σ + √(c_σ(2-c_σ)μ_eff) · C^{-1/2}(m_new - m)/σ
    p_c ← (1-c_c) p_c + √(c_c(2-c_c)μ_eff) · (m_new - m)/σ
    # 5. 协方差更新
    C ← (1 - c_1 - c_μ)C + c_1 p_c p_cᵀ + c_μ Σ w_i y_{(i)} y_{(i)}ᵀ
    # 6. 步长更新
    σ ← σ · exp((c_σ/d_σ)(‖p_σ‖/E‖N(0,I)‖ - 1))
    m ← m_new
    g ← g + 1
返回 m (最佳估计)
```

---

## 5 · 复杂度

每代:
- 采样 + 评估: $O(\lambda \cdot \text{evalcost})$。
- 协方差更新: $O(n^2 \mu)$ (rank-$\mu$ 项)。
- 特征分解 (每 $1/(c_1 + c_\mu)/n$ 代一次): $O(n^3)$。

总: $O(\text{generations} \cdot n^2 \lambda + \text{evalcost})$。

---

## 6 · CMA-ES 的关键优势

1. **不需梯度** — 黑盒友好。
2. **自适应几何** — 自动学问题的尺度和旋转, 不需用户预设。
3. **几乎无需调参** — 默认参数在大部分基准上有效。
4. **平移/旋转不变** — 性能不受坐标系选择影响。
5. **理论保证** — 在凸二次函数上线性收敛。

---

## 7 · 实验

### 7.1 标准测试函数
- Sphere: $f(x) = \sum x_i^2$
- Rosenbrock: $f(x) = \sum (1-x_i)^2 + 100(x_{i+1}-x_i^2)^2$
- Rastrigin (多模态): $f(x) = 10n + \sum (x_i^2 - 10\cos(2\pi x_i))$

CMA-ES 在 $n \le 100$ 维上 **比纯 GA、PSO、(1+1)-ES** 都快 1-3 个数量级。

### 7.2 实际应用领域
- 机器人步态参数 ($n = 10-50$)。
- 神经网络超参 / 权重 (NES 变体)。
- 工程结构优化。
- 量化金融策略。

---

## 8 · 关键贡献

1. **Rank-$\mu$ + Rank-1 协方差更新**: 让 ES 学局部 Hessian 信息。
2. **进化路径累积**: 减小代际方差, 加速学习几何。
3. **CSA 步长适配**: 自动平衡探索/利用尺度。
4. **默认参数 + 数学证明**: 不需要用户调。

---

## 9 · 局限

1. **高维 ($n > 200$) 受限**: $\mathbf{C}$ 矩阵 $O(n^2)$ 内存。
2. **多模态可能局部极小**: Restart-CMA-ES (IPOP-CMA, BIPOP-CMA) 缓解。
3. **样本效率低于 BO**: 但维度高时 BO 更糟。

---

## 10 · 历史影响

- BBOB / COCO 基准上长期排名第一的黑盒优化算法。
- 工业广泛使用 (机器学习超参、机器人、化学)。
- 衍生 sep-CMA-ES, NES, CMA-ME 等。

---

## 11 · 与本论文 (Yi'25) 的关系

本论文综述提到 CMA-ES 作为 co-design 中 "**采样优化**" 代表。本论文 28 维 + 多物体联合 + FEM 评估贵 → CMA-ES 需 $10^3-10^4$ 次评估太慢, 所以转用神经代理 + 梯度。

---

*← [[_参考文献索引]]*
