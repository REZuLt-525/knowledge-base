# 双层优化 Bi-level Optimization

> **一句话理解**：嵌套的优化结构 —— 外层变量 $x$ 的每次更新都依赖一个 **内层最优问题** 的解 $y^*(x)$，在协同设计中常对应 "外层硬件 $\theta_h$ → 内层控制策略 $\pi^*$"。

`#核心概念`

---

## 1. 数学定义

### 1.1 标准 Stackelberg 形式

$$\min_{x \in \mathcal{X}} \;\; F(x, y^*(x))$$
$$\text{s.t.}\;\; y^*(x) \in \arg\min_{y \in \mathcal{Y}} \;\; f(x, y)$$

**变量含义**：
- $x \in \mathcal{X}$ — **外层 (leader) 变量**, 例如硬件参数 $\theta_h$ (慢变, 制造时定)。
- $y \in \mathcal{Y}$ — **内层 (follower) 变量**, 例如控制策略 $\pi$ (快变, 实时调)。
- $\mathcal{X}, \mathcal{Y}$ — 可行约束集 (例如刚度上下界、策略参数空间)。
- $F$ — **上层目标函数**: 任务表现、鲁棒性、成本等。
- $f$ — **下层目标函数**: 策略损失、控制误差等。
- $y^*(x)$ — **隐式定义**: 每个 $x$ 都有一个对应的内层最优解 (注意是 $x$ 的函数)。

**公式解读**：
- 内层: 给定 $x$ 找最佳 $y$。
- 外层: 知道每个 $x$ 都会得到对应 $y^*(x)$ 后, 选最佳 $x$ 让总目标 $F(x, y^*(x))$ 最小。

### 1.2 Co-Design 的标准映射

$$\min_{\theta_h} \;\; J(\theta_h, \pi^*(\theta_h))$$
$$\text{s.t.}\;\; \pi^*(\theta_h) = \arg\max_\pi \;\; \mathbb{E}_{\tau \sim p(\tau|\theta_h, \pi)}\left[\sum_t r_t\right]$$

**变量含义**：
- $\theta_h$ — 硬件 (外层)。
- $\pi$ — 控制策略 (内层)。
- $\tau$ — 轨迹。
- $r_t$ — 每步奖励 (RL 框架, 越大越好)。

**公式解读**：外层选硬件 $\theta_h$, 内层在此硬件下训出最优策略 $\pi^*$ — 这是 Co-Design 中最常见的双层结构。

---

## 2. 求解方法

### 2.1 Implicit Differentiation (IFT, 隐函数定理)

如果内层最优满足 KKT 条件:
$$\nabla_y f(x, y^*(x)) = 0$$

对此式两边对 $x$ 求微分 (链式法则):
$$\nabla_{xy}^2 f + \nabla_{yy}^2 f \cdot \frac{\partial y^*}{\partial x} = 0$$

解出:
$$\boxed{\frac{\partial y^*}{\partial x} = -\big[\nabla_{yy}^2 f\big]^{-1}\nabla_{xy}^2 f}$$

**变量含义**：
- $\nabla_{xy}^2 f \in \mathbb{R}^{\dim y \times \dim x}$ — **混合二阶导**: $f$ 对 $x$ 和 $y$ 各求一次偏导。
- $\nabla_{yy}^2 f \in \mathbb{R}^{\dim y \times \dim y}$ — **关于 $y$ 的 Hessian**: 内层目标对 $y$ 的二阶导。
- 上标 $-1$ — 矩阵求逆。

**公式解读**：在最优解 $y^*$ 附近, 当 $x$ 变 $dx$ 时, $y^*$ 也跟着变 $dy^*$, 二者关系由内层 Hessian 求逆决定。**直觉**: $y^*$ 对 $x$ 的敏感度由"Hessian 越大 = $y^*$ 越钉死在该位置 → 敏感度越小"。

### 2.2 外层梯度组合

代入外层目标的链式法则:
$$\frac{dF}{dx} = \frac{\partial F}{\partial x} + \frac{\partial F}{\partial y}\frac{\partial y^*}{\partial x}$$

**变量含义**：
- $\frac{\partial F}{\partial x}$ — $F$ 对 $x$ 的 **直接** 偏导 (固定 $y$)。
- $\frac{\partial F}{\partial y}\frac{\partial y^*}{\partial x}$ — **间接** 部分: 通过 $y^*$ 受 $x$ 影响传递。

**公式解读**：外层梯度 = 直接影响 + 间接影响 (通过内层最优变化传递)。

**优点**：精确。**缺点**：需 Hessian 求逆, 数值稳定性差, 维度高时贵 ($O(\dim y^3)$)。

### 2.3 Unrolled Differentiation (展开式微分)

不要 IFT, 把内层 $K$ 步迭代视为可微计算图:

```
y_0 → y_1 → y_2 → ... → y_K ≈ y*
```

每步 $y_{k+1} = y_k - \alpha \nabla_y f(x, y_k)$ (内层一步梯度)。

外层反向传播:
```
for outer_iter:
    y = y_0(x)
    for k = 1..K:
        y = y - α ∇_y f(x, y)        # 内层梯度步, 全部记录
    L = F(x, y)
    g_x = autograd.grad(L, x)         # 通过内层链路
    x ← x - η g_x
```

MAML, meta-learning 即用此。

**优点**：实现简单。**缺点**：长 unroll 显存爆 (类似 RNN 反向传播)。

### 2.4 Hardware Proxy (硬件代理) — MORPH 思路

避开内层 RL 训练, 用神经网络代理硬件动态:
$$\hat{T}_\psi: (s, a, \theta_h) \to s'$$

**变量含义**：
- $\hat{T}_\psi$ — 学习的硬件代理 (神经网络, 参数 $\psi$)。
- $s, a, s'$ — 状态、动作、下一状态。
- $\theta_h$ — 硬件参数 (作为代理输入)。

**公式解读**：把"不同硬件下的物理动力学"全部装进一个 NN, 然后在代理上对 $(\theta_h, \pi)$ 做联合梯度更新, 不再分外/内层。详见 [[../03-参考文献/Ref-07-He-2024-MORPH]]。

---

## 3. Co-Design 中的典型算法骨架

```
─── 双层 RL Co-Design (HaaP, MORPH 系列) ──────────
初始化 θ_h, π_φ
重复 outer_iter = 1..M:
    # 内层 K 步 RL
    for inner_iter = 1..K:
        采集轨迹 τ ~ env(θ_h), 用 π_φ
        更新 π_φ via PPO / SAC
    # 估计外层梯度 ∇_{θ_h} J:
    #   - REINFORCE:  E[R · ∇ log p(τ|θ_h)]
    #   - Hardware proxy backprop:  AD through T̂_ψ
    #   - Finite difference (慢)
    θ_h ← θ_h - η_h ∇_{θ_h} J

─── 进化双层 (RoboGrammar) ─────────────────────────
重复:
    采样 K 个候选硬件 θ_h^{(1..K)}
    对每个候选并行训练内层 π
    评估表现 score_k
    用 GA / MCTS / CMA-ES 更新 θ_h 分布

─── 本论文 (Yi'25): 反转的双层 ─────────────────────
外层: 候选位姿采样 (P from AnyGrasp + SDF 增广)
内层: 对刚度 k 做梯度下降 (神经代理可微)

repeat:
    for each object o:
        ℓ_p = NN_θ(p, k, o) for p ∈ P_o
        rank: π = argsort(ℓ_p)
        ℓ_o = Σ_{j=1}^B ℓ_{π(j)}
    L_total = Σ_o ℓ_o
    k ← k - η ∇_k L_total
```

---

## 4. Co-Design 双层结构的时间尺度匹配

| 层 | 变量 | 时间尺度 | 改变方式 |
|---|---|---|---|
| 外 | 硬件 $\theta_h$ (几何 / 刚度 / 拓扑) | 慢 (制造时) | 离线优化 |
| 内 | 控制 $\pi$ / 位姿 $p$ / 动作序列 | 快 (实时) | 在线决策 |

时间尺度不对称带来:
- 外层不能"实时"改 → 必须离线找最优。
- 内层每次外层改后要 **重新适配** → 训练时间是瓶颈。
- 用代理 / proxy 大幅缩短内层成本是趋势。

---

## 5. 收敛性与陷阱

### 5.1 鞍点 / 局部极小

$F$ 关于 $\theta_h$ 通常 **非凸** + 内层 $y^*$ 不连续 → 多重局部极小。
**对策**: 随机初始化 + 重启; 多任务联合 ([[../03-参考文献/Ref-21-Georgiev-PWM]] 思路)。

### 5.2 内层非唯一

若 $\arg\min_y f(x,y)$ 有多解, $y^*(x)$ 不可微。
**对策**: 加正则 $\|y\|^2$ 强制凸; 或用 stochastic estimate。

### 5.3 梯度估计高方差

REINFORCE 估计 $\nabla_{\theta_h} J$ 方差极大。
**对策**: 硬件代理 / control variate / 多样本平均。

---

## 6. 实例对照

| 论文 | 外层 | 内层 | 求解方式 |
|---|---|---|---|
| HaaP ([[../03-参考文献/Ref-05-Chen-2020-HardwareAsPolicy]]) | 硬件参数 | RL 策略 | REINFORCE |
| MORPH ([[../03-参考文献/Ref-07-He-2024-MORPH]]) | 硬件参数 | RL 策略 | 硬件代理 + AD |
| Crawler ([[../03-参考文献/Ref-06-Schaff-2023-CrawlerSimToReal]]) | 软爬虫几何 | PPO | Unrolled + DR |
| RoboGrammar ([[../03-参考文献/Ref-08-Zhao-2020-RoboGrammar]]) | 图语法树 | MPC | MCTS |
| **本论文 (Yi'25)** | **位姿采样 (sampling)** | **刚度梯度** | **NN 代理 + AD** |

---

## 7. 关联概念

- [[Co-Design 协同设计]]
- [[代理模型 Surrogate Model]]
- [[../02-方法分类/强化学习协同进化]]

## 8. 关联文献

- [[../03-参考文献/Ref-04-Xu-2021-可微设计框架]]
- [[../03-参考文献/Ref-05-Chen-2020-HardwareAsPolicy]]
- [[../03-参考文献/Ref-06-Schaff-2023-CrawlerSimToReal]]
- [[../03-参考文献/Ref-07-He-2024-MORPH]]
- [[../03-参考文献/Ref-08-Zhao-2020-RoboGrammar]]

---

*← 返回 [[../00-总览/主索引 MOC]]*
