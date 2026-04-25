# [24] Hansen, Müller & Koumoutsakos 2003 · CMA-ES

> **引用**: N. Hansen, S. D. Müller, P. Koumoutsakos. *Reducing the time complexity of the derandomized evolution strategy with covariance matrix adaptation (CMA-ES).* **Evolutionary Computation**, 11(1):1-18, 2003.

`#文献-方法` `#进化策略` `#里程碑`

---

## 🎯 这篇论文想做什么

进化策略 (ES) 把"父代+突变" 用于连续优化，但传统 ES 收敛慢、各向异性的问题没解决。Hansen 等的目标：

> 给定一个 **黑盒函数** $f: \mathbb{R}^n \to \mathbb{R}$，**在不需梯度** 的情况下，用 **尽可能少的函数评估次数** 找到 $\arg\min f$。

让 ES "**自动学习**" 哪些方向是好方向 → 用一个 **协方差矩阵 $\mathbf{C}$** 编码这一信息。

---

## 🛠️ 实现路径 (算法步骤)

CMA-ES 维护三个核心量：
- 均值 $\mathbf{m} \in \mathbb{R}^n$ —— 当前搜索中心
- 步长 $\sigma > 0$ —— 全局尺度
- 协方差 $\mathbf{C} \in \mathbb{R}^{n\times n}$ —— 形状/方向

每代 (generation) 执行 6 步：

**(1) 采样种群** ($\lambda$ 个候选)：
$$\mathbf{x}_k = \mathbf{m} + \sigma \,\mathbf{B}\mathbf{D}\,\mathbf{z}_k,\quad \mathbf{z}_k \sim \mathcal{N}(\mathbf{0}, \mathbf{I})$$
其中 $\mathbf{C} = \mathbf{B} \mathbf{D}^2 \mathbf{B}^\top$ 是特征分解。

**(2) 评估**：算 $f(\mathbf{x}_1), \ldots, f(\mathbf{x}_\lambda)$，按值排序。

**(3) 选择前 $\mu$ 个 (精英)**：取 weights $w_i$ 加权均值
$$\mathbf{m}_\text{new} = \sum_{i=1}^{\mu} w_i \,\mathbf{x}_{(i)}$$

**(4) 更新进化路径** (path)：
$$\mathbf{p}_\sigma \leftarrow (1-c_\sigma)\mathbf{p}_\sigma + \sqrt{c_\sigma(2-c_\sigma)\mu_\text{eff}}\,\mathbf{C}^{-1/2}\frac{\mathbf{m}_\text{new} - \mathbf{m}}{\sigma}$$
$$\mathbf{p}_c \leftarrow (1-c_c)\mathbf{p}_c + \sqrt{c_c(2-c_c)\mu_\text{eff}}\,\frac{\mathbf{m}_\text{new}-\mathbf{m}}{\sigma}$$
进化路径累积"最近哪些方向被反复选" 的信息。

**(5) 更新协方差** (Rank-1 + Rank-$\mu$)：
$$\mathbf{C} \leftarrow (1-c_1-c_\mu)\mathbf{C} + c_1 \,\mathbf{p}_c\mathbf{p}_c^\top + c_\mu \sum_{i=1}^{\mu} w_i\,\mathbf{y}_{(i)}\mathbf{y}_{(i)}^\top$$
其中 $\mathbf{y}_{(i)} = (\mathbf{x}_{(i)}-\mathbf{m})/\sigma$。

**(6) 更新步长 (CSA)**：
$$\sigma \leftarrow \sigma \exp\left(\frac{c_\sigma}{d_\sigma}\left(\frac{\|\mathbf{p}_\sigma\|}{\mathbb{E}\|\mathcal{N}(\mathbf{0},\mathbf{I})\|} - 1\right)\right)$$
若进化路径模长大于期望，说明步走得太一致 → 加大 $\sigma$；反之缩小。

---

## 📐 关键公式与使用场景

### 公式 1：协方差更新规则
$$\mathbf{C} \leftarrow (1-c_1-c_\mu)\mathbf{C} + c_1\mathbf{p}_c\mathbf{p}_c^\top + c_\mu\sum_i w_i \mathbf{y}_i\mathbf{y}_i^\top$$

**为什么重要**：让 $\mathbf{C}$ "记忆" 哪些方向上一直在改进。这就是 CMA-ES 比普通 ES 更智能的关键——它在 **学习问题的几何**。

**使用场景**：
- **机器人参数调优**：步态参数 (10-100 维) 的非凸优化。
- **co-design 中的硬件结构搜索**：当设计变量是连续 + 噪声评估时，CMA-ES 经常是默认选择 (因为不需梯度)。
- **神经架构超参**：替代网格搜索。

### 公式 2：步长自适应
$$\sigma \leftarrow \sigma \exp\left(\frac{c_\sigma}{d_\sigma}(...)\right)$$

**含义**：步长根据"近期是否同方向" 自动加大或缩小。

**使用场景**：跨尺度优化 —— 既能在远处快速移动 (大 $\sigma$)，又能在最优附近精细搜索 (小 $\sigma$)。

---

## 🎯 实践要点

- **种群大小 $\lambda$**：通常 $\lambda = 4 + \lfloor 3\ln n\rfloor$ 起步；难问题加大。
- **初始化**：$\mathbf{m}_0$ 在合理区域内取，$\sigma_0 \approx$ 设计变量的合理尺度的 1/4。
- **维度上限**：实测 $n \le 100$ 效果极佳，几百维仍能用，更高维需 sep-CMA-ES 等变体。
- **计算复杂度**：每代 $O(n^2 \lambda)$ 主要在协方差矩阵分解。

---

## 🔑 对本论文的意义

- **历史地位**：在可微分模拟出现 **之前**，CMA-ES 是 co-design 领域的"默认黑盒求解器"。Lipson & Pollack (Ref [3]) 等的进化方法本质上就是 CMA-ES 的简化版。
- **本论文为何不用？** 设计空间是 **22 维连续刚度 + 6D 位姿 = 28 维连续**。维度尚可 (在 CMA-ES 的甜区)，但每次评估需 FEM (秒级)，CMA-ES 需要 ~$10^3$-$10^4$ 次评估，太慢。本论文换成 **代理模型 + 梯度下降**，单次评估降到毫秒，3 个数量级提速。
- **CMA-ES 仍是好基线**：未来如果引入"几何拓扑+刚度" 等离散变量，CMA-ES 或其变体可能回归。

---

## 🧭 延伸阅读

- 模块化机器人 GA (LexGA)：[[Ref-25-Kulz-2024-ModularGA]]
- 进化机器人开山：[[Ref-03-Lipson-2000-自动设计生命形态]]
- 贝叶斯优化对比：[[Ref-23-Snoek-2012-PracticalBO]]、[[Ref-22-Calandra-2016-BO]]
- 方法分类：[[../02-方法分类/采样优化方法]]

---

*← [[_参考文献索引]]*
