# [41] Faure et al. 2012 · SOFA 框架

> **引用**: F. Faure, C. Duriez, H. Delingette, J. Allard, B. Gilles, S. Marchesseau, H. Talbot, H. Courtecuisse, G. Bousquet, I. Peterlik, et al. *SOFA: A multi-model framework for interactive physical simulation.* in **Soft Tissue Biomechanical Modeling for Computer Assisted Surgery**, pp. 283-321, 2012.

`#文献-工具` `#软体仿真`

---

## 🎯 这个工具想做什么

SOFA (Simulation Open Framework Architecture) 由 Inria (法国) 团队开发，最初为 **手术仿真** 而生。目标：

> 提供一个 **多模型、可交互、模块化** 的物理仿真框架，支持 FEM、刚体、约束、碰撞、可形变体的统一处理。

---

## 🛠️ 实现路径

### 1. 多模型组合

SOFA 的核心抽象是 **场景图 (scene graph)**：
- 节点 = 物理对象。
- 每个节点可以挂多个 **求解器** (solver)、**力场** (force field)、**约束** (constraint)。

例如一根腱驱软体机器人的场景：
```
Root
├── Soft body (FEM tetrahedra)
│    ├── Mass
│    ├── Force field (Neo-Hookean)
│    └── Constraint (tendon waypoints)
├── Tendon (cable model)
│    └── Length constraint
└── Object (rigid)
     └── Contact constraint
```

### 2. 约束求解器 (LCP / Augmented Lagrangian)

接触和不可伸长腱通过 **拉格朗日乘子法** 求解：
$$\mathbf{M}\dot{\mathbf{v}} + \mathbf{C}\mathbf{v} + \mathbf{K}\mathbf{x} = \mathbf{f}_\text{ext} + \mathbf{J}^\top \boldsymbol{\lambda}$$
$$\mathbf{J}\mathbf{v} = 0 \quad (\text{约束})$$

求解 LCP 得到 $\boldsymbol{\lambda}$ (约束力)。

### 3. 时间积分
- 隐式 Euler / Newmark / RK4。
- 可调步长，平衡精度与速度。

### 4. 插件化
SOFA 有大量插件：
- **STLIB**：软体机器人专用。
- **SoftRobots Plugin**：腱驱、气动专门支持。
- **Sofa-Diff**：实验性可微版本。

---

## 📐 关键技术

### 技术 1：腱不可伸长约束

$$\sum_i \|P_{i+1} - P_i\| = L_0$$

通过雅可比 $\mathbf{J}$ 嵌入动力学方程的约束力。

**使用场景**：
- 真实腱机器人精确模拟。
- **本论文 (Yi'25) 不用此精确约束**——改用均匀张力近似 (Hirose'78)，速度快 10-50 倍。

### 技术 2：FEM 软体接触
**使用场景**：本论文虽未用 SOFA，但其 FEM 思路与 SOFA 一脉相承——只不过迁移到 GPU 友好的 Warp 上。

---

## 🔑 对本论文的意义

- **历史定位**：SOFA 是软体机器人仿真的"事实标准"。
- **本论文为何不用 SOFA**：
  1. SOFA 在 CPU 上跑，难以并行 80k 数据生成；
  2. SOFA 的腱约束求解慢；
  3. 需要 GPU 端可微 → 选 Warp。

简言之：SOFA 是金标准但慢；Warp 是新一代 GPU 速度优先。

---

## 🧭 延伸阅读

- NVIDIA Warp：[[Ref-18-Macklin-2022-Warp]]
- DiffTaichi：[[Ref-19-Hu-2020-DiffTaichi]]
- 软体爬行 (用 SOFA)：[[Ref-06-Schaff-2023-CrawlerSimToReal]]

---

*← [[_参考文献索引]]*
