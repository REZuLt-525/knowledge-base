# [20] Allen et al. 2022 · 用图网络模拟器做反向设计

> **原文**: K. Allen, T. Lopez-Guevara, K. L. Stachenfeld, A. Sanchez Gonzalez, P. Battaglia, J. B. Hamrick, T. Pfaff. *Inverse design for fluid-structure interactions using graph network simulators.* **NeurIPS**, 35:13759-13774, 2022.
> **译名**: 《基于图网络模拟器的流固耦合反向设计》 — 神经信息处理系统大会 (NeurIPS), 2022.

`#文献-方法` `#GNN模拟器` `#反设计`

---

## 1 · 研究问题

正向问题: **给定设计 → 预测物理行为**。
**反向问题** (Inverse Design): **给定期望行为 → 找出对应设计**。

经典反设计需要 **伴随法 + 求解器**, 工程师繁琐。在流固耦合 (fluid-structure interaction, FSI) 这种多物理耦合问题上更难。DeepMind Battaglia 团队的目标:

> 用 [[Ref-28-Pfaff-2021-MeshGraphNets|GNS (Graph Network Simulators)]] 做正向代理, 然后 **直接对设计参数反向传播** 求最优 — 把反设计统一为可微优化问题。

---

## 2 · 形式化

### 2.1 正向 GNS 代理

把网格视为图 $G_t = (V_t, E_t)$:
- 节点 $v_i$ 特征: $(\mathbf{x}_i, \mathbf{v}_i, \text{type}_i, \text{design}_i)$。
- 边 $e_{ij}$ 特征: $(\mathbf{x}_j - \mathbf{x}_i)$ + 等。

GNS 学正向动力学:
$$\hat{s}_{t+1} = f_\theta(s_t, \mathbf{d})$$

其中 $\mathbf{d}$ 是设计参数 (几何控制点等)。

### 2.2 任务损失

给定期望行为 $s^*$ (例如水流绕过物体的特定模式):
$$L(\mathbf{d}) = \big\| \hat{s}_T(\mathbf{d}) - s^*\big\|^2 + \lambda R(\mathbf{d})$$

其中 $\hat{s}_T = f_\theta^T(s_0, \mathbf{d})$ 是 $T$ 步 rollout 后的状态, $R$ 是设计正则。

### 2.3 反向梯度

通过 GNS 自动微分:
$$\nabla_\mathbf{d} L = \frac{\partial L}{\partial \hat{s}_T} \cdot \frac{\partial \hat{s}_T}{\partial \mathbf{d}}$$

链式展开:
$$\frac{\partial \hat{s}_T}{\partial \mathbf{d}} = \sum_{k=0}^{T-1}\left(\prod_{j=k+1}^{T-1} \frac{\partial f_\theta}{\partial s_j}\right) \cdot \frac{\partial f_\theta}{\partial \mathbf{d}}\bigg|_{s_k}$$

由于 GNS 是 NN, 自动微分高效计算。

---

## 3 · 算法

```
─── Inverse Design via GNS ────────────────
─ 阶段 1: 训练 GNS ────────────────────────
1) 用高保真 CFD/FSI 模拟生成 (s_t, d, s_{t+1}) 数据
2) 训 GNS f_θ 学正向动态:
     L_train = Σ ‖f_θ(s_t, d) - s_{t+1}‖²
3) 验证 long rollout 误差

─ 阶段 2: 反向设计 ─────────────────────────
初始化 d = d_0
重复 t = 1..T_max:
    用 GNS rollout T 步: ŝ_T = f_θ^T(s_0, d)
    L = ‖ŝ_T - s*‖² + λ R(d)
    g_d = autograd.grad(L, d)
    d ← d - η g_d
返回 d* (优化后设计)
```

---

## 4 · 关键挑战与对策

### 4.1 长 rollout 误差累积
- 训练时加 long-horizon loss: $L_\text{train} = \sum_{t=1}^{T} \|\hat{s}_t - s_t\|^2$。
- 多次重起 (re-rollout) 用真值校正。
- Curriculum: 先短 rollout 再长。

### 4.2 设计可制造性
- 加几何约束 (例: 体积 $\le V_\max$)。
- 平滑正则: $R(\mathbf{d}) = \|\nabla \mathbf{d}\|^2$。

### 4.3 OOD 鲁棒性
- 训练时多样化设计分布。
- 反设计时检查 $\sigma(\mathbf{d}^*)$ (集成代理方差)。

---

## 5 · 实验

### 5.1 流固耦合 (FSI) 测试

#### 任务 1: 水翼形状反设计
- 目标: 升力/阻力比最大。
- 设计: 翼型轮廓控制点。
- 结果: 反设计找到的翼型与经典 NACA 优化结果接近。

#### 任务 2: 柔性结构在流场中的姿态
- 目标: 让柔性叶片偏转到指定角度。
- 设计: 弹性模量分布。
- 结果: 找到与解析解一致的最优分布。

### 5.2 量化对比

| 方法 | 反设计速度 | 精度 |
|---|---|---|
| 经典伴随法 (FSI) | 数小时 | 高 |
| 进化算法 + 黑盒 | 数日 | 中 |
| **GNS + AD** | **数分钟** | **接近经典** |

GNS 反设计提速数十倍。

---

## 6 · 关键贡献

1. **GNS 用作可微代理**: 第一次系统证明 GNS 可服务反向设计。
2. **流固耦合统一框架**: 不需要分别处理流体和结构。
3. **反传链路**: 通过 NN 反向, 绕过经典伴随复杂性。

---

## 7 · 局限

1. **GNS 训练昂贵**: 需大量 CFD 数据。
2. **rollout 漂移**: 长 rollout 仍有累积误差。
3. **设计空间维度受 GNS 表达力限制**。

(后由 [[Ref-30-Allen-2022-GDM|GD-M]] 扩展到更高维设计。)

---

## 8 · 历史影响

- DeepMind GNS 系列代表作之一。
- 推动"NN 代理 + 反设计" 路线。
- 与 [[Ref-29-deAvilaBelbute-2020-GraphPDE|de Avila Belbute-Peres'20]] 形成代理路线 vs 混合路线对照。

---

## 9 · 与本论文 (Yi'25) 的关系

本论文综述明确引用 Allen'22 作为 **NN 代理 + 反设计** 路线先例。两者哲学一致, 区别:
- Allen: GNS rollout 多步动态 (流场)。
- 本论文: 末态预测 (抓取结果)。

本论文是 Allen'22 思想在机器人 co-design 上的应用 + 简化。

---

*← [[_参考文献索引]]*
