# [21] Georgiev et al. · PWM: 多任务世界模型策略学习

> **原文**: I. Georgiev, V. Giridhar, N. Hansen, A. Garg. *PWM: Policy learning with multi-task world models.* **ICLR** (Thirteenth International Conference on Learning Representations).
> **译名**: 《PWM: 基于多任务世界模型的策略学习》 — 第十三届国际表征学习大会 (ICLR).

`#文献-方法` `#世界模型` `#多任务`

---

## 1 · 研究问题

可微 RL 通过对世界模型 (world model) 反向传播来更新策略, 理论上比 model-free 方法高效。但实际上有两大问题:
- **单任务训练**: 世界模型在长 rollout 上漂移。
- **梯度噪声**: 高方差致策略训练不稳。

UCSD + Nvidia 团队提出:

> 用 **多任务联合训练** 一个 **共享世界模型**, 让它学到 **更稳健的通用动力学规律**, 进而为每个任务的策略提供 **低方差的解析梯度**。

---

## 2 · 形式化

### 2.1 共享世界模型

跨任务 $\{T_1, ..., T_K\}$ 共享:
$$\hat{T}_\theta(s, a) = s'$$

每个任务 $k$ 有自己的奖励函数 $r_k$ 和策略 $\pi_{\phi_k}$。

### 2.2 联合训练目标

世界模型损失:
$$L_\text{wm}(\theta) = \sum_{k=1}^{K} \mathbb{E}_{(s, a, s') \sim \mathcal{D}_k}\left[\big\|\hat{T}_\theta(s, a) - s'\big\|^2\right]$$

策略损失 (基于代理 rollout):
$$L_k(\phi_k, \theta) = -\mathbb{E}_{s_0, \pi_{\phi_k}}\left[\sum_{t=0}^{T-1} \gamma^t r_k(\hat{s}_t, a_t)\right]$$
其中 $\hat{s}_{t+1} = \hat{T}_\theta(\hat{s}_t, \pi_{\phi_k}(\hat{s}_t))$。

### 2.3 联合更新

```
重复:
    # 世界模型更新 (共享所有任务数据)
    L_wm = Σ_k Σ_data ‖T̂_θ(s,a) - s'‖²
    θ ← θ - η_θ ∇θ L_wm
    
    # 任务策略更新 (每任务独立)
    for k = 1..K:
        rollout 用 T̂_θ
        L_k = -Σ_t γ^t r_k(ŝ_t, a_t)
        φ_k ← φ_k - η_φ ∇{φ_k} L_k        # 通过 T̂_θ 反传
```

---

## 3 · 多任务的稳定性优势

### 3.1 单任务 vs 多任务 梯度方差

数学直觉: 共享世界模型的梯度更新方向是多任务平均:
$$\nabla_\theta L_\text{wm} = \sum_k \nabla_\theta L_\text{wm}^{(k)}$$

任务 $i$ 上的随机扰动被任务 $j$ 平均掉, 总梯度方差 $\sim 1/K$ 减小。

### 3.2 OOD 鲁棒性

跨任务训练让世界模型见过更多 $(s, a)$ 分布 → 在每个任务的 OOD 区域表现更好。

### 3.3 防 collapse

单任务可微 RL 容易让策略钻 world model 漏洞 (策略找一个让 model 预测过度乐观的动作)。多任务共享让漏洞少, 策略 robust。

---

## 4 · 算法

```
─── PWM Algorithm ────────────────────────────
初始化 θ (世界模型), φ_1, ..., φ_K (策略)
重复 epoch = 1..E:
    # 每任务采集真实数据
    for k = 1..K:
        在真任务 T_k 上跑 π_{φ_k} 收集 D_k

    # 世界模型更新 (跨任务联合)
    minibatch_wm = sample from ∪_k D_k
    θ ← θ - η_θ ∇θ Σ ‖T̂_θ - s'‖²

    # 每任务策略更新 (代理 rollout)
    for k = 1..K:
        从 D_k 取初始状态 s_0
        ŝ_0 = s_0; rollout T 步:
            a_t = π_{φ_k}(ŝ_t)
            ŝ_{t+1} = T̂_θ(ŝ_t, a_t)
        loss_k = -Σ γ^t r_k(ŝ_t, a_t)
        φ_k ← φ_k - η_φ ∇{φ_k} loss_k
```

---

## 5 · 实验

### 5.1 测试环境
- DeepMind Control Suite (DMC): 多任务版本。
- MetaWorld: 50 个机械臂任务。

### 5.2 基线
- Single-task 可微 RL (DreamerV2)。
- Model-free RL (PPO, SAC)。
- Single-task with planning (MPC)。

### 5.3 量化结果

| 方法 | 平均成功率 | 训练样本 |
|---|---|---|
| Model-free PPO | 60% | 10M |
| Single-task Dreamer | 65% | 5M |
| **PWM (multi-task)** | **78%** | **2M** |

PWM 样本效率提升 2-5×, 收敛更稳。

---

## 6 · 关键贡献

1. **多任务世界模型**: 共享代理稳定梯度。
2. **可微 RL 实用化**: 解决长期初值敏感 + 梯度噪声痛点。
3. **跨任务迁移**: 同一世界模型服务多个任务。

---

## 7 · 局限

1. **任务必须共享物理空间**: 跨域 (如机械手 vs 双足) 不能直接共享。
2. **训练数据需求大**: 仍需 $\sim 10^6$ 样本/任务。
3. **rollout 漂移仍存**: 只是减弱。

---

## 8 · 历史影响

- 推动可微 RL 走向稳定与实用。
- 与 [[Ref-04-Xu-2021-可微设计框架|Xu'21]] 形成对照: 单任务可微梯度问题 → 多任务联合改善。
- 启发 co-design 中"多对象联合训练" (本论文 45 物体联合)。

---

## 9 · 与本论文 (Yi'25) 的关系

本论文综述把 PWM 列为"**多任务可微代理**" 代表。哲学共鸣: 都用多任务 / 多对象联合提升代理稳定性。本论文 45 个 YCB 物体的联合优化正是 PWM 思想在 co-design 的体现 (论文 Fig 4 验证联合 > 单一)。

---

*← [[_参考文献索引]]*
