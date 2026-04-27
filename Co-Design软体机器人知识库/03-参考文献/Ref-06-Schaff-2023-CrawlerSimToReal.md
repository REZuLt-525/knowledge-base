# [6] Schaff et al. 2023 · Sim-to-Real Co-Optimized Soft Crawler

> **原文**: C. Schaff, A. Sedal, S. Ni, M. R. Walter. *Sim-to-real transfer of co-optimized soft robot crawlers.* **Autonomous Robots**, 47(8):1195-1211, 2023.
> **译名**: 《协同优化软体爬行机器人的仿真到现实迁移》 — *自主机器人*, 2023.

`#文献-方法` `#软体机器人` `#Sim-to-Real` `#Co-Design`

---

## 1 · 研究问题

软体机器人 co-design 在仿真里玩得很 fancy，但 **真机迁移成功率长期低下**。Schaff (TTIC) 等的目标:

> 设计一个 **完整的 Sim-to-Real co-design 管线**: 让仿真共优化的软体爬行机器人能在 **真实地毯上行走**, 而不只在 PyBullet 里跑得欢快。

具体场景: 自制软体管状爬行虫, 多个气动腔体串联, 目标平面前进。

---

## 2 · 系统形式化

### 2.1 设计变量 $\boldsymbol{\theta}_h$

软爬行机器人结构参数:
$$\boldsymbol{\theta}_h = (L_1, ..., L_K, r_1, ..., r_K, t_w, E_\text{sim}) \in \mathbb{R}^{2K+2}$$
- $L_i$: 第 $i$ 段长度。
- $r_i$: 截面半径。
- $t_w$: 壁厚。
- $E_\text{sim}$: 仿真材料模量。

### 2.2 控制策略 $\pi_\phi$

气压时序 + 反馈策略:
$$P_i(t) = \pi_\phi(t \,|\, \boldsymbol{\theta}_h)$$
- 输入: 时间步、当前位置、设计参数。
- 输出: 每个气腔气压。
- $\pi_\phi$ 用 PPO 训练。

### 2.3 任务目标

$$J(\boldsymbol{\theta}_h, \boldsymbol{\phi}) = \mathbb{E}_{\xi \sim p(\xi)}\;\;\mathbb{E}_{\tau \sim \pi_\phi(\boldsymbol{\theta}_h)}\left[d_\text{forward}(\tau)\right]$$

其中 $\xi$ 是 DR 随机化参数 (摩擦, 气压响应延迟, 材料模量等)。

### 2.4 双层 + DR 优化

外层: CMA-ES / 进化采样 $\boldsymbol{\theta}_h$
内层: PPO 学 $\pi_\phi$ 给定 $\boldsymbol{\theta}_h$

---

## 3 · 算法

```
─── Schaff Co-Optimization Pipeline ─────────
─ 阶段 1: 仿真 Co-Optimization ─────────────
初始化: CMA-ES 设计种群 {θ_h^{(i)}}
重复 generation = 1..G_max:
    for each θ_h^{(i)}:
        # DR 化训练
        for episode:
            ξ ~ p(ξ)  # 随机化摩擦/延迟/模量
            用仿真 + DR 跑 PPO 训 π_φ^{(i)}
        评估 fitness(θ_h^{(i)}) = mean forward distance
    用 CMA-ES 更新设计分布
得 θ_h*, π_φ*

─ 阶段 2: 真机制造与测试 ──────────────────
- 按 θ_h* 制造软爬虫 (SDM 工艺 + 浇注硅胶)
- 接气源 + Arduino 控制器
- 部署 π_φ*, 在地毯上测前进距离
```

---

## 4 · 关键技术细节

### 4.1 Domain Randomization 范围

| 参数 | 范围 |
|---|---|
| 地面摩擦 $\mu$ | $[0.2, 0.8]$ |
| 气压响应延迟 | $[0, 50]$ ms |
| 软体模量 ±20% | (制造容差) |
| 重力扰动 | ±5% |

DR 训练让策略对真实参数变化鲁棒。

### 4.2 仿真器选择

基于 SOFA ([[Ref-41-Faure-2012-SOFA]]) 的软体 FEM + 气压腔 + 摩擦地面。

### 4.3 真机硬件

- 模塑铸造硅胶 (Dragon Skin 30)。
- Arduino + 比例阀 + 压力传感。
- IMU 测前进距离。

---

## 5 · 实验结果

### 5.1 仿真 vs 真机性能

| 设置 | 前进速度 |
|---|---|
| 仿真 (无 DR) | 12 cm/s |
| 仿真 (含 DR) | 9 cm/s (DR 内噪声) |
| 真机 (无 DR 训) | 1 cm/s 或失败 |
| **真机 (含 DR 训)** | **6 cm/s** |

DR 让 sim2real 迁移成功率从 20% → 80%。

### 5.2 Co-Design vs 固定设计对照

- 固定 (人工设计) 软爬虫 + RL 训控制: 真机 3 cm/s。
- **Co-Design + DR**: 真机 6 cm/s, 提升 **2 倍**。

---

## 6 · 关键贡献

1. **首篇软体 co-design 真机 Sim-to-Real 论文**: 把仿真共优化结果搬到真世界。
2. **DR 的关键作用**: 没有 DR 真机失败率极高。
3. **完整 SOFA-based 管线**: 软体爬虫 SOFA 仿真 + DR + PPO + CMA-ES。

---

## 7 · 局限

1. **设计自由度小** (~10 维), 不能改拓扑。
2. **真机性能仍不及仿真** (6 vs 12 cm/s), DR 没完全 close the gap。
3. **气源外接** — 不全软自主。
4. **PPO + CMA-ES 训练昂贵** — 单次实验数日。

---

## 8 · 历史影响

- 推动 "**软体 co-design 真机化**" 子方向。
- 启发后续 [[Ref-07-He-2024-MORPH|MORPH]] 用硬件代理加速。
- 与 [[Ref-11-Gao-2024-ResidualPhysics|Gao'24]] 残差物理路线并存为两种 sim2real 思路。

---

## 9 · 与本论文 (Yi'25) 的关系

两者都做软体 co-design 的真机迁移, 但路径不同:
- Schaff: locomotion + DR 强训。
- 本论文: manipulation + 静态打印参数标定。

本论文的 sim2real 简化是因为 **仅优化静态结构而非动态策略**, 减少 sim-real gap 来源。

---

*← [[_参考文献索引]]*
