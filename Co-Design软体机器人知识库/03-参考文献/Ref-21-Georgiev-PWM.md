# [21] Georgiev et al. · PWM: Policy Learning with Multi-Task World Models

> **引用**: I. Georgiev, V. Giridhar, N. Hansen, A. Garg. *PWM: Policy learning with multi-task world models.* **ICLR** (Thirteenth International Conference on Learning Representations).

`#文献-方法` `#世界模型` `#多任务`

---

## 🎯 这篇论文想做什么

可微分 RL 通过对世界模型 (world model) 反向传播来更新策略，理论上比 model-free 高效。但实际中：
- 单任务训练，世界模型在长 rollout 上漂移；
- 梯度噪声大，对初值敏感。

Georgiev 等的目标：

> 通过 **多任务联合训练** 一个世界模型，让其更稳健、更通用，从而稳定地为策略提供解析梯度。

---

## 🛠️ 实现路径

### 1. 多任务世界模型

定义共享世界模型 $\hat T_\theta(s, a) = s'$，对一组任务 $\{T_1, \ldots, T_K\}$ 联合训练：
$$L_\text{wm} = \sum_{k=1}^{K} \mathbb{E}_{(s,a,s') \sim D_k}\|\hat T(s,a) - s'\|^2$$

### 2. 任务相关策略

每个任务独立的策略 $\pi_{\phi_k}(a|s)$。

### 3. 通过世界模型对策略反传

$$L_k = \sum_t \gamma^t r_k(s_t, a_t)$$
$$\nabla_{\phi_k} L_k = \sum_t \gamma^t \nabla_{\phi_k} r_k(\hat T(\hat T(...)))$$

multi-task 共享让 $\hat T$ 学到"通用动力学" → 在每个任务上反传梯度更稳定。

### 4. 实验

- 在 RL 基准 (DMC, MetaWorld) 上比单任务可微 RL 显著提升。
- 长 rollout 时漂移更小。

---

## 📐 关键公式与使用场景

### 公式 1：多任务联合损失
$$L_\text{total} = \sum_k L_\text{wm}^{(k)} + \sum_k L_\text{policy}^{(k)}$$

**含义**：让世界模型见过多种任务的动力学。

**使用场景**：
- **机器人操作多任务学习** (拾取、推、放、插入...)。
- **co-design 多物体联合优化** —— 类比！本论文 (Yi'25) 也是多物体联合优化代理 (45 个 YCB 物体)，思想完全一致。

### 公式 2：通过模型反传策略梯度
$$\nabla_\phi J = \nabla_\phi \mathbb{E}\big[r(\hat T_\theta(s, \pi_\phi(s)))\big]$$

**使用场景**：
- 任何 model-based 可微 RL；
- co-design 中"任务损失对硬件参数" 的反传 (本论文同思想)。

---

## 🔑 对本论文 (Yi'25) 的意义

本论文 §Related Work 引用 PWM 作为 **"多任务可微代理 + 联合优化"** 的同路人。两者并行：

| 方面 | PWM | 本论文 |
|---|---|---|
| 代理 | 多任务世界模型 (rollout) | 单步神经物理 (末态) |
| 优化对象 | 策略参数 $\phi$ | 设计参数 (刚度 $k$) |
| 共享方式 | 多任务共享世界模型 | 多物体共享代理 |
| 收益 | 稳定梯度 + 减少初值敏感 | 稳定梯度 + 缓解局部极小 |

→ 论文 Fig 4 显示 **联合优化比个体优化收敛更稳**——印证 PWM 的多任务学习思想在 co-design 同样有效。

---

## 🧭 延伸阅读

- 可微 co-design 痛点：[[Ref-04-Xu-2021-可微设计框架]]
- MORPH 硬件代理：[[Ref-07-He-2024-MORPH]]
- 概念：[[../01-核心概念/代理模型 Surrogate Model]]、[[../01-核心概念/双层优化 Bi-level Optimization]]

---

*← [[_参考文献索引]]*
