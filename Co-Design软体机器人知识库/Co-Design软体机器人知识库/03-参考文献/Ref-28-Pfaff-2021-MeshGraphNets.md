# [28] Pfaff et al. 2021 · MeshGraphNets

> **原文**: T. Pfaff, M. Fortunato, A. Sanchez-Gonzalez, P. Battaglia. *Learning mesh-based simulation with graph networks.* **ICLR 2021**.
> **译名**: 《用图网络学习基于网格的物理仿真》 — 国际表征学习大会 (ICLR), 2021.

`#文献-方法` `#GNN模拟器` `#里程碑`

---

## 1 · 研究问题

物理仿真 (流体、布料、固体) 通常用 FEM/SPH 等, **计算贵且需领域知识**。神经网络是否能直接学物理? 如果能, 速度提升多少, 精度多少?

DeepMind Battaglia 团队的目标:

> 训一个 **图神经网络 (GNN)** 直接学习"网格状态 → 下一时刻状态"映射, 在精度上接近 FEM 但速度快 10-100×, 并能 **泛化到不同分辨率 / 形状 / 边界条件**。

---

## 2 · 形式化

### 2.1 网格 → 图

物理网格 $M = (V, E_M)$ 转为图 $G_t$:

$$G_t = (V_t, E_t^{(M)} \cup E_t^{(W)})$$

**变量含义**：
- $V$ — 网格顶点集合。
- $E^{(M)}$ — **网格边** (mesh edges): 物理拓扑连接。
- $E^{(W)}$ — **世界边** (world edges): 添加 **空间近邻边** (KD-tree 找半径内顶点) — 用于建模接触和远场相互作用。
- $G_t$ — $t$ 时刻图。

**公式解读**：除物理网格连接外, 还加入"近邻边"让模型感知非接触相互作用。

节点特征:
$$\mathbf{v}_i = (\mathbf{x}_i, \dot{\mathbf{x}}_i, \text{type}_i)$$

**变量含义**：
- $\mathbf{x}_i \in \mathbb{R}^3$ — 节点 $i$ 位置。
- $\dot{\mathbf{x}}_i$ — 节点速度 (历史 1-2 步差分)。
- $\text{type}_i$ — 节点类型 one-hot (流体/固体/边界等)。

边特征:
$$\mathbf{e}_{ij} = (\mathbf{x}_j - \mathbf{x}_i,\;\; \|\mathbf{x}_j - \mathbf{x}_i\|)$$

**变量含义**：
- 相对位移向量 + 标量距离 (4 维)。

**公式解读**：边特征只用 **相对量** → 平移不变性。

### 2.2 编码-处理-解码架构

```
─── 编码器 (Encoder) ─────────────────────
v_i^0 = MLP_node(v_i)
e_ij^0 = MLP_edge(e_ij)

─── 处理器 (Processor): L 层消息传递 ─────
for ℓ = 1..L:
    e_ij^ℓ = MLP^{ℓ}_e([e_ij^{ℓ-1}, v_i^{ℓ-1}, v_j^{ℓ-1}])
    v_i^ℓ = MLP^{ℓ}_v([v_i^{ℓ-1}, Σ_{j∈N(i)} e_ij^ℓ])

─── 解码器 (Decoder) ─────────────────────
Δv_i = MLP_dec(v_i^L)
```

### 2.3 消息传递公式

**边更新**:
$$\mathbf{e}_{ij}^{\ell+1} = f_e^\ell([\mathbf{e}_{ij}^\ell, \mathbf{v}_i^\ell, \mathbf{v}_j^\ell])$$

**变量含义**：
- $\mathbf{e}_{ij}^\ell, \mathbf{v}_i^\ell$ — 第 $\ell$ 层边/节点特征。
- $f_e^\ell$ — 第 $\ell$ 层边更新 MLP。
- $[\cdot, \cdot]$ — 向量拼接 (concatenation)。

**公式解读**：每条边的新特征由其两端节点 + 自身原特征更新 — 边"知道"它连接的两个节点信息。

**节点更新**:
$$\mathbf{v}_i^{\ell+1} = f_v^\ell\Big(\mathbf{v}_i^\ell,\; \sum_{j \in \mathcal{N}(i)} \mathbf{e}_{ij}^{\ell+1}\Big)$$

**变量含义**：
- $\mathcal{N}(i)$ — 节点 $i$ 的邻居集合。
- $\sum$ — 邻居边特征求和 (置换不变聚合)。

**公式解读**：每个节点用 **自身特征 + 所有相邻边特征求和** 更新 — 这就是 "**消息传递**" (message passing): 节点之间通过边传递信息。

堆叠 $L$ 层后, 节点知道 $L$-跳邻域信息。最后预测 $\Delta \mathbf{v}_i$, 然后用半隐式 Euler 积分得下一时刻位置。

### 2.4 时间积分

$$\mathbf{v}_{i,t+1} = \mathbf{v}_{i,t} + \Delta \mathbf{v}_i \cdot dt$$
$$\mathbf{x}_{i,t+1} = \mathbf{x}_{i,t} + \mathbf{v}_{i,t+1} \cdot dt$$

**变量含义**：
- $\Delta \mathbf{v}_i$ — 网络预测的速度增量。
- $dt$ — 时间步长。

**公式解读**：标准半隐式 Euler 积分, 比显式更稳定。

### 2.5 训练损失

逐步监督:
$$L_\text{step} = \big\|\Delta \mathbf{v}_\text{pred} - \Delta \mathbf{v}_\text{true}\big\|^2$$

**变量含义**：
- $\Delta \mathbf{v}_\text{pred}$ — GNN 预测。
- $\Delta \mathbf{v}_\text{true}$ — FEM 真值。

长时间步监督 (减漂移):
$$L_\text{rollout} = \sum_{t=1}^{H} \big\|\hat{\mathbf{x}}_t - \mathbf{x}_t\big\|^2$$

**变量含义**：
- $\hat{\mathbf{x}}_t$ — GNN 多步 rollout 预测。
- $\mathbf{x}_t$ — 真值轨迹。
- $H$ — 多步预测长度 (例 5)。

**公式解读**：让 GNN 不只单步准, 多步累积也准 → 减少 rollout 漂移。

加入 **noise injection** 在训练时让模型对自身预测漂移鲁棒 (类似 schedule sampling)。

---

## 3 · 自适应网格

### 3.1 重网格指标

每个节点 $i$ 估计 **局部误差** / **重要性**:
$$\text{importance}(v_i) = \|\nabla \mathbf{v}_i\|$$

**变量含义**：$\nabla \mathbf{v}_i$ — 速度场在 $v_i$ 处的局部梯度幅度。变形剧烈处梯度大 → 重要性高 → 加密。

### 3.2 自适应操作

```
每 K 步执行:
    for each edge e_ij:
        if importance > threshold_high:
            edge_split(e_ij)            # 加密
        if importance < threshold_low:
            edge_collapse(e_ij)         # 抽稀
```

精度-成本自适应平衡。

---

## 4 · 实验

### 4.1 测试任务

| 任务 | 物理 | 网格大小 |
|---|---|---|
| 布料 | 弹簧 + 风力 | 1k 顶点 |
| 流体 | Eulerian | 5k 顶点 |
| 结构变形 | FEM | 2k 顶点 |
| 流-体耦合 (FSI) | 多物理 | 10k 顶点 |

### 4.2 量化对比

| 方法 | 单步预测误差 | 速度 |
|---|---|---|
| 真实 FEM/CFD | 0 (基准) | 1× |
| 简化模型 | 大 | 10× |
| **MGN (本文)** | **接近 FEM** | **10-100×** |

### 4.3 泛化性
- 训 1k 顶点网格, 测 10k 顶点: 性能轻微下降但仍合理。
- 训方形几何, 测圆形: 同样可行。

### 4.4 自适应优势
- 固定网格 1000 顶点: 误差 5%。
- 自适应 (平均 600 顶点): 误差 3%, 速度 1.5×。

---

## 5 · 关键贡献

1. **MGN 架构**: 网格图 + 消息传递 + 自适应。
2. **多物理统一**: 同一框架处理流体、布料、结构。
3. **泛化性**: 不同分辨率 / 几何 / 边界条件通用。
4. **训练技术**: noise injection + long-horizon loss 减 rollout 漂移。

---

## 6 · 局限

1. **训练数据需求大**: 需高保真 FEM/CFD 真值。
2. **接触场景仍弱**: 自适应 + 世界边帮助但不完美。
3. **GPU 内存**: 大网格 + 多层 MLP 内存占用大。
4. **长 rollout 漂移**: 减弱但未消除。

---

## 7 · 历史影响

- 引发 **"GNN + 物理仿真"** 研究热潮。
- 衍生 GNS, MeshGraphNets-Visualizer, Inverse Design GNS ([[Ref-20-Allen-2022-InverseDesignGNS]])。
- 启发本论文 (Yi'25) 等 "**NN 替代物理**" 思想。

---

## 8 · 与本论文 (Yi'25) 的关系

本论文综述把 MGN 列为 **"GNN 模拟器"** 代表。哲学共鸣: 都用 NN 替代物理。区别:
- MGN: rollout 多步动态预测。
- 本论文: 末态预测 (单次前向)。

本论文选 PointNet+MLP 而非 MGN, 因为末态预测更轻量, 训练更快。

---

*← [[_参考文献索引]]*
