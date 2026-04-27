# [11] Gao et al. 2024 · Sim-to-Real of Soft Robots with Learned Residual Physics

> **原文**: J. Gao, M. Y. Michelis, A. Spielberg, R. K. Katzschmann. *Sim-to-real of soft robots with learned residual physics.* **IEEE RAL**, 2024.
> **译名**: 《基于学习残差物理的软体机器人仿真到现实迁移》 — *IEEE 机器人与自动化快报*, 2024.

`#文献-方法` `#Sim-to-Real` `#软体机器人`

---

## 1 · 研究问题

软体机器人仿真到现实 (sim2real) 迁移长期低效, 主要因为材料属性、阻尼、空气泄漏、温度漂移等大量小误差累积形成 reality gap $\Delta$。ETH Zürich Robert Katzschmann 团队提出:

> **不修复仿真器**, 而是用 **神经网络学一个"残差函数"** $g_\theta$ 补偿仿真与真机的差距, 用极少量真机数据 ($O(\text{分钟})$) 即可显著缩小 sim-real gap。

---

## 2 · 形式化

### 2.1 仿真基线 (粗模型)

设有低保真仿真:
$$s_{t+1}^\text{sim} = f^\text{sim}(s_t, a_t; \xi)$$

例: PCC 软臂模型, 或低分辨率 FEM。

### 2.2 残差网络

学习残差:
$$g_\theta(s_t, a_t) \approx s_{t+1}^\text{real} - s_{t+1}^\text{sim}$$

修正后的预测:
$$\hat{s}_{t+1} = f^\text{sim}(s_t, a_t; \xi) + g_\theta(s_t, a_t)$$

### 2.3 训练目标

收集少量真机轨迹 $\mathcal{D}_\text{real} = \{(s_t, a_t, s_{t+1}^\text{real})\}$:

$$\theta^* = \arg\min_\theta \sum_{(s, a, s') \in \mathcal{D}_\text{real}} \big\| f^\text{sim}(s, a; \xi) + g_\theta(s, a) - s'\big\|^2$$

### 2.4 部署

用混合模型训控制策略:
$$\pi^* = \arg\max_\pi \mathbb{E}\left[\sum_t r_t \mid \hat{f} = f^\text{sim} + g_\theta\right]$$

或直接用混合模型做轨迹优化 / MPC。

---

## 3 · 算法

```
─── Residual Physics Pipeline ──────────────
─ 阶段 1: 仿真基线 ─────────────────────────
建立粗仿真 f_sim (例: PCC 软臂, 简单 FEM)
确定参数 ξ (材料模量、阻尼)

─ 阶段 2: 真机数据采集 (少量!) ──────────────
让真机执行多种动作, 记录:
    D_real = {(s_t, a_t, s_{t+1}^real)}_{i=1}^{N}
N ~ 几千点, 几分钟操作即可

─ 阶段 3: 残差 NN 训练 ──────────────────────
for epoch = 1..E:
    L = Σ ‖f_sim(s, a) + g_θ(s, a) - s'‖²
    θ ← θ - η ∇θ L
得 g_θ*

─ 阶段 4: 控制策略 ─────────────────────────
基于混合模型 ŝ = f_sim + g_θ* 训 RL/MPC

─ 阶段 5: 真机部署 ─────────────────────────
策略迁移到真机, 闭环控制
```

---

## 4 · 实验

### 4.1 硬件

ETH 自制软臂 (PneuNet 风格):
- 6 段串联气动腔。
- 每段独立气压控制。
- 末端 IMU + 视觉跟踪。

### 4.2 实验设置

- 基线仿真: 简化 PCC 模型, 假设每段常曲率。
- 真机采集: 20 分钟随机动作 → 数千数据点。
- 残差 NN: 3 层 MLP, 输入 (s, a) 共 12 维, 输出 6 维状态修正。

### 4.3 量化结果 (位姿误差)

| 模型 | 单步预测误差 | 长 rollout 误差 (10 步) |
|---|---|---|
| 仿真 (PCC) | 5-8 mm | 30 mm |
| 仿真 + 残差 | **0.5-1 mm** | **3 mm** |
| 完整 FEM | 0.3 mm | 1.5 mm (但 100× 慢) |

**残差物理 ≈ 完整 FEM 精度**, 但快 50 倍。

### 4.4 控制任务
- 末端位置跟踪。
- 轨迹跟随。
- 残差物理控制器 vs 仅仿真控制器: 真机误差从 ~30% → ~5%。

---

## 5 · 关键贡献

1. **残差物理范式**: 把"修复仿真器" 转为"学差距", 工程上简单。
2. **数据效率**: 少量真机数据即可。
3. **保留物理一致性**: 仿真基线提供主干, NN 只补细节 → OOD 鲁棒。
4. **快速部署**: 混合模型推理仍然快。

---

## 6 · 局限

1. **OOD 仍受限**: 残差 NN 在训练分布外失效。
2. **动作空间限制**: 必须近似训练时的动作类型。
3. **仿真基线必须够好**: 若 $f^\text{sim}$ 完全错误, 残差也无能为力。
4. **不能处理拓扑变化**: 比如刺破软体后场景。

---

## 7 · 历史影响

- 与 [[Ref-06-Schaff-2023-CrawlerSimToReal|Schaff'23]] 的 DR 路线对比: DR 是 "**让策略对所有可能 ξ 鲁棒**", 残差是"**精准学一个 ξ 的差距**"——两者各有优势。
- 推动 "**物理 + 学习混合**" 思想在 robotics 中传播 (类似 [[Ref-29-deAvilaBelbute-2020-GraphPDE|de Avila Belbute-Peres'20]] 在 CFD)。

---

## 8 · 与本论文 (Yi'25) 的关系

两者都做软体 sim2real, 但路径不同:
- Gao'24: 学 **动态残差** (运行时弥合)。
- 本论文: 静态 **打印参数 ↔ 模量标定** (制造时弥合)。

两种思路可未来组合: 静态标定 + 残差物理 = 双层 sim2real。

---

*← [[_参考文献索引]]*
