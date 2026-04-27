# FEM 有限元法

> **一句话理解**：把求解域 $\Omega$ 离散为有限多个简单形状 (单元 element)，在每个单元内用形函数 (shape function) 插值场量，然后通过弱形式 (变分原理) 将偏微分方程 (PDE) 转为代数线性方程 $\mathbf{K}\mathbf{u} = \mathbf{f}$ 求解。

`#核心概念`

---

## 1. FEM 完整工作流

### 1.1 强形式 PDE (线弹性问题)

$$\nabla \cdot \boldsymbol{\sigma} + \mathbf{b} = 0,\quad \mathbf{x} \in \Omega$$

**变量含义**：
- $\Omega \subset \mathbb{R}^3$ — **求解域**, 即物体占据的空间。
- $\mathbf{x} \in \Omega$ — 域内点的位置坐标。
- $\boldsymbol{\sigma} \in \mathbb{R}^{3 \times 3}$ — **应力张量**, 描述每个点的内部受力 (单位 Pa)。
- $\nabla \cdot \boldsymbol{\sigma}$ — 应力的散度 (向量), 物理意义 = 单位体积净力。
- $\mathbf{b} \in \mathbb{R}^3$ — **体力**, 例如重力 $\rho \mathbf{g}$ (单位 N/m³)。

**公式解读**：这是 **Cauchy 平衡方程** — 物体每点处净力为零 (静态情况)。是连续介质力学的基本方程。

边界条件:

$$\mathbf{u} = \bar{\mathbf{u}},\quad \mathbf{x} \in \Gamma_D \quad (\text{Dirichlet})$$
$$\boldsymbol{\sigma} \cdot \mathbf{n} = \bar{\mathbf{t}},\quad \mathbf{x} \in \Gamma_N \quad (\text{Neumann})$$

**变量含义**：
- $\Gamma_D$ — **位移边界** (固定面)。
- $\Gamma_N$ — **力边界** (受外力面)。
- $\bar{\mathbf{u}}$ — 在 $\Gamma_D$ 上规定的位移值 (例如 0 表示固定)。
- $\mathbf{n}$ — 边界外法向单位向量。
- $\bar{\mathbf{t}}$ — 在 $\Gamma_N$ 上的力面密度 (单位 Pa)。

**公式解读**：边界要么"位置已知" (Dirichlet) 要么"受力已知" (Neumann)。

### 1.2 弱形式 (变分原理)

乘以 **试函数 (test function)** $\mathbf{v}$ 并分部积分:

$$\int_\Omega \boldsymbol{\sigma}(\mathbf{u}) : \nabla \mathbf{v}\, dV = \int_\Omega \mathbf{b} \cdot \mathbf{v}\, dV + \int_{\Gamma_N} \bar{\mathbf{t}} \cdot \mathbf{v}\, dA$$

**变量含义**：
- $\mathbf{v}$ — 任意位移检验场 (满足边界条件)。
- $\nabla \mathbf{v}$ — 检验场梯度 (3×3 矩阵)。
- $:$ — 双点积 (两矩阵对应元素相乘求和), $\mathbf{A} : \mathbf{B} = \sum_{ij} A_{ij} B_{ij}$。
- $dV, dA$ — 体积积分 / 面积积分微元。

**公式解读**：把"对每点都满足平衡" 转为"任何虚位移下做的功也平衡"——这是更弱但等价的形式, 适合数值求解。

### 1.3 离散化

把 $\Omega$ 分成 $N_e$ 个单元 $\{\Omega_e\}$, 节点位移用 **形函数** $N_i(\mathbf{x})$ 插值:

$$\mathbf{u}(\mathbf{x}) \approx \sum_{i=1}^{N_n} N_i(\mathbf{x})\,\mathbf{u}_i$$

**变量含义**：
- $N_n$ — 总节点数。
- $\mathbf{u}_i \in \mathbb{R}^3$ — 第 $i$ 个节点的位移 (待求未知量)。
- $N_i(\mathbf{x})$ — **形函数**: 在节点 $i$ 处为 1, 在其他节点处为 0, 单元内线性 / 多项式插值。

**公式解读**：把"无穷自由度的连续位移场"用"有限节点位移加权"近似。形函数像"插值基函数"。

### 1.4 全局线性系统

把弱形式中 $\mathbf{u}, \mathbf{v}$ 都用形函数代入, 得 **线性方程组**:

$$\boxed{\mathbf{K}\,\mathbf{U} = \mathbf{F}}$$

**变量含义**：
- $\mathbf{U} \in \mathbb{R}^{3 N_n}$ — 所有节点位移堆叠 (未知向量)。
- $\mathbf{F} \in \mathbb{R}^{3 N_n}$ — 等效节点力 (来自体力 + 边界力)。
- $\mathbf{K} \in \mathbb{R}^{3N_n \times 3N_n}$ — **全局刚度矩阵**: 描述结构对位移的"硬度"。

**全局刚度矩阵装配**:
$$\mathbf{K} = \sum_e \mathbf{K}_e,\quad \mathbf{K}_e = \int_{\Omega_e} \mathbf{B}^\top \mathbf{D} \mathbf{B}\, dV$$

**变量含义**：
- $\mathbf{K}_e$ — 第 $e$ 个单元的局部刚度矩阵。
- $\mathbf{B} \in \mathbb{R}^{6 \times 3 n_e}$ — **应变-位移矩阵** ($n_e$: 单元节点数), 把节点位移转为单元应变。
- $\mathbf{D} \in \mathbb{R}^{6 \times 6}$ — **本构矩阵**: 把应变映射到应力, 由材料 $E, \nu$ 决定。
- $dV$ — 单元体积微元。

**公式解读**：每个单元对应一个小刚度矩阵, 整个结构刚度 = 所有单元贡献求和。

### 1.5 单元类型对照表

| 元素 | 维度 | 节点数 | 应用 |
|---|---|---|---|
| 线段 (truss) | 1D | 2 | 桁架 |
| 三角形 / 四边形 | 2D | 3 / 4 | 平面应力 |
| **四面体 Tet** | 3D | 4 | **本论文软体** |
| 六面体 Hex | 3D | 8 | 规则结构 |

本论文用 **线性四面体网格** —— 软体一体打印手指自动网格化为 tet。

### 1.6 时间积分 (动力学)

对动态问题:
$$\mathbf{M}\ddot{\mathbf{U}} + \mathbf{C}\dot{\mathbf{U}} + \mathbf{K}\mathbf{U} = \mathbf{F}_\text{ext}(t)$$

**变量含义**：
- $\mathbf{M}$ — 质量矩阵 (对角或一致)。
- $\mathbf{C}$ — 阻尼矩阵。
- $\dot{\mathbf{U}}, \ddot{\mathbf{U}}$ — 节点速度、加速度。
- $\mathbf{F}_\text{ext}(t)$ — 时变外力。

**公式解读**：质量项 + 阻尼项 + 弹性项 = 外力, 牛顿第二定律的有限元离散版。

#### 显式 Euler (简单)
$$\mathbf{U}_{t+1} = \mathbf{U}_t + dt\, \dot{\mathbf{U}}_t$$
$$\dot{\mathbf{U}}_{t+1} = \dot{\mathbf{U}}_t + dt\, \mathbf{M}^{-1}(\mathbf{F}_\text{ext} - \mathbf{C}\dot{\mathbf{U}}_t - \mathbf{K}\mathbf{U}_t)$$

**变量含义**：$dt$ 时间步长。**特点**: 简单, 但 $dt$ 必须很小才稳定。

#### 隐式 Euler (稳定, 大步长)
$$(\mathbf{M} + dt\,\mathbf{C} + dt^2\,\mathbf{K})\,\Delta\dot{\mathbf{U}} = dt\,(\mathbf{F}_\text{ext} - \mathbf{C}\dot{\mathbf{U}}_t - \mathbf{K}\mathbf{U}_t)$$

**特点**: 每步解一次大线性系统, 但允许大 $dt$。

本论文 Warp 用 **半隐式** 时间积分, 4000 fps × 850 frames per simulation。

---

## 2. 软体 FEM (大变形 + 超弹性)

### 2.1 变形梯度

参考构型 $\mathbf{X} \in \Omega_0$ (初始未变形位置) → 当前构型 $\mathbf{x} \in \Omega_t$ (变形后位置)。

$$\mathbf{F} = \frac{\partial \mathbf{x}}{\partial \mathbf{X}} \in \mathbb{R}^{3 \times 3}$$

**变量含义**：
- $\mathbf{F}$ — **变形梯度**, 描述局部如何变形的 3×3 矩阵。
- $J = \det(\mathbf{F})$ — 局部体积比 ($J = 1$ 不可压, $J > 1$ 膨胀, $J < 1$ 压缩, $J < 0$ 不物理)。

**公式解读**：每点附近的变形可由 $\mathbf{F}$ 完全描述, 比"位移梯度"更适合大变形。

### 2.2 不变量

Cauchy-Green 张量:
$$\mathbf{C} = \mathbf{F}^\top\mathbf{F},\quad \mathbf{B} = \mathbf{F}\mathbf{F}^\top$$

**变量含义**：
- $\mathbf{C}$ — **右 Cauchy-Green 张量** (参考构型上的)。
- $\mathbf{B}$ — **左 Cauchy-Green 张量** (当前构型上的)。

**主不变量**:
$$I_1 = \text{tr}(\mathbf{C}),\quad I_2 = \tfrac{1}{2}((\text{tr}\mathbf{C})^2 - \text{tr}(\mathbf{C}^2)),\quad I_3 = J^2$$

**变量含义**：$\text{tr}$ = 迹 (对角元素之和)。$I_1, I_2, I_3$ 与坐标系无关, 只描述形变本身。

### 2.3 Neo-Hookean 本构 (本论文用)

自由能密度:
$$\psi(\mathbf{F}) = \tfrac{\mu}{2}(I_1 - 3) - \mu \log J + \tfrac{\lambda}{2}\log^2 J$$

**变量含义**：
- $\psi$ — 单位参考体积内的弹性能 (J/m³)。
- $\mu = E / [2(1+\nu)]$ — **剪切模量** (Pa)。
- $\lambda = E\nu / [(1+\nu)(1-2\nu)]$ — **第一拉梅常数** (Pa)。
- $E$ — 杨氏模量 (Pa)。
- $\nu$ — 泊松比 (无量纲, 软体常 ≈ 0.49 近不可压)。

**公式解读**：
- 第一项 $\frac{\mu}{2}(I_1 - 3)$ — 与剪切变形相关, 无变形时 ($I_1 = 3$) 为 0。
- 第二项 $-\mu \log J$ — 防体积塌缩 ($J \to 0$ 时 $\to +\infty$)。
- 第三项 $\frac{\lambda}{2}\log^2 J$ — 体积变化的二次惩罚。

整体保证: 大变形 + 不可压趋势 + 反转能量发散。

### 2.4 应力计算

第一 Piola-Kirchhoff 应力:
$$\mathbf{P} = \frac{\partial \psi}{\partial \mathbf{F}} = \mu(\mathbf{F} - \mathbf{F}^{-\top}) + \lambda \log J\,\mathbf{F}^{-\top}$$

**变量含义**：
- $\mathbf{P} \in \mathbb{R}^{3 \times 3}$ — **PK1 应力**, 在 **参考构型** 上定义。
- $\mathbf{F}^{-\top} = (\mathbf{F}^{-1})^\top = (\mathbf{F}^\top)^{-1}$ — 逆转置。

Cauchy 应力:
$$\boldsymbol{\sigma} = \frac{1}{J}\mathbf{P}\mathbf{F}^\top$$

**公式解读**：把"参考构型应力"转换到"当前构型应力"。这是 FEM 实际计算时所用的应力。

### 2.5 节点力计算

每个四面体单元贡献节点力:
$$\mathbf{f}_i^e = -\int_{\Omega_e^0} \mathbf{P}\,\nabla N_i\, dV$$

**变量含义**：
- $\mathbf{f}_i^e \in \mathbb{R}^3$ — 单元 $e$ 在节点 $i$ 上的内力。
- $\Omega_e^0$ — 参考构型下单元 $e$ 体积。
- $\nabla N_i$ — 形函数 $N_i$ 的梯度。

**公式解读**：积分 = 应力沿单元体积的"功"。负号因为是 **反作用力** (从应力推得作用在节点上的内部反力)。组装到全局 $\mathbf{F}_\text{int}$, 与外力配合更新加速度。

---

## 3. 接触约束

### 3.1 KKT 互补条件

物理接触满足 (Karush-Kuhn-Tucker 条件):
$$0 \le g(\mathbf{u}) \perp \boldsymbol{\lambda} \ge 0$$

**变量含义**：
- $g(\mathbf{u})$ — **间隙函数**: 物体表面之间距离, $g \ge 0$ 表示无穿透。
- $\boldsymbol{\lambda} \ge 0$ — **接触力**: 永远是推 (不能拉)。
- $\perp$ — 互补条件: $g \cdot \lambda = 0$ (要么没接触 $g > 0, \lambda = 0$, 要么接触 $g = 0, \lambda > 0$)。

**公式解读**：用数学严格描述"物体不能互相穿透 + 接触力只能是推"。

### 3.2 Penalty 法 (本论文风格)

$$F_n = k_n \cdot \max(0, -g)$$

**变量含义**：
- $k_n$ — 接触刚度 (大数, 例如 $10^6$ N/m)。
- $\max(0, -g)$ — 穿透量 (穿透时为正, 不穿透时为 0)。

**公式解读**：穿透越深力越大, 像弹簧。简单但不严格 (允许小穿透)。

### 3.3 平滑可微版 (Xu 2021)

$$F_n = k_n \cdot \log(1 + e^{-g/\epsilon})$$

详见 [[可微分模拟 Differentiable Simulation]]。

---

## 4. 本论文 (Yi'25) 的 FEM 工作流

```
─── 步骤 1 · 几何与网格 ───────────────────────
软指 STL (CAD 设计)
    ↓ 自动 tetrahedralization (四面体化)
四面体网格 (~10⁴ tet, ~5×10³ nodes)
    ↓ 顶点对齐
tendon waypoints 处保证有顶点

─── 步骤 2 · 材料赋值 ─────────────────────────
22 个 block 各分配 log E ∈ [13.5, 17.0]
ν = 0.49 (近不可压 TPU)

─── 步骤 3 · 边界条件 ─────────────────────────
基座: Dirichlet 固定 (u = 0)
腱 waypoint: 施加切向力 (uniform tension)
        T_∥S_i = (I - n_S_i n_S_i^⊤) T_i
        T_i = f_T · (P_{i+1} - P_i) / ‖P_{i+1} - P_i‖

─── 步骤 4 · 时间积分 ─────────────────────────
Warp 半隐式, dt = 0.25 ms (4000 fps)
跑 850 frames (~ 0.21 s 物理时间)
动作: 拉腱 + 提起夹爪

─── 步骤 5 · 输出 ────────────────────────────
末态:
  · 物体合力 f ∈ ℝ⁶
  · 物体位姿偏移 Δq ∈ ℝ³
  · 是否触地 c_g ∈ {0,1}
```

---

## 5. FEM 的可微性

### 5.1 隐式微分

对 $\mathbf{K}\mathbf{U} = \mathbf{F}$ 关于参数 $\boldsymbol{\theta}$ 求导:
$$\frac{\partial \mathbf{K}}{\partial \boldsymbol{\theta}}\mathbf{U} + \mathbf{K}\frac{\partial \mathbf{U}}{\partial \boldsymbol{\theta}} = \frac{\partial \mathbf{F}}{\partial \boldsymbol{\theta}}$$

解出:
$$\frac{\partial \mathbf{U}}{\partial \boldsymbol{\theta}} = \mathbf{K}^{-1}\left[\frac{\partial \mathbf{F}}{\partial \boldsymbol{\theta}} - \frac{\partial \mathbf{K}}{\partial \boldsymbol{\theta}}\mathbf{U}\right]$$

**变量含义**：所有项都是数值矩阵, $\mathbf{K}^{-1}$ 是刚度矩阵的逆。

**公式解读**：节点位移对参数的敏感度可由 (a) 力对参数敏感度、(b) 刚度矩阵对参数敏感度 + 当前位移 推得。

### 5.2 伴随法 (避免显式求逆)

引入伴随变量 $\boldsymbol{\lambda}$, 解伴随方程:
$$\mathbf{K}^\top \boldsymbol{\lambda} = \frac{\partial J}{\partial \mathbf{U}}^\top$$

**变量含义**：
- $J$ — 目标函数 (例如指尖位移)。
- $\boldsymbol{\lambda}$ — 伴随向量, 维度等于 $\mathbf{U}$。

最终梯度:
$$\frac{dJ}{d\boldsymbol{\theta}} = \boldsymbol{\lambda}^\top \left[\frac{\partial \mathbf{F}}{\partial \boldsymbol{\theta}} - \frac{\partial \mathbf{K}}{\partial \boldsymbol{\theta}}\mathbf{U}\right]$$

**公式解读**：通过解一个与原方程同结构的伴随系统, 一次性算出 $J$ 对所有 $\theta_i$ 的梯度——计算成本与设计变量数无关。

### 5.3 接触场景的失效

接触切换 → $\mathbf{K}$ 不连续变化 → 伴随梯度抖动。**本论文为此不直接对 FEM 求梯度**, 而是用 NN 代理。

---

## 6. 仿真器对照

| 工具 | 接口 | 物理 | GPU | AD |
|---|---|---|---|---|
| ABAQUS / ANSYS | 商业 | 完备 | 部分 | ❌ |
| **Warp** ([[../03-参考文献/Ref-18-Macklin-2022-Warp]]) | Python | FEM/PBD | ✅ | ✅ |
| **DiffTaichi** ([[../03-参考文献/Ref-19-Hu-2020-DiffTaichi]]) | DSL | MPM/FEM | ✅ | ✅ |
| **SOFA** ([[../03-参考文献/Ref-41-Faure-2012-SOFA]]) | C++/Python | 软体 + 约束 | 部分 | 实验性 |
| FEniCS / deal.II | Python/C++ | 通用 PDE | 部分 | 部分 |

---

## 7. 关联概念

- [[可微分模拟 Differentiable Simulation]]
- [[神经物理模拟器 Neural Physics]]
- [[腱驱动 Tendon-driven]]
- [[刚度分布 Stiffness Distribution]]

## 8. 关联文献

- [[../03-参考文献/Ref-18-Macklin-2022-Warp]]
- [[../03-参考文献/Ref-19-Hu-2020-DiffTaichi]]
- [[../03-参考文献/Ref-41-Faure-2012-SOFA]]
- [[../03-参考文献/Ref-12-Chen-2018-拓扑优化软夹爪]] (FEM + 拓扑优化)
- [[../03-参考文献/Ref-50-Craig-2020-MaterialsMechanics]] (材料力学基础)

---

*← 返回 [[../00-总览/主索引 MOC]]*
