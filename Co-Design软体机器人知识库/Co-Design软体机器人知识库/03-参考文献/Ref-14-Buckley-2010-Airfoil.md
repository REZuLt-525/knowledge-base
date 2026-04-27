# [14] Buckley, Zhou & Zingg 2010 · 翼型气动优化

> **原文**: H. Buckley, B. Zhou, D. Zingg. *Airfoil optimization using practical aerodynamic design requirements.* **Journal of Aircraft**, 47, 2010. doi:10.2514/1.C000256
> **译名**: 《基于实用气动设计要求的翼型优化》 — *飞行器学报*, 2010.

`#文献-方法` `#拓扑优化` `#伴随法`

---

## 1 · 研究问题

翼型 (airfoil) 设计长期靠工程经验。CFD (Computational Fluid Dynamics) 可数值评估升力阻力, 但 **多工况 + 实际工程约束** 下找最优外形仍是难题。Toronto 大学 David Zingg 团队的目标:

> 把 **伴随法 (adjoint method)** 与 **多工况约束 + 工程实用约束** 结合, 给出一种 **解决实际飞机设计问题** 的翼型优化方法 — 不仅在单一工况下减阻, 而是在多种巡航 / 起降 / 高度组合下都鲁棒。

---

## 2 · 形式化

### 2.1 翼型参数化

用 **B-spline 控制点** 描述上下表面:
$$y(x) = \sum_{i=0}^{n} c_i \cdot B_i(x)$$

**变量含义**：
- $x$ (m) — 沿翼弦方向坐标。
- $y(x)$ (m) — 翼型上/下表面高度。
- $c_i$ — 第 $i$ 个 **控制点高度** (设计变量)。
- $B_i(x)$ — 第 $i$ 个 B-spline 基函数 (在 $x$ 处对 $c_i$ 的权重)。
- $n$ — 控制点数 (典型 30)。

**公式解读**：把翼型形状用控制点参数化, 每个控制点高度独立可调, B-spline 保证形状光滑。

设计变量向量 $\mathbf{x} = (c_0, ..., c_n)^\top \in \mathbb{R}^{n+1}$。

### 2.2 流场求解 (CFD)

求解 Reynolds-Averaged Navier-Stokes (RANS):

**质量守恒**:
$$\frac{\partial \rho}{\partial t} + \nabla \cdot (\rho \mathbf{u}) = 0$$

**变量含义**：
- $\rho$ (kg/m³) — 流体密度。
- $\mathbf{u}$ (m/s) — 流速向量场。
- $\nabla \cdot$ — 散度算子。

**公式解读**：单位体积流体质量随时间变化 = 流入流出净量 (流体不消失)。

**动量守恒**:
$$\frac{\partial (\rho \mathbf{u})}{\partial t} + \nabla \cdot (\rho \mathbf{u} \mathbf{u}) + \nabla p = \nabla \cdot \boldsymbol{\tau}$$

**变量含义**：
- $p$ (Pa) — 压力。
- $\boldsymbol{\tau}$ (Pa) — 粘性应力张量 (描述流体内部摩擦)。

**公式解读**：单位体积动量变化 + 对流通量 + 压力梯度 = 粘性力 (Navier-Stokes 方程)。

**能量守恒**:
$$\frac{\partial (\rho E)}{\partial t} + \nabla \cdot (\rho E \mathbf{u} + p \mathbf{u}) = \nabla \cdot (\boldsymbol{\tau} \cdot \mathbf{u}) + \nabla \cdot (k \nabla T)$$

**变量含义**：
- $E$ (J/kg) — 比总能 (动能 + 内能)。
- $T$ (K) — 温度。
- $k$ (W/(m·K)) — 热导率。

**公式解读**：能量平衡 (含粘性发热和热传导)。

离散后得线性系统:
$$\mathbf{R}(\mathbf{u}, \mathbf{x}) = 0$$

**变量含义**：
- $\mathbf{R}$ — **残差向量**: 离散方程的残差。
- $\mathbf{u}$ — 流场状态变量 (节点上的密度 / 速度 / 压力)。
- $\mathbf{x}$ — 翼型设计变量。

**公式解读**：离散后 NS 方程变成"找 $\mathbf{u}$ 使 $\mathbf{R} = 0$"的代数问题。

### 2.3 目标函数

阻力 + 升力比:
$$J(\mathbf{u}, \mathbf{x}) = w_1 \cdot C_D - w_2 \cdot C_L + w_3 \cdot |C_M|^2$$

**变量含义**：
- $C_D$ — **阻力系数** (无量纲, 越小越好)。
- $C_L$ — **升力系数** (越大越好, 故 $-C_L$ 出现在最小化目标里)。
- $C_M$ — **力矩系数** (尽量接近 0)。
- $w_1, w_2, w_3$ — 权重。

**公式解读**：综合考虑阻力小 + 升力大 + 力矩小 (稳定)。

---

## 3 · 伴随法梯度推导 ⭐

### 3.1 拉格朗日

$$\mathcal{L}(\mathbf{u}, \mathbf{x}, \boldsymbol{\lambda}) = J(\mathbf{u}, \mathbf{x}) + \boldsymbol{\lambda}^\top \mathbf{R}(\mathbf{u}, \mathbf{x})$$

**变量含义**：
- $\boldsymbol{\lambda}$ — **拉格朗日乘子向量** (维度同 $\mathbf{u}$)。
- 由于 $\mathbf{R} = 0$, $\mathcal{L} = J$ (与原目标一致)。

**公式解读**：用拉格朗日乘子把约束 $\mathbf{R} = 0$ 代入目标。

### 3.2 求导

$$\frac{d\mathcal{L}}{d\mathbf{x}} = \frac{\partial J}{\partial \mathbf{x}} + \frac{\partial J}{\partial \mathbf{u}}\frac{d\mathbf{u}}{d\mathbf{x}} + \boldsymbol{\lambda}^\top\left(\frac{\partial \mathbf{R}}{\partial \mathbf{x}} + \frac{\partial \mathbf{R}}{\partial \mathbf{u}}\frac{d\mathbf{u}}{d\mathbf{x}}\right)$$

**公式解读**：链式求导, 含直接项 ($\partial J / \partial \mathbf{x}$) 和间接项 (通过 $\mathbf{u}$ 传递)。

选 $\boldsymbol{\lambda}$ 使最棘手的 $\dfrac{d\mathbf{u}}{d\mathbf{x}}$ 项消失:

$$\boxed{\left(\frac{\partial \mathbf{R}}{\partial \mathbf{u}}\right)^\top \boldsymbol{\lambda} = -\frac{\partial J}{\partial \mathbf{u}}^\top}$$ ← **伴随方程**

**变量含义**：
- $\frac{\partial \mathbf{R}}{\partial \mathbf{u}} \in \mathbb{R}^{n_u \times n_u}$ — CFD 雅可比矩阵 (流场方程对状态求导)。
- 转置后乘以 $\boldsymbol{\lambda}$ 等于负的目标对状态梯度。

**公式解读**：解一次伴随系统 (与原方程同形式) 即得 $\boldsymbol{\lambda}$, 把昂贵的 $d\mathbf{u}/d\mathbf{x}$ 消除。

### 3.3 最终梯度

$$\boxed{\frac{dJ}{d\mathbf{x}} = \frac{\partial J}{\partial \mathbf{x}} + \boldsymbol{\lambda}^\top\frac{\partial \mathbf{R}}{\partial \mathbf{x}}}$$

**关键性质**: 计算梯度只需 **解一次伴随系统**, **与设计变量数 $\dim \mathbf{x}$ 无关**。

**直觉**：
- 不用伴随法, 每个 $x_i$ 都要重解一次 CFD ($n_x$ 次)。
- 用伴随法, 总共只需 2 次 (一次正向 + 一次伴随)。
- 当 $n_x = 100$ 时, 伴随法快 50×。

---

## 4 · 多工况优化

实际飞机要在多种工况 (Mach 数 $M_k$, 攻角 $\alpha_k$) 下都性能良好:

$$\min_\mathbf{x} \;\; \sum_{k=1}^{K} w_k \cdot J_k(\mathbf{u}^{(k)}, \mathbf{x})$$

**变量含义**：
- $k$ — 工况索引。
- $K$ — 工况总数 (例 5)。
- $w_k$ — 工况权重。
- $J_k$ — 第 $k$ 工况下的目标。

每个工况独立求 CFD + 伴随, 然后梯度加权:
$$\frac{dJ_\text{total}}{d\mathbf{x}} = \sum_k w_k \frac{dJ_k}{d\mathbf{x}}$$

**公式解读**：多工况梯度按权重相加 → 优化方向是各工况"妥协方向"。

---

## 5 · 实用约束

论文强调 **实际工程约束**:
- **结构强度**: 翼型最小厚度 $\ge t_\text{min}$。
- **油箱体积**: 翼型截面积 $\ge V_\text{tank}/L$。
- **前缘半径**: 制造可行性 + 失速性能。
- **力矩平衡**: $|C_M| \le C_M^\text{max}$。

约束加为惩罚或显式 KKT。

---

## 6 · 实验

### 6.1 测试工况
- 巡航: $M = 0.85$, $\alpha = 1.5°$。
- 高升力: $M = 0.6$, $\alpha = 5°$。
- 高速冲刺: $M = 0.92$, $\alpha = 0°$。

### 6.2 优化结果
- 阻力减少 5-10%。
- 多工况鲁棒性 (单工况优化在其他工况下退化, 多工况优化则平均最优)。

### 6.3 量化
- 设计变量 $n \sim 30$ 控制点。
- 每代 1 次 CFD + 1 次伴随 ~ 数小时。
- 优化收敛 ~ 50 代 (天级)。

---

## 7 · 关键贡献

1. **多工况伴随法**: 实用化飞机设计。
2. **工程约束**: 把数学优化与制造约束结合。
3. **数值稳定**: 离散伴随选择 + 网格收敛性分析。

---

## 8 · 局限

1. **依赖网格**: 网格质量影响伴随精度。
2. **不处理多学科耦合**: 仅气动, 不含结构、热、声。
3. **不处理拓扑**: 只优化形状, 不能加 / 减翼面。

(后续 [[Ref-15-He-2019-DAFoam|DAFoam]] 把多学科加进来。)

---

## 9 · 历史影响

- 经典 CFD 优化教材引用文献。
- 推动 OpenFOAM + 伴随法工具化。
- 启发后续 DAFoam ([[Ref-15-He-2019-DAFoam]]) 开源框架。

---

## 10 · 与本论文 (Yi'25) 的关系

作为 **可微设计在工程界的经典先例**, 本论文综述把它列入 "**梯度优化方法**" 类别。本论文软体接触场景下伴随法不可用 (非线性 + 不连续), 转用 NN 代理路线。

---

*← [[_参考文献索引]]*
