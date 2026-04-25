# [7] He & Ciocarlie 2024 · MORPH

> **引用**: Z. He and M. Ciocarlie. *MORPH: Design co-optimization with reinforcement learning via a differentiable hardware model proxy.* **ICRA 2024**, pp. 7764-7771.

`#文献-方法` `#强化学习` `#硬件代理` `#Co-Design` `#里程碑`

---

## 🎯 这篇论文想做什么

[[Ref-05-Chen-2020-HardwareAsPolicy|HaaP'20]] 把硬件视作策略变量，但用 REINFORCE 估硬件梯度 —— 方差大、训练慢、很难真机部署。He & Ciocarlie 的目标：

> 找一种 **既保留 RL 的策略学习能力**，**又能高效计算硬件梯度** 的 co-design 方案，**并能成功 Sim-to-Real**。

---

## 🛠️ 实现路径

### 核心创新：可微硬件代理 (Differentiable Hardware Proxy)

把"机器人硬件 → 状态-动作转移" 这一不可微过程用 **神经网络 $f_\theta$** 近似：
$$s_{t+1} \approx f_\theta(s_t, a_t, h)$$
- $s$：状态 (关节角、速度等)；
- $a$：动作；
- $h$：硬件参数 (例如手指长度、关节摩擦)。

这个 $f_\theta$ **训练一次** 之后可以替代真实仿真用于梯度反传。

### 三阶段训练流程

```
阶段 A. 硬件代理预训练
   - 在仿真器中遍历 h 范围, 采样 (s, a, h) → s'
   - 训练 f_θ 逼近真实物理转移

阶段 B. RL 联合优化
   - 用 f_θ 替代仿真做 RL rollout
   - 同步更新策略 π_φ 和硬件 h (通过反向传播)
   - 损失: 任务奖励 + 硬件正则

阶段 C. 真机微调
   - 把得到的 (h*, π_φ*) 部署到真机
   - 用少量真机数据微调策略 (DR + sim2real)
```

### 关键技术点

**(1) 硬件代理的训练损失**：
$$L_\text{proxy} = \|f_\theta(s, a, h) - s'_\text{sim}\|^2 + \lambda\|f_\theta(s, a, h) - s'_\text{real}\|^2$$
若有少量真机数据，加权混合。

**(2) 联合更新**：
$$h \leftarrow h - \alpha_h \nabla_h L_\text{RL}(\pi_\phi, h)$$
$$\phi \leftarrow \phi - \alpha_\phi \nabla_\phi L_\text{RL}(\pi_\phi, h)$$
两条链路都通过 $f_\theta$ 反向传播。

**(3) 硬件分布**：训练时 $h \sim p(h)$ 让代理对硬件变化有泛化能力。

---

## 📐 关键公式与使用场景

### 公式 1：可微代理替代仿真
$$\hat{s}_{t+1} = f_\theta(s_t, a_t, h)$$

**使用场景**：
- **代替 PyBullet/MuJoCo 做 RL 内层 rollout** —— 推理快、可微、不卡 CUDA。
- **机器人形态搜索**：连杆长度、关节摩擦等硬件参数与策略联合学习。

### 公式 2：联合梯度
$$\nabla_h J(\pi_\phi, h) = \mathbb{E}\big[\partial_h\big(R(f_\theta(s, a, h))\big)\big]$$

**使用场景**：
- 给硬件参数提供 **解析、低方差** 梯度，避免 REINFORCE 的高方差。
- 当真实环境梯度不存在时，是 co-design 的"通用桥梁"。

### 实验数据 (论文报告)

- 多关节夹爪在 8 项操作任务上比 HaaP 提速 ~3-5×。
- Sim-to-Real 成功率 70-85%。
- 学到的硬件参数与人类工程师"反直觉"——例如某些关节摩擦最优值非零。

---

## 🔑 对本论文 (Yi'25) 的关键意义

⭐ **MORPH 是和本论文最接近的精神兄弟**：

| 维度 | MORPH (2024) | 本论文 (Yi'25) |
|---|---|---|
| 代理类型 | 状态转移 $f(s,a,h)$ | 末态预测 $f(\text{obj}, k)$ |
| 控制 | RL 策略 $\pi_\phi$ | 候选位姿采样 + 排序 |
| 优化 | 联合 $\nabla_h, \nabla_\phi$ | 外层位姿 + 内层刚度梯度 |
| 任务粒度 | 多步轨迹 | 单步抓取 (末态) |
| 训练成本 | 大 (需 RL 收敛) | 小 (1 小时代理训练) |

**两者的共同信念**：原始物理梯度不可靠 → 用神经代理。
**两者的差异**：本论文不需 RL，因为抓取是单步任务，让代理走简单路径就够。

---

## 🧭 延伸阅读

- 前身：[[Ref-05-Chen-2020-HardwareAsPolicy]]
- 可微模拟对比：[[Ref-04-Xu-2021-可微设计框架]]
- 反设计 (代理 + 梯度)：[[Ref-20-Allen-2022-InverseDesignGNS]]
- 概念：[[../01-核心概念/代理模型 Surrogate Model]]、[[../01-核心概念/双层优化 Bi-level Optimization]]
- 方法分类：[[../02-方法分类/强化学习协同进化]]

---

*← [[_参考文献索引]]*
