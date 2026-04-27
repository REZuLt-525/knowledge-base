# 抓取位姿 Grasp Pose

> **一句话理解**：夹爪相对世界 (或相机) 的 **6-DoF 位姿** $g = (t, R, w) \in \mathbb{R}^3 \times SO(3) \times \mathbb{R}_+$，决定 "在哪里、怎么对齐、张多大" 去夹物体。

`#核心概念`

---

## 1. 形式化定义

### 1.1 6-DoF Grasp 表达

$$g = (\mathbf{t}, \mathbf{R}, w)$$

**变量含义**：
- $\mathbf{t} \in \mathbb{R}^3$ — **夹爪中心位置** (世界坐标系下)。
- $\mathbf{R} \in SO(3)$ — **夹爪朝向** 旋转矩阵 (常用 quaternion / Euler / 6D 向量参数化)。
- $w \in [w_\min, w_\max]$ — **夹爪张开宽度** (开合距离)。

**公式解读**：完整描述夹爪在 3D 空间的姿态: 位置 (3) + 朝向 (3) + 开合 (1) = 7 维。

部分系统额外包含:
- **接近向量** $\mathbf{a}$: 夹爪从哪个方向接近物体。
- **闭合向量** $\mathbf{c}$: 夹爪手指闭合方向。

### 1.2 仿真表示

本论文中夹爪附加 6D 自由 + 2 个 prismatic (开合):
$$\xi = (t_x, t_y, t_z, r_r, r_p, r_y, s_1, s_2) \in \mathbb{R}^8$$

**变量含义**：
- $(t_x, t_y, t_z)$ — 位置 3 维。
- $(r_r, r_p, r_y)$ — Roll, Pitch, Yaw 三个欧拉角。
- $(s_1, s_2)$ — 两根手指的开合距离。

---

## 2. 抓取候选生成

### 2.1 经典解析方法

#### Force Closure (力闭合)

选 $N$ 个接触点 $\{(\mathbf{p}_i, \mathbf{n}_i)\}_{i=1}^N$ 使力螺旋张成 $\mathbb{R}^6$ 空间。

数学条件:
$$\text{rank}(\mathbf{W}) = 6 \quad \text{且} \quad \mathbf{0} \in \text{relative interior of } \text{conv}\{\text{columns of }\mathbf{W}\}$$

**变量含义**：
- $\mathbf{W} \in \mathbb{R}^{6 \times N}$ — **Wrench 矩阵**, 每列是接触点 $i$ 在 6D 力螺旋空间的贡献。
- $\mathbf{p}_i$ — 接触点位置。
- $\mathbf{n}_i$ — 接触法线 + 摩擦锥。
- $\text{conv}$ — 凸包。
- $\mathbf{0}$ — 6D 零向量。

**公式解读**：
- 条件 1 ($\text{rank} = 6$): 6 个独立方向都能产生力。
- 条件 2 (0 在凸包内): 接触力可以平衡 **任何外力扰动** (没有任何方向逃跑)。

满足两条件 = 抓取力闭合 = 物体被任意约束。

### 2.2 数据驱动方法 (本论文用)

**AnyGrasp** ([[../03-参考文献/Ref-46-Fang-2023-AnyGrasp]]) 工作流:
```
RGB-D → 点云 P
    ↓ 3D CNN / PointNet++
体素特征
    ↓
   ┌─── ApproachNet → 抓取主方向 a ∈ S²
   ├─── OperationNet → 旋转角 + 宽度 (旋转主轴 + w)
   └─── ScoreNet → 抓取置信度 s ∈ [0,1]
输出 K 个候选 {g_k, s_k}
```

---

## 3. 本论文位姿采样工作流

```
─── 步骤 1 · AnyGrasp 候选 ──────────────────
RealSense × 2 → 点云 → AnyGrasp → 候选位姿 {g_k}
    问题: 默认夹爪短, 候选位姿与本论文长指碰撞

─── 步骤 2 · SDF 位姿增广 ────
对每个 AnyGrasp 候选 g, 加随机扰动:
    t' = t + N(μ_t, σ_t)
    r' = r + N(μ_r, σ_r)
然后局部优化 SDF 损失 (见下方公式)

─── 步骤 3 · 神经代理评分 ────────────────────
for each augmented pose p ∈ P:
    ℓ_p = NN_θ(p, k, obj)               # 1 ms / pose
    score 包含 (force, displacement, ground_contact)
按 ℓ_p 排序, 取 Top-B = 5。

─── 步骤 4 · 联合优化 / 部署 ─────────────────
[优化] 把 Top-B 损失之和反传到刚度 k 上。
[部署] 选最低 ℓ_p 对应的位姿 p*, 由 Franka 执行。
```

### SDF 位姿增广损失 (本论文 Eq. 4 init)

$$L_\text{init} = w_d \sum_{i} \max(d(i), 0) + w_p \sum_{i} \max(-d(i), 0)$$

**变量含义**：
- $i$ — 夹爪 mesh 上采样的顶点索引。
- $d(i)$ — **SDF 值**: 顶点 $i$ 到物体或地面的有符号距离 (正 = 在外部, 负 = 穿透)。
- $w_d$ — **靠近权重** (小, 例如 0.1)。
- $w_p$ — **穿透惩罚权重** (大, 例如 100)。

**公式解读**：
- **第 1 项 $\max(d, 0)$**: 当 $d > 0$ (在物体外) 时 = $d$ (鼓励靠近), $d \le 0$ (已穿透) 时 = 0。
- **第 2 项 $\max(-d, 0)$**: 当 $d < 0$ (穿透) 时 = $|d|$ (惩罚穿透), $d \ge 0$ 时 = 0。

**直觉**：
- $w_p \gg w_d$: 优先无碰撞 (硬约束) > 贴合物体 (软约束)。
- 一起最小化 → 找到一个"靠近物体但不穿透"的位姿。

优化变量: $t_y, s_1, s_2$ (vertical + opening), 因为深度方向和开合最影响碰撞。

---

## 4. SDF (有符号距离函数) 详解

### 4.1 定义

对刚体物体 $O$:
$$d(\mathbf{x}; O) = \begin{cases}
+\min_{\mathbf{y} \in \partial O} \|\mathbf{x} - \mathbf{y}\|, & \mathbf{x} \notin O \\
-\min_{\mathbf{y} \in \partial O} \|\mathbf{x} - \mathbf{y}\|, & \mathbf{x} \in O
\end{cases}$$

**变量含义**：
- $\mathbf{x}$ — 查询点 (3D 空间任意位置)。
- $O$ — 物体 (3D 实体)。
- $\partial O$ — 物体表面。
- $\mathbf{y}$ — 表面上点。
- $\|\cdot\|$ — 欧几里得距离。

**公式解读**：
- 在物体 **外** 部 ($\mathbf{x} \notin O$): SDF = 到最近表面距离 (正)。
- 在物体 **内** 部 ($\mathbf{x} \in O$): SDF = -(到最近表面距离) (负)。
- 表面上: SDF = 0。

正负号自动表示 "**在外/在内**"。

### 4.2 计算

预计算 SDF 网格 (e.g., Marching Cubes 法) 或在线查询 KD-Tree。本论文对每个 YCB 物体 + 地面预生成 SDF。

### 4.3 SDF 在位姿优化中的作用

夹爪 mesh 上有 $N_v$ 个采样顶点 $\{\mathbf{x}_i\}$。对每个顶点:
- $d(\mathbf{x}_i) > 0$: 在物体外, 理想。
- $d(\mathbf{x}_i) < 0$: 穿透, 需排斥。
- $d(\mathbf{x}_i) \to 0^+$: 贴合, 理想。

最优化让所有 $d(\mathbf{x}_i)$ 接近 0 但 $\ge 0$。

---

## 5. 抓取评估指标

### 5.1 成功率
$$\text{SR} = \frac{\#\{\tau : \text{物体抓住且提起}\}}{\#\{\tau\}}$$

**公式解读**：成功试验数 / 总试验数。

### 5.2 鲁棒性 / 抗扰动

抓后施加冲量 $J = 0.1$ N·s (本论文设置), 测仍保持抓取的比例。

### 5.3 接触面积 / 夹持力

通过 FEM 算物体表面积分接触压力。

### 5.4 时间开销

位姿生成 + 评估 + 执行的总耗时。

---

## 6. 抓取位姿 vs Co-Design 的耦合

### 6.1 为何位姿和刚度耦合?

不同位姿下 **物体接触位置** 不同 → 软指变形模式不同 → 最优刚度分布不同。

例:
- 圆柱物从顶端抓 → 软指尖更好 (贴合圆顶)。
- 圆柱物从侧面抓 → 软中段更好 (贴合圆柱面)。

因此 **位姿 + 刚度** 必须联合优化。

### 6.2 联合优化的两种结构

| 外层 / 内层 | 例子 |
|---|---|
| 设计 (慢) / 控制 (快) | MORPH ([[../03-参考文献/Ref-07-He-2024-MORPH]]) |
| 控制 (sampling) / 设计 (gradient) | **本论文 (Yi'25)** |

本论文反转了通常的双层顺序: 因为位姿天然离散 (来自 AnyGrasp), 设计天然连续 (刚度 ∈ ℝ^22), 连续部分用梯度更高效。

---

## 7. 关联概念

- [[代理模型 Surrogate Model]]
- [[神经物理模拟器 Neural Physics]]
- [[Co-Design 协同设计]]
- [[Sim-to-Real]]

## 8. 关联文献

- [[../03-参考文献/Ref-46-Fang-2023-AnyGrasp]] (位姿生成)
- [[../03-参考文献/Ref-47-Qi-2017-PointNet]] (点云特征)
- [[../03-参考文献/Ref-44-Calli-2015-YCB]] (基准物体集)

---

*← 返回 [[../00-总览/主索引 MOC]]*
