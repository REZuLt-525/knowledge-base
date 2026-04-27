# 柔性铰链 Flexure Joint

> **一句话理解**：通过 **薄壁段的弹性变形** 替代传统轴-销旋转副实现关节运动的机构元件。其等效转动刚度由材料模量 $E$、截面几何 ($b, h$)、长度 $l$ 决定，可用 Pseudo-Rigid-Body Model (PRBM) 近似为"刚性 link + 扭簧"。

`#核心概念`

---

## 1. 几何参数

最简化矩形截面薄梁:
```
    宽 b
  ┌───────┐
  │       │ 厚 h
  │       │
  └───────┘
       长 l
```

---

## 2. 等效弯曲刚度 (欧拉-伯努利梁)

弯曲角 $\theta$ 与外加弯矩 $M$ 的线性关系:

$$M = k_\theta \cdot \theta$$

**变量含义**：
- $M$ (N·m) — 外加弯矩。
- $\theta$ (rad) — 弯曲角度。
- $k_\theta$ (N·m/rad) — **等效转动刚度**, 类似扭簧的"硬度系数"。

**公式解读**：胡克定律的旋转版——弯矩与角度成正比。

### 2.1 等效刚度公式

$$\boxed{k_\theta = \frac{EI}{l} = \frac{E \cdot b h^3}{12\,l}}$$

**变量含义**：
- $E$ (Pa) — **杨氏模量** (材料属性), TPU 95A ≈ 30-60 MPa。
- $I$ (m⁴) — **截面惯性矩**, 矩形为 $bh^3/12$。
- $b$ (m) — 截面宽度。
- $h$ (m) — 截面厚度 (沿弯曲方向)。
- $l$ (m) — 薄段长度。

**公式解读**：
- $E$ 越大 → 材料越硬 → $k_\theta$ 越大。
- $h$ 越厚 → 抗弯越强, 但 **$h^3$ 关系** 让厚度影响极敏感 (厚度翻倍, 刚度 8 倍)。
- $l$ 越长 → 越易弯 (杠杆效应), $k_\theta$ 越小。

### 2.2 关键观察: $h^3$ 灵敏度

$$k_\theta \propto h^3$$

厚度小变化 → 刚度大变化。本论文固定 $h = 2$ mm, **通过 $E$ 调刚度** (而非 $h$, 因为 $h$ 是制造层面较固定的几何)。

---

## 3. Pseudo-Rigid-Body Model (PRBM)

Howell ([[../03-参考文献/Ref-39-Howell-2013-CompliantMechanisms]]) 提出的近似: 把柔性段等效为 **刚体 link + 扭簧**:

```
柔性段 (实际)              PRBM 等效
══════════                 ●─────●
                            扭簧 k_θ
```

### 3.1 PRBM 转动刚度公式

$$k_\theta^\text{PRBM} = \gamma \cdot K_\Theta \cdot \frac{EI}{l}$$

**变量含义**：
- $\gamma \approx 0.85$ — **长度系数**: PRBM 等效连杆相对真实长度的比例 (查 Howell 表)。
- $K_\Theta \approx 2.65$ — **刚度系数**: 反映分布柔性集中到一处后的等效。

**公式解读**：精确等效需查表, 但本质形式与欧拉-伯努利公式相同, 多个修正系数。

### 3.2 PRBM 适用范围

- $\theta \le 60°$: 误差 < 0.5%。
- $\theta > 60°$: 需用椭圆积分精确解或 FEM。

**用途**：让运动学/动力学分析像传统刚性连杆一样做。

---

## 4. 大变形修正

### 4.1 椭圆积分 (精确解)

精确弯曲方程:
$$M(s) = EI \frac{d\phi}{ds}$$

**变量含义**：
- $\phi(s)$ — 弧长参数化下的切向角。
- $s$ — 弧长参数 (沿梁的曲线坐标)。

**公式解读**：弯矩 = $EI$ × 曲率, 这是 Euler-Bernoulli 梁理论的基本方程。

求解涉及椭圆积分, 需数值方法。

### 4.2 FEM (本论文路线)

直接用四面体网格 + Neo-Hookean 本构, 自动捕捉大变形。详见 [[FEM 有限元法]]。

---

## 5. 与本论文 (Yi'25) 软指结构

### 5.1 整体架构

软指由 **flexure block (薄)** 与 **segment block (厚)** 交替组成:

```
[seg] [flx] [seg] [flx] [seg] [flx] [seg] [flx] [seg] [flx] [seg]
 厚    薄    厚    薄    厚    薄    厚    薄    厚    薄    厚
 11 块, 其中 5 个 flexure + 6 个 segment
```

每根手指 11 块 × 2 指 = 22 块, 每块独立分配 $E$ → **22 维刚度向量** $\mathbf{k} \in \mathbb{R}^{22}$。

### 5.2 几何尺寸 (论文 Fig 2a)

- Flexure 块: 厚度 $h_\text{flex} = 2$ mm, 长度 $l_\text{flex} \sim 5$ mm。
- Segment 块: 较厚 (>10 mm), 几乎不弯, 提供刚性支撑。
- 整指长 $L = 60$ mm。

### 5.3 单根 flexure 的等效刚度

对 flexure 块 ($b = 10$ mm, $h = 2$ mm, $l = 5$ mm):
$$k_\theta = \frac{E \cdot 10 \cdot 2^3}{12 \cdot 5} \approx 1.33\,E\;\text{(单位: N·mm/rad)}$$

**示例**:
- $E = 1$ MPa: $k_\theta \approx 1.33$ N·mm/rad (软)。
- $E = 24$ MPa: $k_\theta \approx 32$ N·mm/rad (硬)。

24× 范围对应仿真中 $\log E \in [13.5, 17.0]$ 的设计空间。

### 5.4 段块 (segment) 的"刚度"含义不同

Segment 块虽厚, 但本论文在仿真中也分配 $E$ — 影响压痕响应、整体形变量。论文用 **压痕测试** 标定其等效模量, 与 flexure 区别处理 (见下节)。

---

## 6. 制造: 3D 打印结构调刚度

本论文 NinjaFlex Cheetah TPU 95A 的 **基础模量** $E_0 \approx 30-50$ MPa, 但通过结构参数可在大范围内调节 **等效模量**:

### 6.1 Flexure 块 (薄段)

- **Wall loops** (壳层数): 1 (最软) → 多 (硬)。
- **Bottom shell layers** (底层数): 类似。

更多壁 = 实心比例高 = 等效更硬。

### 6.2 Segment 块 (厚段)

- **Infill type**: grid / gyroid / honeycomb。
- **Infill density**: 10% (软) - 100% (硬)。
- **Infill direction**: 影响各向异性。

### 6.3 实测 (论文 Fig 5b/c)

- Flexure 三点弯曲: $E_\text{flex} \in [\sim 5, \sim 50]$ MPa (随壁数)。
- Segment 压痕: $E_\text{seg} \in [\sim 1, \sim 60]$ MPa。

仿真 $\log E$ 范围 $[13.5, 17.0]$ 经过线性映射对应到打印参数。

---

## 7. 测量公式 (本论文 §4.2)

### 7.1 三点弯曲测试 (Flexure 块)

跨距 $L$, 中点加载 $P$, 中点挠度 $\delta$:

$$\boxed{\delta = \frac{P L^3}{48 EI}}$$

反求杨氏模量:
$$E = \frac{P L^3}{48 I \delta}$$

**变量含义**：
- $P$ (N) — 中点加载力。
- $L$ (m) — 两支撑点间距离 (跨距)。
- $I = b h^3/12$ (m⁴) — 截面惯性矩。
- $\delta$ (m) — 中点向下挠度。

**公式解读**：标准的"梁中点挠度"公式, 三点弯曲是基础材料力学测试方法。

### 7.2 压痕测试 (Segment 块)

球状压头 (半径 $a$) 压入弹性半空间, 力 $P$ 与下压量 $\delta$:

$$\boxed{E = \frac{(1-\nu^2) P}{2 a \delta}}$$

**变量含义**：
- $\nu \approx 0.49$ — 泊松比 (近不可压软材料)。
- $a$ (m) — 压头半径。
- $\delta$ (m) — 压头下压距离。

**公式解读**：来自 Boussinesq 解 (经典弹性力学)。$\nu^2$ 修正因为压痕涉及多向应变。

---

## 8. 柔性铰链 vs 传统铰链

| 维度 | 柔性 | 传统 (轴+套) |
|---|---|---|
| 零件数 | 1 | 多 |
| 摩擦 | 几乎零 | 有 |
| 寿命 | $10^4-10^6$ 循环 | $10^7+$ |
| 行程 | $\le 60°$ (大变形 90°+) | $\ge 360°$ |
| 制造 | 一体成型 | 装配 |
| 成本 | 低 (3D 打印) | 高 |

---

## 9. 关联概念

- [[腱驱动 Tendon-driven]]
- [[刚度分布 Stiffness Distribution]]
- [[FEM 有限元法]]
- [[软体机器人 Soft Robotics]]

## 10. 关联文献

- [[../03-参考文献/Ref-39-Howell-2013-CompliantMechanisms]] (PRBM 理论)
- [[../03-参考文献/Ref-10-Odhner-2014-SDMhand]] (SDM Hand 柔性关节)
- [[../03-参考文献/Ref-40-Dollar-2010-SDMhand]] (SDM 早期)
- [[../03-参考文献/Ref-50-Craig-2020-MaterialsMechanics]] (材料力学基础)

---

*← 返回 [[../00-总览/主索引 MOC]]*
