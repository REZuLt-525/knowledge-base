# 软体机器人 Soft Robotics

> **一句话理解**：主体由 **柔顺材料** ($E \le 100$ MPa, 多为弹性体) 构成的机器人，具有连续介质、无穷自由度、被动顺应等特征。其建模需大变形非线性力学，控制需考虑欠驱动 + 形态计算。

`#核心概念`

---

## 1. 量化定义

### 1.1 材料模量分类 (Rus & Tolley 2015)

$$\text{Soft}: E \le 1 \text{ MPa} \quad\to\quad \text{Compliant}: 1 - 100 \text{ MPa} \quad\to\quad \text{Rigid}: E \gg 100 \text{ MPa}$$

**变量含义**：
- $E$ — **杨氏模量** (Pa, 材料硬度的标准衡量)。
- 软橡胶 $E \approx 0.07$ MPa (Ecoflex)。
- TPU $E \approx 30-50$ MPa。
- 钢 $E \approx 200$ GPa = $2 \times 10^{11}$ Pa。

本论文夹爪材料 NinjaFlex Cheetah TPU 95A: $E \approx 30-50$ MPa → **柔顺类 (compliant)** 而非严格的 "soft"。

---

## 2. 软体机器人的关键特征

### 2.1 连续介质动力学

不是有限多刚体连接, 而是 PDE 描述的连续场:

$$\rho \ddot{\mathbf{u}} = \nabla \cdot \boldsymbol{\sigma}(\mathbf{u}) + \mathbf{b}$$

**变量含义**：
- $\rho$ (kg/m³) — 材料密度。
- $\mathbf{u}(\mathbf{x}, t) \in \mathbb{R}^3$ — **位移场**: 每个空间点 $\mathbf{x}$ 在时间 $t$ 的位移向量。
- $\ddot{\mathbf{u}}$ — 二阶时间导, 即加速度场。
- $\boldsymbol{\sigma}$ — **应力张量** (3×3 矩阵), 由本构关系给出。
- $\nabla \cdot \boldsymbol{\sigma}$ — 应力散度 (向量), 表示净力。
- $\mathbf{b}$ — 体力 (重力等)。

**公式解读**：连续介质形式的 **牛顿第二定律**: 单位体积的"质量乘以加速度 = 应力净作用 + 外体力"。每点都满足。

需要 [[FEM 有限元法]] 等数值方法离散求解。

### 2.2 大变形非线性

变形 > 10% 时小应变假设失效, 需用 **变形梯度** $\mathbf{F} = \partial \mathbf{x}/\partial \mathbf{X}$ 与超弹性本构 (Neo-Hookean, Mooney-Rivlin, Ogden)。

#### Neo-Hookean (本论文用)

$$\psi(\mathbf{F}) = \tfrac{\mu}{2}(\text{tr}(\mathbf{F}\mathbf{F}^\top) - 3) - \mu \log J + \tfrac{\lambda}{2}\log^2 J$$

**变量含义**：
- $\psi$ — 单位参考体积内的弹性自由能 (J/m³)。
- $\mathbf{F} \in \mathbb{R}^{3 \times 3}$ — 变形梯度。
- $J = \det(\mathbf{F})$ — 体积比。
- $\mu, \lambda$ — 拉梅常数, 由 $E, \nu$ 决定。
- $\text{tr}$ — 矩阵迹。

**公式解读**：超弹性材料的应力来自这个能量函数对 $\mathbf{F}$ 求导。具体含义见 [[FEM 有限元法]]。

### 2.3 欠驱动 + 形态计算

#### 欠驱动 (underactuated)

$$\dim(\text{actuator}) \ll \dim(\text{configuration})$$

**变量含义**：
- $\dim(\text{actuator})$ — 主动电机 / 执行器数量。
- $\dim(\text{configuration})$ — 系统总自由度 (软体连续介质理论上 $\infty$)。

**公式解读**：少马达驱动多关节, 物体接触决定其余形状 → 被动适配。

#### 形态计算 (morphological computation)

让结构本身完成"计算":
- 均匀压力腱布线 ([[../03-参考文献/Ref-42-Hirose-1978-UniformPressure|Hirose 1978]]): 几何决定贴合行为。
- 双稳态阀 ([[../03-参考文献/Ref-33-Rothemund-2018-BistableValve|Rothemund 2018]]): 物理结构编码控制逻辑。

---

## 3. 主流驱动机制

### 3.1 气动 (Pneumatic)

PneuNet 经典结构: 多气腔不对称膨胀 → 弯曲。

气压 → 力学:
$$F_\text{actuation} = P \cdot A_\text{eff}$$

**变量含义**：
- $P$ (Pa) — 气压 (~10-100 kPa)。
- $A_\text{eff}$ (m²) — 有效作用截面积。
- $F$ (N) — 总力。

**公式解读**：力 = 压强 × 面积, 标准定义。

### 3.2 腱驱动 (Tendon-driven)

电机远端拉绳, 末端结构变形。详见 [[腱驱动 Tendon-driven]]。本论文方案。

### 3.3 电活性聚合物 (EAP)

电场 → 形变。**介电弹性体执行器 (DEA)** 为代表:
$$\sigma_\text{Maxwell} = \epsilon_0 \epsilon_r E_\text{field}^2$$

**变量含义**：
- $\sigma_\text{Maxwell}$ (Pa) — Maxwell 应力 (电场产生)。
- $\epsilon_0 = 8.85 \times 10^{-12}$ F/m — 真空介电常数。
- $\epsilon_r$ — 相对介电常数 (~3-10 for polymer)。
- $E_\text{field}$ (V/m) — 电场强度 (~$10^7$ V/m, 即每毫米千伏)。

**公式解读**：电场作用导致材料压缩 → 平面方向膨胀 (类似气球放气)。需要高压 (kV)。

### 3.4 形状记忆合金 / 聚合物 (SMA / SMP)

加热 → 相变 → 形变。响应慢但能量密度高。

### 3.5 磁驱动

外加磁场 → 嵌入磁颗粒受力。无线驱动, 适合体内机器人。

---

## 4. 制造工艺总览

详细见 [[../03-参考文献/Ref-01-Rus-2015-软体机器人综述]] §制造工艺章节。简表:

| 工艺 | 设备 | 材料 | 分辨率 | 周期 |
|---|---|---|---|---|
| Soft lithography | 模具 + 真空 | PDMS / 硅胶 | 10 μm | 天-周 |
| SDM | CNC + 浇注 | 聚氨酯 | 1 mm | 小时 |
| **FDM 3D 打印** | 桌面 | TPU | 100 μm | 小时 |
| DLP / SLA | 桌面 | 软树脂 | 50 μm | 小时 |
| PolyJet | 工业 | 多树脂 | 30 μm | 小时 |
| 激光切割 + 折叠 | 激光机 | PET / 纸 | 10 μm | 分钟 |

本论文用 FDM + NinjaFlex Cheetah, 因为成本低 + 桌面 + co-design 闭环友好。

---

## 5. 建模与控制路线总览

### 5.1 PCC (常曲率假设)

每段视为常曲率弧:
$$(\kappa_i, \phi_i, l_i)$$

**变量含义**：
- $\kappa_i$ (1/m) — 曲率 (倒数 = 半径)。
- $\phi_i$ (rad) — 方位角。
- $l_i$ (m) — 弧长。

**公式解读**：把无穷自由度软臂用 $3N$ 个参数描述, $N$ 段。逆运动学有闭式解, 但物体接触下不成立。

### 5.2 Cosserat 杆

把软臂视作一维连续介质杆, 静力平衡:
$$\frac{d\mathbf{n}}{ds} + \mathbf{f} = 0,\;\; \frac{d\mathbf{m}}{ds} + \mathbf{r}'\times\mathbf{n} + \boldsymbol{\tau} = 0$$

**变量含义**：
- $s$ — 弧长参数 (沿杆方向)。
- $\mathbf{n}(s) \in \mathbb{R}^3$ — 内力 (剪力 + 轴力)。
- $\mathbf{m}(s) \in \mathbb{R}^3$ — 内力矩 (弯矩 + 扭矩)。
- $\mathbf{f}, \boldsymbol{\tau}$ — 外分布力 / 力矩。
- $\mathbf{r}'$ — 杆切向。
- $\times$ — 向量叉乘。

**公式解读**：连续杆的力 / 力矩平衡方程。比 PCC 更精确 (变形可任意), 但求解需 BVP / 数值积分。

### 5.3 FEM (本论文)

完整 3D 离散 + Neo-Hookean。详见 [[FEM 有限元法]]。

### 5.4 数据驱动 / 神经代理

直接从数据学映射, 跳过物理推导。详见 [[神经物理模拟器 Neural Physics]]。

### 5.5 残差物理

物理基线 + NN 学差距。详见 [[../03-参考文献/Ref-11-Gao-2024-ResidualPhysics]]。

---

## 6. 软体 vs 刚性机器人 对比

| 指标 | 软体 | 刚性 |
|---|---|---|
| 自由度 | $\infty$ (连续) | 有限 (离散) |
| 弹性模量 | $\le 100$ MPa | $\ge 100$ GPa |
| 形状适应性 | 高 | 低 |
| 力控精度 | 低 (~1 N) | 高 (~0.01 N) |
| 安全性 (人机交互) | 高 | 低 |
| 速度 | 慢 (~Hz) | 快 (~kHz) |
| 寿命 | 中 (材料疲劳) | 长 |
| 制造成本 | 低 | 高 |

---

## 7. 关联概念

- [[FEM 有限元法]]
- [[腱驱动 Tendon-driven]]
- [[柔性铰链 Flexure Joint]]
- [[刚度分布 Stiffness Distribution]]
- [[神经物理模拟器 Neural Physics]]

## 8. 关联文献

- [[../03-参考文献/Ref-01-Rus-2015-软体机器人综述]]
- [[../03-参考文献/Ref-02-Shintake-2018-软体夹爪综述]]
- [[../03-参考文献/Ref-31-Piazza-2019-机器手百年]]
- [[../03-参考文献/Ref-13-Chen-2020-软体设计综述]]

---

*← 返回 [[../00-总览/主索引 MOC]]*
