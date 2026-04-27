# [50] Craig Jr. & Taleff 2020 · 材料力学 (教材)

> **原文**: R. R. Craig Jr. and E. M. Taleff. *Mechanics of Materials.* John Wiley & Sons, 2020.
> **译名**: 《材料力学》 — John Wiley & Sons 出版社, 2020.

`#文献-教材` `#材料力学`

---

## 1 · 教材定位

材料力学 (Mechanics of Materials) 是工科本科必修课 — 研究 **杆、梁、轴** 在各种载荷 (拉、压、弯、扭) 下的应力、应变、变形。Craig & Taleff 的版本是工程院校广泛采用的标准教材。

---

## 2 · 与本论文 (Yi'25) 相关的核心知识

### 2.1 应力 - 应变线弹性 (Hooke 定律)

一维:
$$\sigma = E \cdot \varepsilon$$

**变量含义**：
- $\sigma$ (Pa) — **应力**: 单位面积上的力 (N/m²)。
- $E$ (Pa) — **杨氏模量**: 材料"硬度"标志, 定义为应力与应变之比。
- $\varepsilon$ (无量纲) — **应变**: 相对变形 = $\Delta L / L$。

**公式解读**：材料在小变形下, 应力与应变成正比 (类似弹簧 $F = kx$ 的连续介质版本)。$E$ 大 = 硬, $E$ 小 = 软。

**典型 $E$ 数值**:
- 软橡胶: $\sim 1$ MPa = $10^6$ Pa。
- TPU: $\sim 30$ MPa。
- 木材: $\sim 10$ GPa。
- 钢: $\sim 200$ GPa = $2 \times 10^{11}$ Pa。

跨越 5 个数量级!

### 2.2 三维各向同性弹性

$$\boldsymbol{\sigma} = \mathbf{D}(E, \nu) \cdot \boldsymbol{\varepsilon}$$

**变量含义**：
- $\boldsymbol{\sigma}, \boldsymbol{\varepsilon}$ — 应力 / 应变张量 (3×3, 用 6 维向量 Voigt 形式表示)。
- $\mathbf{D}$ — 6×6 弹性常数矩阵, 由 $E, \nu$ 决定。
- $\nu$ — **泊松比** (无量纲, 横向应变 / 纵向应变)。

**公式解读**：3D 推广。各向同性材料只需两个常数 $E, \nu$ 完全描述弹性响应。

### 2.3 梁弯曲基本方程

Euler-Bernoulli 梁:

$$\frac{d^2 v}{dx^2} = \frac{M(x)}{EI}$$

**变量含义**：
- $x$ (m) — 梁轴向坐标。
- $v(x)$ (m) — 梁挠度 (垂直方向位移)。
- $M(x)$ (N·m) — 弯矩。
- $EI$ (N·m²) — **抗弯刚度**: 等于杨氏模量 × 截面惯性矩。
- $I$ (m⁴) — **截面惯性矩**, 矩形为 $bh^3/12$ ($b$ 宽, $h$ 厚)。

**公式解读**：挠度的二阶导 (即曲率) 与 $M / (EI)$ 成正比 — 弯矩越大、$EI$ 越小 → 弯曲越厉害。

### 2.4 三点弯曲测试 (本论文用) ⭐

跨距 $L$, 中间施加力 $P$:
$$\boxed{\delta = \frac{PL^3}{48 EI}}$$

反求弹性模量:
$$\boxed{E = \frac{P L^3}{48 I \delta}}$$

**变量含义**：
- $P$ (N) — 中点向下加载力。
- $L$ (m) — 两支撑点间距 (跨距)。
- $I = bh^3/12$ (m⁴) — 截面惯性矩。
- $\delta$ (m) — 中点向下挠度。

**公式解读**:
- 公式来自 Euler-Bernoulli 梁方程 + 边界条件求解。
- $L^3$ 关系让长跨距更敏感 (放大效应)。
- $I$ 大 (粗梁) → 同样力下挠度小。

**本论文应用**: 测软指 flexure 块的等效模量 — 给定打印参数的样本, 测 $\delta$ 反求 $E$。

### 2.5 压痕测试 (本论文 segment 块测量)

对柱状压头压入弹性体 (Boussinesq 解):

$$\boxed{E = \frac{(1-\nu^2)P}{2 a \delta}}$$

**变量含义**：
- $\nu \approx 0.49$ — 泊松比 (近不可压 TPU)。
- $a$ (m) — 压头半径。
- $P$ (N) — 加载力。
- $\delta$ (m) — 压头下沉深度。
- $(1-\nu^2)$ — 多向应变修正因子。

**公式解读**：
- $\nu^2$ 修正因为压痕涉及多向应变 (压头下方材料水平方向也变形)。
- 不同于三点弯曲, 压痕是 **局部测试** (适合厚块)。

**本论文应用**: 测 segment 块 (厚, 不易做三点弯曲) 的等效模量。

### 2.6 缩放律

如果几何 **完全相似** (所有尺寸按比例 $k$ 缩放), 但材料 $E$ 不同:

同样载荷下变形量与 $E$ 反比:
$$\delta \propto \frac{1}{E}$$

**公式解读**：变形与 $E$ 反比 (从 $\delta = PL^3/(48 EI)$ 看 $\delta \propto 1/E$)。

**关键推论**: **相对刚度** (即 $E_i$ 之间的比例) 决定变形比例, 而不是 $E$ 的绝对值。

---

## 3 · 公式应用场景: 本论文 §6.3 ⭐

### 3.1 引用句

本论文 §6.3 写道:
> "Since all of the flexure blocks and segment blocks have the same geometry, respectively, the deformation behavior **mainly depends on relative stiffness** [50]."

### 3.2 数学含义

由于所有 flexure 块几何相同, 所有 segment 块几何相同, 设它们的等效模量分别为 $\{E_\text{flex}^{(i)}\}, \{E_\text{seg}^{(j)}\}$。给定外力 $P$, 第 $i$ 个 flexure 块的变形:

$$\delta_\text{flex}^{(i)} = \frac{P L^3}{48 E_\text{flex}^{(i)} \cdot I}$$

整指变形是各段叠加:
$$\delta_\text{tip} = \sum_i \delta_\text{flex}^{(i)} + \sum_j \delta_\text{seg}^{(j)} \propto \sum_i \frac{1}{E_\text{flex}^{(i)}} + \sum_j \frac{1}{E_\text{seg}^{(j)}}$$

**公式解读**：整指行为只取决于 $\{1/E^{(i)}\}$ 的 **相对值**, 不依赖绝对值。

### 3.3 实践含义 (本论文 Sim-to-Real)

```
仿真:    log E ∈ [13.5, 17.0]  (相对值)
                ↓
实测打印参数 ↔ 真实 E:
    flexure 块: 三点弯曲 → E_real_flex
    segment 块: 压痕 → E_real_seg
                ↓
线性映射:
    log E_print = α · log E_sim + β
                ↓
3D 打印: 选最匹配的打印参数实现 E_print
                ↓
真机部署
```

由于整指行为对相对刚度敏感, 这条 **线性映射策略** 即可实现 Sim-to-Real。

---

## 4 · 教材其他相关章节

### 4.1 屈服与失效

Von Mises 等效应力:
$$\sigma_\text{vM} = \sqrt{\frac{(\sigma_1 - \sigma_2)^2 + (\sigma_2 - \sigma_3)^2 + (\sigma_3 - \sigma_1)^2}{2}}$$

**变量含义**：
- $\sigma_1, \sigma_2, \sigma_3$ — 三个 **主应力** (应力张量特征值)。
- $\sigma_\text{vM}$ — 等效单轴拉应力。

**公式解读**：把三向应力综合为一个等效拉应力, 方便用单轴屈服准则比较。

屈服判据: $\sigma_\text{vM} \ge \sigma_y$ (材料屈服强度)。

**用途**: 设计需保证软指最大应力 < TPU 屈服应力 (~10 MPa)。

### 4.2 屈曲 (Buckling)

Euler 临界载荷:
$$P_\text{cr} = \frac{\pi^2 EI}{(K l)^2}$$

**变量含义**：
- $P_\text{cr}$ (N) — 临界压载 (超此值梁会突然弯曲)。
- $K$ — 边界条件系数 (两端铰支为 1, 一端固定一端自由为 2)。
- $l$ — 梁长。

**公式解读**：细长梁压缩时不会无限承载, 而是在临界值发生屈曲。$EI$ 大、$l$ 小 → 抗屈曲强。

**用途**: 防止细长指节在压载下失稳。

### 4.3 大变形 (本教材较少, 详见专门书籍)

工程应变 vs Lagrangian / Almansi 应变定义。
本论文用 Neo-Hookean 大变形本构, 详见 [[FEM 有限元法]]。

---

## 5 · 关键贡献 (作为教材)

1. **系统化基础**: 让工程师有统一数学语言。
2. **覆盖完整**: 从应力分析到失效判据全包。
3. **示例丰富**: 每章数十个解题示例。
4. **现代版本**: 包含计算工具 (FEM 简介)。

---

## 6 · 与本论文 (Yi'25) 的关系

本论文引用此教材的 **唯一目的**: 解释为何 "**相对刚度** 决定行为", 从而合法化 **线性 Sim-to-Real 映射** 策略。

具体: 缩放律 + 三点弯曲 / 压痕公式 → 让 22 维 log E 仿真值能 **离散映射** 到 22 套打印参数, 而无需重新校准每个 block 的绝对模量。

这是本论文 Sim-to-Real 简洁性的理论基础。

---

*← [[_参考文献索引]]*
