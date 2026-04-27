# [12] Chen et al. 2018 · 拓扑优化软索驱动夹爪

> **原文**: F. Chen, W. Xu, H. Zhang, Y. Wang, J. Cao, M. Y. Wang, H. Ren, J. Zhu, Y. Zhang. *Topology optimized design, fabrication, and characterization of a soft cable-driven gripper.* **IEEE RAL**, 3(3):2463-2470, 2018.
> **译名**: 《拓扑优化软索驱动夹爪的设计、制造与表征》 — *IEEE 机器人与自动化快报*, 2018.

`#文献-方法` `#拓扑优化` `#软体夹爪`

---

## 1 · 研究问题

软夹爪几何 (指状空腔、片状弯折) 通常基于经验, 但 **没人能给数学最优答案**。给定材料预算和性能目标, **算法能否自动决定材料分布?**

南科大 / 港科大 Michael Y. Wang 团队首次将 **SIMP 拓扑优化** 应用于软索驱动夹爪:

> 给定 (a) 设计区域 $\Omega$, (b) 边界载荷 $\mathbf{f}$ (腱拉力), (c) 输出位移目标点 $\mathbf{x}_\text{out}$, (d) 材料体积预算 $V^*$, **求材料分布 $\rho_e \in [0,1]$ 使得指尖输出位移最大化**。

---

## 2 · 形式化

### 2.1 设计变量 — 单元密度

把设计区域 $\Omega$ 离散为 $N_e$ 个有限元 (例如平面四边形单元), 每个单元一个 **密度变量**:

$$\rho_e \in [0, 1],\quad e = 1, 2, ..., N_e$$

**变量含义**：
- $e$ — 单元编号 (1 到 $N_e$)。
- $\rho_e$ — 第 $e$ 个单元的"材料密度":
  - $\rho_e = 1$: 实材 (有材料)。
  - $\rho_e = 0$: 空 (无材料)。
  - 中间值: 中间灰度 (优化中允许, 最后阈值化)。

**公式解读**：把"哪里有材料"用每个像素 0/1 描述, 但优化时松弛为 [0,1] 连续可微。

### 2.2 SIMP 插值 (核心)

为让连续松弛收敛到 0/1, 定义单元有效模量:

$$E_e = E_\text{min} + \rho_e^p (E_0 - E_\text{min})$$

**变量含义**：
- $E_0$ (Pa) — **实材模量** (典型 ~MPa 级软材料)。
- $E_\text{min}$ (Pa) — **极小值** (~$10^{-9} E_0$, 防奇异避免空区刚度矩阵奇异)。
- $p \ge 1$ — **惩罚指数** (通常 $p = 3$)。
- $E_e$ — 该单元等效模量。

**公式解读**：
- $\rho_e = 1$: $E_e = E_0$ (实材)。
- $\rho_e = 0$: $E_e = E_\text{min}$ (近似空)。
- $\rho_e = 0.5, p=3$: $E_e \approx 0.125 E_0$ — 中间灰度的等效模量 **远低于权重比例**, 这就是惩罚效应: **不利于灰度** → 最终解收敛到 0/1。

### 2.3 全局刚度矩阵

$$\mathbf{K}(\boldsymbol{\rho}) = \sum_{e=1}^{N_e} \rho_e^p \,\mathbf{K}_e^0$$

**变量含义**：
- $\mathbf{K} \in \mathbb{R}^{n \times n}$ — 全局刚度矩阵 ($n$: 总自由度数)。
- $\mathbf{K}_e^0 \in \mathbb{R}^{n \times n}$ — 单元 $e$ 在 $E_0$ 下的刚度矩阵 (除该单元处, 其他元素为 0)。
- $\boldsymbol{\rho} = (\rho_1, ..., \rho_{N_e})^\top$ — 全部单元密度。

**公式解读**：每个单元贡献按密度幂律加权 → 所有单元贡献求和 = 全局刚度。

### 2.4 平衡方程

$$\mathbf{K}(\boldsymbol{\rho})\,\mathbf{U} = \mathbf{F}$$

**变量含义**：
- $\mathbf{U} \in \mathbb{R}^n$ — 节点位移向量 (待求未知量)。
- $\mathbf{F} \in \mathbb{R}^n$ — 等效节点力 (来自腱拉力等)。

**公式解读**：标准 FEM 线性系统, 给定密度 $\boldsymbol{\rho}$ 后能解出 $\mathbf{U}$。

### 2.5 优化问题

$$\max_{\boldsymbol{\rho}}\;\; u_\text{out} = \mathbf{e}_\text{out}^\top \mathbf{U}$$

$$\text{s.t.} \;\; \mathbf{K}(\boldsymbol{\rho})\,\mathbf{U} = \mathbf{F}$$

$$\;\;\;\; \sum_{e=1}^{N_e} v_e \,\rho_e \le V^*$$

$$\;\;\;\; 0 \le \rho_e \le 1, \;\forall e$$

**变量含义**：
- $u_\text{out}$ — 指尖输出位移 (标量目标)。
- $\mathbf{e}_\text{out} \in \mathbb{R}^n$ — **选择向量**, 在指尖位移分量处为 1, 其他为 0。
- $v_e$ — 单元 $e$ 的体积 (m³)。
- $V^*$ — 材料预算 (m³)。

**公式解读**：
- **目标**: 最大化指尖位移 (软夹爪要"位移大")。
- **约束 1**: FEM 平衡 (物理一致)。
- **约束 2**: 总材料量不超预算。
- **约束 3**: 密度 0-1 之间。

---

## 3 · 灵敏度分析 — 伴随法

### 3.1 朴素方法的代价

直接计算 $\partial u_\text{out} / \partial \rho_e$ 需对每个 $e$ 重解 PDE → $N_e$ 次。$N_e \sim 10^4$ 不可行。

### 3.2 拉格朗日

$$\mathcal{L} = u_\text{out} + \boldsymbol{\lambda}^\top (\mathbf{F} - \mathbf{K}\mathbf{U})$$

**变量含义**：
- $\boldsymbol{\lambda} \in \mathbb{R}^n$ — **拉格朗日乘子** / 伴随变量。
- 由于 $\mathbf{K}\mathbf{U} = \mathbf{F}$, $\mathcal{L} = u_\text{out}$ (与原目标一致)。

**公式解读**：用拉格朗日乘子把约束代入目标。

### 3.3 对 $\rho_e$ 求导

$$\frac{\partial \mathcal{L}}{\partial \rho_e} = \frac{\partial u_\text{out}}{\partial \rho_e} - \boldsymbol{\lambda}^\top\frac{\partial \mathbf{K}}{\partial \rho_e}\mathbf{U} - \boldsymbol{\lambda}^\top\mathbf{K}\frac{\partial \mathbf{U}}{\partial \rho_e}$$

**公式解读**：链式求导后有三项, 包含 $\frac{\partial \mathbf{U}}{\partial \rho_e}$ (要求每 $e$ 都解一次, 昂贵)。

**关键技巧**: 选 $\boldsymbol{\lambda}$ 使最后一项消失:

$$\mathbf{K}\boldsymbol{\lambda} = \mathbf{e}_\text{out}$$ ← 伴随方程

**公式解读**：解一次伴随系统 (与原方程同形式), 把昂贵的 $\frac{\partial \mathbf{U}}{\partial \rho_e}$ 消除。

### 3.4 最终灵敏度公式

代入解得:
$$\boxed{\frac{\partial u_\text{out}}{\partial \rho_e} = -p\,\rho_e^{p-1}(E_0 - E_\text{min})\;\mathbf{u}_e^\top \mathbf{k}_e^0\,\boldsymbol{\lambda}_e}$$

**变量含义**：
- $\mathbf{u}_e \in \mathbb{R}^{n_e}$ — 单元 $e$ 的节点位移子向量。
- $\boldsymbol{\lambda}_e \in \mathbb{R}^{n_e}$ — 单元 $e$ 的伴随变量子向量。
- $\mathbf{k}_e^0$ — 单元 $e$ 在 $E_0$ 下的局部刚度矩阵。
- $p \rho_e^{p-1}$ — SIMP 幂律对 $\rho_e$ 求导。

**公式解读**：每个单元的灵敏度由 (a) 该单元位移、(b) 该单元伴随变量、(c) 该单元刚度矩阵 三者决定, **完全局部** 计算。

**复杂度**: 解 $\mathbf{K}\mathbf{U} = \mathbf{F}$ + $\mathbf{K}\boldsymbol{\lambda} = \mathbf{e}_\text{out}$ = 共 **2 次 PDE 求解** (与 $N_e$ 无关)！

---

## 4 · OC 更新规则

### 4.1 更新公式

Optimality Criteria 法迭代更新:

$$\rho_e^{(k+1)} = \begin{cases}
\max(0, \rho_e^{(k)} - m), & \text{if } \rho_e^{(k)} B_e^\eta \le \max(0, \rho_e^{(k)} - m) \\
\min(1, \rho_e^{(k)} + m), & \text{if } \min(1, \rho_e^{(k)} + m) \le \rho_e^{(k)} B_e^\eta \\
\rho_e^{(k)} B_e^\eta, & \text{otherwise}
\end{cases}$$

**变量含义**：
- $k$ — 迭代步数。
- $m$ — **步长上限** (例如 0.2), 限制每步密度变化。
- $\eta$ — **阻尼系数** (例如 0.5), 控制更新平稳性。
- $B_e$ — **更新指标**, 由灵敏度决定 (见下)。

**公式解读**: 三种情况:
- 第 1 行: 下界饱和 (设计想减得太多)。
- 第 2 行: 上界饱和 (设计想加得太多)。
- 第 3 行: 正常更新, $\rho_e \cdot B_e^\eta$。

### 4.2 更新指标

$$B_e = -\dfrac{\partial u_\text{out}/\partial \rho_e}{\Lambda \,v_e}$$

**变量含义**：
- $\partial u_\text{out}/\partial \rho_e$ — 灵敏度 (正常情况下是负数, 增加密度反而减小目标)。
- $\Lambda$ — 拉格朗日乘子, 通过二分法选取以 **满足体积约束**。
- $v_e$ — 单元体积。
- $B_e > 1$: 该单元应增加密度。
- $B_e < 1$: 应减少。
- $B_e = 1$: 处于"最优" (KKT 条件)。

---

## 5 · 完整算法工作流

```
─── Topology Optimization for Soft Gripper ───
1) 离散设计区域 Ω 为 N_e 单元
2) 初始化 ρ_e = V*/(N_e · v_e), 均匀
3) 迭代 k = 1..K:
    a) 组装 K(ρ); 解 KU = F → 得 U
    b) 解 Kλ = e_out → 得 λ
    c) 计算灵敏度 ∂u_out/∂ρ_e (公式见 §3.4)
    d) 密度过滤 (Helmholtz PDE 防棋盘格):
       (-r²∇² + 1) ρ̃ = ρ
    e) OC 更新 ρ
    f) 检查收敛: max|Δρ| < tol
4) 阈值化: ρ̃_e > 0.5 → 1, else → 0
5) 平滑 + 转 STL
6) 多材料 3D 打印 (硬骨架 + 软体)
```

### 密度过滤
$$(-r^2\nabla^2 + 1) \tilde{\rho} = \rho$$

**变量含义**：
- $r$ — 过滤半径 (单元几何尺度的 1.5-3 倍)。
- $\nabla^2$ — Laplace 算子 (二阶空间偏导)。
- $\tilde{\rho}$ — 过滤后密度。
- $\rho$ — 原始密度。

**公式解读**：解 Helmholtz 方程相当于 **空间平滑**, 防止 "棋盘格" 不可制造的微结构 (相邻像素 0/1/0/1 交替)。

---

## 6 · 实验

### 6.1 设计场景
- 设计区域: 60 × 30 × 5 mm 矩形薄板。
- 输入: 5 N 腱拉力。
- 体积约束: 50% 实材。
- 输出目标: 最大化指尖侧向位移。

### 6.2 优化结果
- 算法收敛 ~150 迭代。
- 优化后位移 5.2 mm, 比基线 (均匀分布) 提升 1.8×。
- 涌现 **类似自然手指的"骨架 + 软组织"** 分层。

### 6.3 制造与测试
- 多材料 PolyJet 打印 (硬塑料 + 软橡胶)。
- 实物变形与仿真匹配 (~5% 误差)。
- 抓取 100+ 物体, 成功率 90%+。

---

## 7 · 关键贡献

1. **首次将 SIMP 用于软夹爪**: 把工程优化经典移植到软体机器人。
2. **完整工作流**: 从 SIMP → 伴随法 → OC → 制造, 端到端。
3. **量化提升**: 自动设计比经验设计性能高 2× 量级。

---

## 8 · 局限

1. **静态线性问题**: 不处理大变形、动态接触。
2. **设计变量极多**: $N_e \sim 10^4$, 求解慢。
3. **灰度后处理依赖人工**。
4. **不联合优化控制**: 假设腱拉力固定。

---

## 9 · 历史影响

- 软体机器人拓扑优化的代表作。
- 衍生 [[Ref-13-Chen-2020-软体设计综述]] 综述。
- 影响后续连续设计 (本论文) 与生成式设计 ([[Ref-26-Ha-2021-Fit2Form]])。

---

## 10 · 与本论文 (Yi'25) 的关系

本论文是 "**结构级 SIMP**": 几何固定, 仅优化 22 维 block 模量。Chen'18 是几何拓扑优化, 本论文是材料分布优化。两者哲学一致, 工具不同 (NN 代理 vs 伴随法)。

---

*← [[_参考文献索引]]*
