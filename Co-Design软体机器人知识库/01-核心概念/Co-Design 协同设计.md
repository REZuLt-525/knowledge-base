# Co-Design 协同设计

> **一句话理解**：把 **机械形态 / 物理参数 $\theta_h$** 与 **控制策略 / 行为参数 $\theta_c$** 当作 **同一个优化问题** 的耦合变量，求解 $\min_{\theta_h, \theta_c} J(\theta_h, \theta_c)$，而不是先固定 $\theta_h$ 再调 $\theta_c$。

`#核心概念`

---

## 1. 形式化定义

### 1.1 单目标 Co-Design 优化问题

$$\min_{\theta_h \in \Theta_h, \;\theta_c \in \Theta_c} \;\; J(\theta_h, \theta_c) \;=\; \mathbb{E}_{\tau \sim p(\tau|\theta_h, \theta_c)}\left[\sum_{t=0}^{T-1} \ell(s_t, u_t)\right]$$

**变量含义**：
- $\theta_h \in \Theta_h$ — **硬件设计变量**: 几何尺寸、拓扑、材料模量、电机选型、传感放置等。本论文 $\theta_h \in \mathbb{R}^{22}$ (刚度向量)。
- $\theta_c \in \Theta_c$ — **控制 / 策略变量**: PID 参数、神经网络权重、轨迹序列等。本论文 $\theta_c \in \mathbb{R}^6$ (抓取位姿)。
- $\Theta_h, \Theta_c$ — 各自的可行约束集 (例如刚度上下界、关节限位)。
- $J$ — **任务表现** (越小越好)。如抓取损失。
- $\tau$ — 一条轨迹: $(s_0, u_0, s_1, u_1, ..., s_T)$。
- $p(\tau | \theta_h, \theta_c)$ — 给定硬件和策略下轨迹分布 (含随机扰动)。
- $\ell(s_t, u_t)$ — 每步损失 (位姿误差、能耗、违规惩罚)。
- $\mathbb{E}$ — 对轨迹分布期望; 实际由蒙特卡洛或采样估计。
- $T$ — 轨迹时长。

**公式解读**：找一对最佳的"硬件 + 策略", 让在它们配合下产生的轨迹平均损失最小。**关键**: 两组变量 **同时优化**, 而不是先后。

### 1.2 多目标 / 多任务版本

$$\min_{\theta_h, \theta_c} \;\; \sum_{i=1}^{K} w_i \cdot J_i(\theta_h, \theta_c)$$

**变量含义**：
- $i$ — 任务 / 物体索引 (本论文 $K = 45$ YCB 物体)。
- $J_i$ — 第 $i$ 个任务的损失。
- $w_i$ — 任务权重 (本论文等权 $w_i = 1/K$)。

**公式解读**：希望硬件 + 策略组合在 **多种任务** 上都好, 不只优化某单一物体, 缓解过拟合。

### 1.3 与"传统两阶段"对比

```
传统:    arg min_{θ_h}  R_design(θ_h)        # 工程师手工
         ↓
         arg min_{θ_c}  J(θ_h, θ_c)          # 控制工程师调参
         ↓ (上一步若不满意)
         手工迭代 ...

Co-Design: arg min_{θ_h, θ_c}  J(θ_h, θ_c)   # 一起求解
```

---

## 2. Co-Design 的三种实现形态

### 2.1 联合梯度优化

如果 $J$ 对 $\theta_h, \theta_c$ 都可微:

$$\theta_h \leftarrow \theta_h - \eta_h \nabla_{\theta_h} J$$
$$\theta_c \leftarrow \theta_c - \eta_c \nabla_{\theta_c} J$$

**变量含义**：
- $\eta_h, \eta_c$ — 学习率 (硬件 / 策略各自一个, 因尺度不同)。
- $\nabla J$ — 损失梯度, 由 AD 给出。

**公式解读**：标准梯度下降, 同时对两组变量更新。代表: [[../03-参考文献/Ref-04-Xu-2021-可微设计框架|Xu'21 DiffHand]]。

### 2.2 双层优化

$$\min_{\theta_h} J(\theta_h, \theta_c^*(\theta_h))$$
$$\theta_c^*(\theta_h) = \arg\min_{\theta_c} f(\theta_h, \theta_c)$$

**变量含义**：
- $\theta_c^*(\theta_h)$ — 给定 $\theta_h$ 时的 **最优策略** (内层结果)。
- $f$ — 内层目标 (常等于 $J$ 或其放松版)。

**公式解读**：每次外层 $\theta_h$ 改变后, 内层重新找最佳策略。详见 [[双层优化 Bi-level Optimization]]。代表: [[../03-参考文献/Ref-07-He-2024-MORPH|MORPH]]。

### 2.3 协同进化

设计种群 $\{\theta_h^{(i)}\}$ 与策略种群 $\{\theta_c^{(j)}\}$ 各自进化, 互相作为环境:

```
G_h ← random init  (设计种群)
G_c ← random init  (策略种群)
重复:
    评估每对 (θ_h^{(i)}, θ_c^{(j)}) → score
    选 top-k 设计 + top-k 策略
    交叉变异 → 新一代
```

代表: [[../03-参考文献/Ref-03-Lipson-2000-自动设计生命形态|Lipson'00]]。

---

## 3. Co-Design 关键挑战

### 3.1 高维非凸搜索空间

$\dim(\Theta_h) + \dim(\Theta_c)$ 可达 $10^3 - 10^6$, 损失景观 **非凸 + 多局部极小**。

**对策**:
- 代理模型加速评估 ([[代理模型 Surrogate Model]])。
- 多任务联合 (本论文 45 物体)。
- 随机重启 + 大学习率退火。

### 3.2 时间尺度差

设计在 **制造时一次性** 确定; 控制 **实时变化**。
**对策**: 离线 co-design + 在线微调。

### 3.3 不可微部分

拓扑 / 离散结构 / 接触不连续。
**对策**:
- 平滑接触 (sigmoid / log)。
- 神经代理 (本论文)。
- 进化 / 规划方法 ([[../03-参考文献/Ref-08-Zhao-2020-RoboGrammar|RoboGrammar]])。

### 3.4 过度特化

co-design 容易过拟合单任务。
**对策**: 多任务联合损失 $\sum_i w_i J_i$, Domain randomization, 元学习。

---

## 4. 本论文 (Yi'25) 的 Co-Design 工作流

### 4.1 损失函数 (论文 Eq. 4)

$$L_\text{opt}(p, k) = w_1 \sum_{o \in O}\big(\|f\|_2 + \|\Delta q\|_2\big) + w_2 \sum_{o \in O}\big(|\min(f_y, 0)| + |\min(\Delta q_y, 0)| + c_g\big)$$

**变量含义**：
- $p \in \mathbb{R}^6$ — 抓取位姿 (位置 + 朝向)。
- $k \in \mathbb{R}^{22}$ — 22 维刚度向量。
- $O$ — 物体集 (45 件 YCB)。
- $f \in \mathbb{R}^6$ — 末态合力 (3 力 + 3 力矩)。
- $\Delta q \in \mathbb{R}^3$ — 末态物体位移。
- $c_g \in \{0, 1\}$ — 是否触地 (失败信号)。
- $f_y, \Delta q_y$ — y 方向 (向下) 分量。
- $\min(\cdot, 0)$ — 取负部分 (惩罚下降, 不惩罚上升)。
- $w_1, w_2$ — 权重系数。

**公式解读**：
- **第一项**: 让末态尽量稳 (合力小 + 位移小)。
- **第二项**: 重点惩罚 **物体下落 + 触地** (抓取失败的标志)。

### 4.2 联合优化算法 (论文 Algorithm 1)

```
─── 离线阶段 ───────────────────────────────────
1) 数据生成 (FEM in Warp):
   80,000 样本 = 45 物体 × 50 位姿 × 50 刚度
2) 训神经物理代理 NN_θ:
   输入: (物体点云, 质心, 密度, 22D 刚度 k, 6D 位姿 p)
   输出: (末态合力 f, 位移 Δq, 是否触地 c_g)
   损失: L1 (1 hr 训练, 1 GPU)

─── 联合优化 ───────────────────────────────────
变量: 刚度 k ∈ ℝ^22 (设计) + 位姿 P_o (控制)

repeat T_max iterations:
    for each object o ∈ O:
        ℓ[p] = NN_θ(p, k, o)  for all p ∈ P_o
        π_o = argsort(ℓ)             # 由小到大排
        ℓ_top_B = Σ_{j=1}^B ℓ[π_o(j)]   # 取 Top-B=5
    L_total = Σ_o ℓ_top_B
    k ← k - η ∇_k L_total            # autograd 通过 NN
return k*, {best pose for each object}

─── 部署阶段 ───────────────────────────────────
固定 k* → 3D 打印 (FDM, NinjaFlex, wall loops + infill)
真机抓取: AnyGrasp 候选 → NN_θ 排序 → 选最优 → Franka 执行
```

**关键**: 内外层翻转 — 大多数 co-design 把"设计"作为外层、"控制"作为内层；本论文反过来 — **位姿采样为外层** (因 AnyGrasp 离散候选), **刚度梯度下降为内层** (因 22 维连续可微)。

---

## 5. Co-Design 的发展脉络

| 年代 | 代表 | 核心思想 |
|---|---|---|
| 2000 | Lipson Nature ([[../03-参考文献/Ref-03-Lipson-2000-自动设计生命形态|3]]) | 进化形态+控制 |
| 2018-2020 | DiffTaichi ([[../03-参考文献/Ref-19-Hu-2020-DiffTaichi|19]]), HaaP ([[../03-参考文献/Ref-05-Chen-2020-HardwareAsPolicy|5]]) | 可微 / RL 联合 |
| 2021 | DiffHand ([[../03-参考文献/Ref-04-Xu-2021-可微设计框架|4]]), Fit2Form ([[../03-参考文献/Ref-26-Ha-2021-Fit2Form|26]]) | 端到端可微 / 生成式 |
| 2024 | MORPH ([[../03-参考文献/Ref-07-He-2024-MORPH|7]]) | 硬件代理 |
| 2025 | **本论文** | 神经代理 + 多物体联合 |

详见 [[../04-研究脉络/Co-Design 研究时间线]]。

---

## 6. 关联概念

- [[双层优化 Bi-level Optimization]]
- [[代理模型 Surrogate Model]]
- [[神经物理模拟器 Neural Physics]]
- [[可微分模拟 Differentiable Simulation]]
- [[../02-方法分类/梯度优化方法]]
- [[../02-方法分类/强化学习协同进化]]
- [[../02-方法分类/采样优化方法]]

## 7. 关联文献

- [[../03-参考文献/Ref-03-Lipson-2000-自动设计生命形态]] (开山)
- [[../03-参考文献/Ref-04-Xu-2021-可微设计框架]]
- [[../03-参考文献/Ref-05-Chen-2020-HardwareAsPolicy]]
- [[../03-参考文献/Ref-07-He-2024-MORPH]]
- [[../03-参考文献/Ref-08-Zhao-2020-RoboGrammar]]

---

*← 返回 [[../00-总览/主索引 MOC]]*
