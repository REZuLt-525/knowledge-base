# [7] He & Ciocarlie 2024 · MORPH

> **原文**: Z. He and M. Ciocarlie. *MORPH: Design co-optimization with reinforcement learning via a differentiable hardware model proxy.* **ICRA 2024**, pp. 7764-7771.
> **译名**: 《MORPH: 通过可微硬件模型代理实现强化学习驱动的设计协同优化》 — ICRA 2024.

`#文献-方法` `#强化学习` `#硬件代理` `#Co-Design` `#里程碑`

---

## 1 · 研究问题

[[Ref-05-Chen-2020-HardwareAsPolicy|HaaP'20]] 把硬件作为 RL 策略变量, 但用 REINFORCE 估硬件梯度 → 方差极大。Columbia Ciocarlie 组两年后续作的目标：

> **能否找到一种方法，让 RL 既保留对策略的高效优化，又能高效计算硬件梯度** —— 同时保证最终设计能 **Sim-to-Real**？

核心思路: 训一个 **可微硬件代理 (Differentiable Hardware Proxy)** $\hat{T}_\psi$, 让原本不可微的物理仿真"变可微", 然后端到端反传。

---

## 2 · 系统形式化

### 2.1 三组件

- 真实仿真 $T(s, a, \theta_h) \to s'$ — 不可微 (PyBullet/MuJoCo)。
- **硬件代理** $\hat{T}_\psi(s, a, \theta_h) \to s'$ — 神经网络, 可微。
- 策略 $\pi_\phi(a | s)$。

### 2.2 代理训练目标

$$\psi^* = \arg\min_\psi \;\; \mathbb{E}_{(s, a, \theta_h, s') \sim \mathcal{D}}\;\; \big\| \hat{T}_\psi(s, a, \theta_h) - s' \big\|^2$$

数据 $\mathcal{D}$ 通过在 $\theta_h$ 范围内随机采样 + 调用真实仿真获得。

### 2.3 RL 目标 (基于代理)

$$J(\phi, \theta_h) = \mathbb{E}_{s_0, \pi_\phi}\left[\sum_{t=0}^{T-1} \gamma^t r(s_t, a_t)\right]$$

其中状态轨迹用代理生成: $s_{t+1} = \hat{T}_\psi(s_t, a_t, \theta_h)$。

### 2.4 联合梯度

由于 $\hat{T}_\psi$ 可微, 链式反传:
$$\nabla_{\theta_h} J = \sum_t \frac{\partial r}{\partial s_t}\frac{\partial s_t}{\partial \theta_h}$$
$$\frac{\partial s_{t+1}}{\partial \theta_h} = \frac{\partial \hat{T}_\psi}{\partial \theta_h} + \frac{\partial \hat{T}_\psi}{\partial s_t} \cdot \frac{\partial s_t}{\partial \theta_h}$$

策略梯度:
$$\nabla_\phi J = \mathbb{E}\left[\sum_t \nabla_\phi \log \pi_\phi(a_t|s_t) \cdot Q^\pi(s_t, a_t)\right]$$

---

## 3 · 三阶段训练流程

```
─── 阶段 A · 硬件代理预训练 ───────────────────
1) 在 θ_h ∈ Θ_h 范围内均匀采样 N_h 个候选
2) 对每个候选, 用真仿真采集 N_traj 条轨迹
3) 训练 NN ψ:
     ψ ← argmin Σ ‖T̂_ψ(s, a, θ_h) - s'‖²
4) 验证代理在 held-out 数据上误差 < ε

─── 阶段 B · RL 联合优化 ────────────────────
初始化 (φ_0, θ_h_0)
重复:
    用代理 T̂_ψ rollout K 步
    计算 RL 损失 + 硬件正则
    ∇_φ J, ∇_θ_h J = autograd.grad(L, [φ, θ_h])
    更新 (φ, θ_h)
得到 (φ*, θ_h*)

─── 阶段 C · 真机迁移 ───────────────────────
1) 制造 θ_h*
2) 在真机收集少量数据
3) 用真机数据微调 φ
```

---

## 4 · 实验

### 4.1 任务
- 多自由度夹爪抓取 (8 类物体)。
- 机械手手内操作 (魔方旋转)。

### 4.2 量化对比

| 方法 | 抓取成功 | 训练时长 | 真机成功 |
|---|---|---|---|
| 固定硬件 + PPO | 60% | 1× | 55% |
| HaaP (REINFORCE) | 70% | 5× | 60% |
| **MORPH (代理)** | **85%** | **3×** | **75%** |

代理路线比 REINFORCE 快 ~2-3×, 真机迁移 ~75% 成功率, 显著提升。

### 4.3 代理质量分析
- 代理误差随训练数据线性下降。
- $\hat{T}_\psi$ 在多 $\theta_h$ 间共享参数 → 减少数据需求。

---

## 5 · 关键贡献

1. **可微硬件代理** 概念，解决 RL co-design 的梯度方差痛点。
2. **三阶段流程** (代理预训 → 联合 RL → 真机微调) 成为后续工作模板。
3. **真机部署成功** 验证 sim-to-real 可行。

---

## 6 · 局限

1. **代理训练成本**: 需大量真仿真数据预训。
2. **代理 OOD**: $\theta_h$ 远离训练范围时失效。
3. **rollout 漂移**: 长 rollout 误差累积。

---

## 7 · 历史影响

- 推动 "**代理 + RL**" 路线成为 co-design 主流之一。
- 启发本论文 (Yi'25) 等"代理替代物理梯度" 思路。

---

## 8 · 与本论文 (Yi'25) 的关系

最接近的精神兄弟。区别: MORPH 学 **状态转移代理** 服务 RL，本论文学 **末态代理** 服务采样 + 梯度——本论文更轻量但不能处理多步控制。

---

*← [[_参考文献索引]]*
