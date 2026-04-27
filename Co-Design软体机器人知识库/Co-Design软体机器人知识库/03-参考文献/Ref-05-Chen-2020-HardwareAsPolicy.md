# [5] Chen, He, Ciocarlie 2020 · Hardware as Policy (HaaP)

> **原文**: T. Chen, Z. He, M. Ciocarlie. *Hardware as policy: Mechanical and computational co-optimization using deep reinforcement learning.* arXiv:2008.04460, 2020.
> **译名**: 《硬件即策略：基于深度强化学习的机械与计算协同优化》 — arXiv 预印本, 2020.

`#文献-方法` `#强化学习` `#Co-Design`

---

## 1 · 研究问题

强化学习 (RL) 已经能用一组参数 $\boldsymbol{\phi}$ 描述策略 $\pi_\phi(a | s)$。Columbia Matei Ciocarlie 团队的关键洞察：

> **既然策略由参数控制，硬件 (尺寸、刚度、电机布置) 也可以视作参数化的"策略一部分"。** 把硬件参数 $\boldsymbol{\theta}_h$ 与策略参数 $\boldsymbol{\phi}$ 一起放入 RL 的更新循环，**让 RL 同时优化软件和硬件**。

这是 RL 路线 co-design 的先驱工作。

---

## 2 · 系统形式化

### 2.1 拓展策略

$$\pi_\theta(a | s) \quad\Rightarrow\quad \pi_{(\phi, \theta_h)}(a | s)$$

其中 $\boldsymbol{\theta}_h \in \Theta_h$ 是硬件参数 (例如手指长度 $L_i$, 关节刚度 $k_j$, 电机最大扭矩 $\tau_\max$)。

### 2.2 RL 目标

$$J(\boldsymbol{\phi}, \boldsymbol{\theta}_h) = \mathbb{E}_{\tau \sim p(\tau | \pi_\phi, \theta_h)}\left[\sum_{t=0}^{T-1} \gamma^t r(s_t, a_t)\right]$$

每次环境重置时, 仿真器根据 $\boldsymbol{\theta}_h$ **重建** 机器人 (改变 URDF / FEM 网格)。

### 2.3 梯度估计 (REINFORCE / PPO)

由于物理仿真不可微 (PyBullet, MuJoCo)，硬件梯度需用 score function:
$$\nabla_{\theta_h} J \approx \mathbb{E}_\tau\left[ R(\tau) \cdot \nabla_{\theta_h} \log p(\tau | \pi_\phi, \theta_h) \right]$$

但 $p(\tau)$ 通常不显式依赖 $\boldsymbol{\theta}_h$ (硬件影响进入仿真黑盒)。论文做近似:
1. 把 $\boldsymbol{\theta}_h$ 当作 stochastic variable (加少量噪声 $\boldsymbol{\theta}_h \sim p(\theta_h)$)。
2. 用 REINFORCE 估梯度。

策略梯度则正常 PPO:
$$\nabla_\phi J = \mathbb{E}_\tau\left[\nabla_\phi \log \pi_\phi(a_t | s_t) \cdot \hat{A}_t\right]$$

### 2.4 顺序图结构 (Sequential Graph)

为减少高维硬件搜索难度, 提出 **粗-细 unfreezing**:

```
阶段 1: 固定细参数, 只优化粗参数 (总长度)
阶段 2: 粗参数稳定后, 解锁中等参数 (每段长度)
阶段 3: 解锁细参数 (关节安装角)
```

每阶段都跑完整 RL 训练循环。

---

## 3 · 算法

```
─── HaaP Algorithm ──────────────────────────
初始化 (φ_0, θ_h_0); 阶段索引 stage = 1
重复 outer_iter = 1..N:
    # PPO 策略更新
    采集 K 条轨迹 τ
    计算 advantage Â_t
    φ ← φ + η_φ · Σ ∇_φ log π · Â
    
    # 硬件更新 (每 K 个 outer_iter 一次)
    if outer_iter % K_h == 0:
        θ_h ← θ_h + η_h · 估计梯度 (REINFORCE)
        if 阶段收敛: stage += 1; unfreeze 细参数
        if θ_h 出范围: clip 到可行域
返回 (φ*, θ_h*)
```

---

## 4 · 实验

### 4.1 任务
1. **5-DoF 抓取** — 优化夹爪形态 + 控制策略抓取多种物体。
2. **机械臂抛接球** — 优化臂长 + 投掷时机。
3. **抓取后操作** — 优化手指刚度 + 操作动作序列。

### 4.2 对照
- **Baseline 1**: 固定硬件, RL 训策略。
- **Baseline 2**: 网格搜索硬件 + 每次 RL 训策略 (双层但低效)。
- **HaaP**: 联合 RL 优化。

### 4.3 量化结果

抓取任务:
- Baseline 1: 60% 成功率。
- Baseline 2: 70% 成功率 (但 50× 计算)。
- **HaaP: 78% 成功率**, 同样计算预算下显著提升。

---

## 5 · 关键贡献

1. **概念**: "Hardware as Policy" — 硬件参数纳入 RL 更新链。
2. **顺序图结构**: 粗-细 unfreezing 减少高维搜索难度。
3. **演示效果**: 同样计算预算下比固定硬件 + 双层都好。

---

## 6 · 局限

1. **REINFORCE 梯度方差大** — 需要大量样本，训练慢。
2. **硬件需要 stochastic 化** 才能给出梯度 — 引入额外噪声。
3. **顺序图阶段切换是手工的** — 不够自动。

(这些局限被后续 [[Ref-07-He-2024-MORPH|MORPH]] 用 **可微硬件代理** 解决。)

---

## 7 · 历史影响

- 第一次系统提出 "硬件作为 RL 策略一部分"。
- 启发 MORPH (同 Ciocarlie 组续作)。
- 推动 RL co-design 路线。

---

## 8 · 与本论文 (Yi'25) 的关系

哲学相通: 都把 "**硬件 + 决策**" 作为统一优化目标。差异:
- HaaP 用 RL + REINFORCE 估硬件梯度 (高方差)。
- 本论文用 NN 代理 + AD (低方差, 1000× 快)。

---

*← [[_参考文献索引]]*
