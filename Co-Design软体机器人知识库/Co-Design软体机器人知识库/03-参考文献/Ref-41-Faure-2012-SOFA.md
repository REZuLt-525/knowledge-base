# [41] Faure et al. 2012 · SOFA 框架

> **原文**: F. Faure, C. Duriez, H. Delingette, J. Allard, B. Gilles, S. Marchesseau, H. Talbot, H. Courtecuisse, G. Bousquet, I. Peterlik, et al. *SOFA: A multi-model framework for interactive physical simulation.* in **Soft Tissue Biomechanical Modeling for Computer Assisted Surgery**, pp. 283-321, 2012.
> **译名**: 《SOFA: 用于交互式物理仿真的多模型框架》 — 《计算机辅助手术中的软组织生物力学建模》, 2012.

`#文献-工具` `#软体仿真`

---

## 1 · 工具定位

2000 年代医学界需要 **手术仿真器** — 让医生在虚拟环境中练习手术。Inria (法国) François Faure + Christian Duriez 团队联合欧洲多家大学, 开发了 **SOFA (Simulation Open Framework Architecture)** 框架, 目标:

> 提供一个 **多模型、可交互、模块化** 的物理仿真框架, 支持 FEM / 刚体 / 约束 / 碰撞 / 可形变体的统一处理, 主要用于 **医学手术仿真**, 但也广泛适用于软体机器人。

SOFA 一直是 **软体机器人学术界的事实标准仿真器** (~10 年), 直到 2020s 被 Warp / DiffTaichi 部分取代。

---

## 2 · 框架架构

### 2.1 Scene Graph (场景图)

SOFA 的核心抽象 — 每个物理对象是一个节点:

```
Root (世界)
├── 对象 1: 软体手术工具
│    ├── MechanicalState (位置, 速度, 力)
│    ├── Mass (质量分布)
│    ├── ForceField (Neo-Hookean 等本构)
│    ├── Constraint (位姿约束)
│    └── Topology (FEM 网格)
├── 对象 2: 患者组织
│    ├── ...
└── Solver (时间积分器)
    ├── EulerImplicit
    └── ConjugateGradient (CG 求解线性系统)
```

模块化设计 → 用户组合不同物理实现复杂混合系统。

### 2.2 物理求解流程

每个时间步:
$$\mathbf{M}\dot{\mathbf{v}} + \mathbf{C}\mathbf{v} + \mathbf{K}\mathbf{x} = \mathbf{f}_\text{ext} + \mathbf{J}^\top \boldsymbol{\lambda}$$
$$\mathbf{J}\mathbf{v} = 0\;\;(\text{约束})$$

求解步骤:
1. 计算外力 $\mathbf{f}_\text{ext}$。
2. 隐式 Euler 离散: $\mathbf{v}_{t+1} - \mathbf{v}_t = dt \cdot \ddot{\mathbf{q}}$。
3. 求解线性系统 (CG 迭代)。
4. 处理约束 (LCP / Augmented Lagrangian)。
5. 更新状态。

---

## 3 · 软体本构

### 3.1 内置本构模型

| 模型 | 适用 | 公式 |
|---|---|---|
| Linear elastic | 小变形 | $\boldsymbol{\sigma} = \mathbf{D}\boldsymbol{\varepsilon}$ |
| St. Venant-Kirchhoff | 中等变形 | $\boldsymbol{\sigma} = \lambda \text{tr}(\mathbf{E})\mathbf{I} + 2\mu\mathbf{E}$ |
| Neo-Hookean | 大变形, 默认 | $\psi(\mathbf{F}) = \frac{\mu}{2}(I_1 - 3) - \mu \log J + \frac{\lambda}{2}\log^2 J$ |
| Mooney-Rivlin | 橡胶 | $\psi = c_1(I_1-3) + c_2(I_2-3)$ |

### 3.2 数值积分

显式 Euler / 隐式 Euler / Newmark 等可选。
软体大变形通常用隐式 Euler (稳定性好)。

---

## 4 · 接触求解

### 4.1 LCP (Linear Complementarity Problem)

接触约束:
$$0 \le g(\mathbf{u}) \perp \boldsymbol{\lambda} \ge 0$$
($g$: 间隙函数, $\boldsymbol{\lambda}$: 接触力)

转化为 LCP:
$$\mathbf{w} = \mathbf{M}\boldsymbol{\lambda} + \mathbf{q},\;\; 0 \le \mathbf{w} \perp \boldsymbol{\lambda} \ge 0$$

用 PGS (Projected Gauss-Seidel) 或 Lemke 算法迭代求解。

### 4.2 Augmented Lagrangian

加 penalty 项:
$$L_\text{aug}(\mathbf{u}, \boldsymbol{\lambda}) = L(\mathbf{u}, \boldsymbol{\lambda}) + \frac{\rho}{2}\|g(\mathbf{u})\|^2$$
$\rho$ 大 → 接近硬约束。

---

## 5 · 软体机器人插件 (SoftRobots Plugin)

由 Inria + Lille 大学 Christian Duriez 团队维护, 为 SOFA 加入软体机器人专用工具:

### 5.1 腱驱动 (cable model)

腱不可伸长约束:
$$\sum_i \|\mathbf{P}_{i+1} - \mathbf{P}_i\| = L_0$$

通过雅可比 $\mathbf{J}$ 嵌入约束求解器:
$$\mathbf{J}\mathbf{v} = -\dot{L}_\text{actuator}$$

### 5.2 气动腔体

充气压力 → 内壁法向力:
$$\mathbf{f}_\text{cavity} = P \cdot \mathbf{n} \cdot dA$$

腔体体积变化反算压力。

### 5.3 形状记忆合金 (SMA)

温度驱动相变:
$$E(T), \;\; \boldsymbol{\varepsilon}(T)$$
温度场作为额外状态。

---

## 6 · 性能与生态

### 6.1 性能基准

| 任务 | SOFA (CPU 单核) | Warp (GPU) |
|---|---|---|
| FEM 1k tet 1k 步 | ~30 s | ~0.1 s |
| 软体机器人 + 接触 | ~分钟 | ~秒 |

CPU 是 SOFA 主要瓶颈 (设计于 2007 年代)。

### 6.2 应用领域

- 微创手术训练 (Inria 创业公司 InSimo 商业化)。
- 软体机器人原型 (例 Schaff 2023, [[Ref-06-Schaff-2023-CrawlerSimToReal]])。
- 工业碰撞测试。
- 教学。

---

## 7 · 关键贡献

1. **统一框架**: 多模型 + 模块化 + 场景图。
2. **开源 + 学术友好**: GitHub 公开, 无商业壁垒。
3. **手术仿真专用**: 软组织建模深度优化。
4. **软体机器人插件**: 腱、气动、SMA 一体支持。

---

## 8 · 局限

1. **CPU 主导, GPU 弱**: 不能并行大规模数据生成。
2. **C++ 实现**: 不像 PyTorch 易用, 学习曲线陡。
3. **可微性弱**: sofa-diff 实验性, 不如 Warp / DiffTaichi 成熟。
4. **生态老**: 很多依赖与现代 ML 框架不兼容。

---

## 9 · 历史影响

- 软体机器人 / 医学仿真 ~10 年事实标准。
- 衍生 InSimo 商业化、SoftRobots Plugin、HiPro 项目。
- Christian Duriez 是软体仿真领域世界级专家。

---

## 10 · 与本论文 (Yi'25) 的关系

本论文综述把 SOFA 列为 **可微仿真** 替代品之一。本论文 **不用 SOFA**, 因为:
1. SOFA CPU 慢, 不能并行 80k 数据生成。
2. SOFA AD 弱。
3. SOFA C++ 不像 Python 易迭代。

转用 Warp ([[Ref-18-Macklin-2022-Warp]]) → GPU + Python + AD 三者俱佳。

---

*← [[_参考文献索引]]*
