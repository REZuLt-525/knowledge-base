# Sim-to-Real 仿真到现实迁移

> **一句话理解**：在仿真器学到的策略 / 设计 $\theta_\text{sim}^*$ 部署到真机后，由于 **Reality Gap (sim-real 差距)** $\Delta = f^\text{real} - f^\text{sim}$，性能下降。Sim-to-Real 是一组 **缩小 $\Delta$** 或 **使策略对 $\Delta$ 鲁棒** 的技术总称。

`#核心概念`

---

## 1. 形式化设定

### 1.1 仿真 vs 真实动力学

仿真:
$$s^\text{sim}_{t+1} = f^\text{sim}(s_t, u_t; \xi)$$

真实:
$$s^\text{real}_{t+1} = f^\text{real}(s_t, u_t)$$

**变量含义**：
- $s_t$ — 状态向量。
- $u_t$ — 控制输入。
- $\xi$ — 仿真参数 (材料、摩擦、延迟)。
- $f^\text{sim}, f^\text{real}$ — 仿真器 / 真实物理的转移函数。

误差 (Reality Gap):
$$\Delta(s, u, \xi) = f^\text{real}(s, u) - f^\text{sim}(s, u; \xi)$$

**公式解读**：相同的状态 + 动作下, 真实演化与仿真预测的差异。$\Delta$ 越大, sim2real 越难。

### 1.2 迁移目标

策略 $\pi^*$ 在仿真上学得, 部署后满足:
$$\mathbb{E}_{\tau \sim p^\text{real}}\left[\sum_t \ell(s_t, u_t)\right] - \mathbb{E}_{\tau \sim p^\text{sim}}\left[\sum_t \ell\right] \le \epsilon$$

**变量含义**：
- $\tau \sim p^\text{real}$ — 真机轨迹分布。
- $\tau \sim p^\text{sim}$ — 仿真轨迹分布。
- $\epsilon$ — 允许性能差距。

**公式解读**：策略在真机上的表现 (左侧第一项) 应当接近仿真表现 (左侧第二项)。

---

## 2. Sim-Real Gap 的来源

| 类别 | 例子 | 影响 |
|---|---|---|
| 动力学误差 | 摩擦系数、电机响应、空气阻力 | 中-大 |
| 几何误差 | 制造容差、磨损 | 小-中 |
| 材料属性 | 杨氏模量批次差、温度漂移 | 大 (软体) |
| 传感噪声 | 相机标定、深度噪声 | 中 |
| 延迟 | 网络 / 控制周期 | 中 |
| 接触模型 | 库伦摩擦 vs 真实粘滑 | 大 |

软体机器人 sim-real gap **明显大于** 刚体——本论文专门讨论此问题。

---

## 3. 五大主流 Sim-to-Real 技术

### 3.1 域随机化 (Domain Randomization, DR)

仿真时随机化参数分布, 让策略对参数变化鲁棒:

$$\xi \sim p(\xi), \quad \pi^* = \arg\max_\pi \mathbb{E}_{\xi \sim p(\xi)}\left[\sum_t r_t \mid \xi\right]$$

**变量含义**：
- $p(\xi)$ — **预设的参数分布** (通常宽于真实可能值)。
- $r_t$ — 每步奖励。
- $\mathbb{E}_{\xi \sim p(\xi)}$ — 对参数随机性的期望 (蒙特卡洛估计)。

**公式解读**：让策略不只在 **特定 $\xi$** 下好, 而是在 **大范围 $\xi$** 平均下好 → 真实参数落在范围内, 策略仍有效。

```
for episode in 训练:
    ξ ← 采样 (μ, σ_friction, m_object, ...)  # 从宽分布
    用 ξ 配置仿真
    用当前策略 π 跑一集
    update π
```

OpenAI Cube (2019)、MORPH ([[../03-参考文献/Ref-07-He-2024-MORPH]]) 都用 DR。

### 3.2 系统辨识 (System Identification, SI)

少量真机数据 + 优化拟合仿真参数:

$$\xi^* = \arg\min_\xi \;\; \sum_{(s, u, s') \in \mathcal{D}_\text{real}} \big\| f^\text{sim}(s, u; \xi) - s' \big\|^2$$

**变量含义**：
- $\mathcal{D}_\text{real}$ — 真机采集的少量数据集 $(s, u, s')$ 三元组。
- $\xi^*$ — 最佳拟合仿真参数。

**公式解读**：调 $\xi$ 让仿真器对真机数据预测最准, 然后用 $\xi^*$ 配置仿真训策略。

### 3.3 残差物理 (Residual Physics)

仿真基线 + NN 学差距:

$$\hat{f}(s, u) = f^\text{sim}(s, u; \xi) + g_\theta(s, u)$$

**变量含义**：
- $g_\theta$ — NN 残差网络。
- $\hat{f}$ — 修正后的预测。

**公式解读**：粗仿真给主干, NN 学差距。详见 [[../03-参考文献/Ref-11-Gao-2024-ResidualPhysics]]。

训练目标:
$$\theta^* = \arg\min_\theta \sum_{(s,u,s') \in \mathcal{D}_\text{real}} \|f^\text{sim}(s, u; \xi) + g_\theta(s, u) - s'\|^2$$

### 3.4 域适应 (Domain Adaptation)

把仿真观测 → 真机风格:
$$o^\text{real-style}_t = G_\phi(o^\text{sim}_t)$$

**变量含义**：
- $G_\phi$ — 风格转换网络 (CycleGAN 等)。
- $o$ — 观测 (RGB 图等)。

**公式解读**：让仿真"长得像"真机, 训出来策略在真机上认得环境。

### 3.5 真机微调 (Real-world Fine-tuning)

$$\pi_\text{deploy} = \pi_\text{sim} - \eta \sum_{\tau \in \mathcal{D}_\text{real}} \nabla L(\tau, \pi)$$

**变量含义**：
- $\pi_\text{sim}$ — 仿真训出的初始策略。
- $\nabla L(\tau, \pi)$ — 真机数据上的梯度。
- $\eta$ — 微调学习率 (通常很小)。

**公式解读**：仿真初训后, 用少量真机数据继续 SGD。
**风险**: 真机数据贵 + RL 探索可能损坏机器人。

---

## 4. 本论文 (Yi'25) 的 Sim-to-Real 流程 ⭐

本论文走 **"Sim-to-Real 简化版"** — 不学动力学差距, 而是 **静态标定材料-结构对应**。

### 步骤 1: 仿真侧的刚度参数化

仿真用 **log-modulus**:
$$\log E \in [13.5, 17.0]$$

**变量含义**：$E$ 是杨氏模量 (Pa)。$\log$ 域范围 $[13.5, 17.0]$ 对应 $E \approx [0.7\text{ MPa}, 24\text{ MPa}]$。

**为何用 log**？$E$ 跨多个数量级, log 域更适合优化器 (避免大尺度变量主导梯度)。

22 维向量 $\mathbf{k} \in \mathbb{R}^{22}$ 的每个分量都是 log $E$。

### 步骤 2: 3D 打印参数到模量映射

通过结构调刚度 (单材料 NinjaFlex Cheetah TPU):

| 块 | 可调参数 | 测量方法 |
|---|---|---|
| Flexure 块 (薄, 2mm) | wall loops, bottom shell | 三点弯曲 |
| Segment 块 (厚) | infill type/density/direction | 压痕 |

#### 三点弯曲测 $E$
$$E = \frac{P L^3}{48 I \delta}$$

**变量含义**：
- $P$ (N) — 中点加载力。
- $L$ (m) — 跨距。
- $I = bh^3/12$ (m⁴) — 截面惯性矩。
- $\delta$ (m) — 中点挠度。

#### 压痕测 $E$
$$E = \frac{(1-\nu^2) P}{2 a \delta}$$

**变量含义**：
- $\nu \approx 0.49$ — 泊松比 (近不可压软材料)。
- $a$ (m) — 压头半径。
- $\delta$ (m) — 下压深度。

每种打印参数组合做 **3 试样 × 3 次测试** → 取均值。

### 步骤 3: 建立映射表

得到 (打印参数 → 真实 $E$) 的离散查找表 / 拟合曲线:
$$E_\text{real} = h(\text{wall\_loops}, \text{infill\_density}, \text{infill\_direction})$$

### 步骤 4: 线性映射仿真 → 真机

$$\log E_\text{print} = \alpha \cdot \log E_\text{sim} + \beta$$

**变量含义**：
- $\alpha, \beta$ — 线性系数, 通过最佳拟合让 $E_\text{print}$ 范围 = $E_\text{real}$ 范围。

**公式解读**：把仿真 log E 范围 $[13.5, 17.0]$ 拉伸/平移到真实 $E$ 范围。

**为何线性映射成立**? 论文 §6.3 引用 [[../03-参考文献/Ref-50-Craig-2020-MaterialsMechanics]] 解释: 几何相同的情况下变形对 **相对刚度** (而非绝对值) 敏感, 故 22 维相对值的合适比例即可。

### 步骤 5: 部署

针对优化得到的 $\mathbf{k}^*$ (Table 4 给出), 每个 block 选对应打印参数, 3D 打印软指。

---

## 5. 真机部署的位姿管线

```
两台 Intel RealSense D435 → RGB-D → 合并点云
        │
        ▼
AnyGrasp ([[../03-参考文献/Ref-46-Fang-2023-AnyGrasp]])  → 候选 6D 位姿集合 P
        │
        ▼
SDF 增广 (避免穿透 / 碰地)
        │
        ▼
神经代理 NN_θ(p, k*, obj) → 排序  → 选 best pose p*
        │
        ▼
Franka 7-DoF 臂执行 (Dynamixel XL330 拉腱)
        │
        ▼
垂直提起 10 cm → 测试是否抓住
```

---

## 6. 评估指标

- **抓取成功率**: 抓住并提起。
- **抗扰动率**: 抓住后受 $\sim 0.1$ N·s 冲量后仍保持。
- **多物体泛化**: in-domain (YCB) vs out-of-domain (KIT/EGAD)。

本论文 Table 2 真机结果: 10 种物体 (15g - 180g), 优化夹爪平均 70-90% 成功率, 显著优于 rigid 和 soft 基线。

---

## 7. Sim-to-Real 设计 trade-off

| 决策 | 选项 | trade-off |
|---|---|---|
| 标定 vs 学习残差 | 静态测量 vs NN | 测量准但贵, NN 快但需数据 |
| 单材料 vs 多材料 | TPU only (本论文) | 简单 vs 灵活 |
| DR 强度 | 弱 vs 强 | 训练成本 vs 鲁棒 |
| 真机微调 | 是 vs 否 | 性能 vs 风险 |

本论文选 **单材料 + 静态标定 + 无 DR** — 工程上最简洁的 sim2real 方案, 尤其适合软体场景的精度需求。

---

## 8. 关联概念

- [[Co-Design 协同设计]]
- [[代理模型 Surrogate Model]]
- [[刚度分布 Stiffness Distribution]]

## 9. 关联文献

- [[../03-参考文献/Ref-06-Schaff-2023-CrawlerSimToReal]] (DR + 双层)
- [[../03-参考文献/Ref-11-Gao-2024-ResidualPhysics]] (残差)
- [[../03-参考文献/Ref-07-He-2024-MORPH]] (硬件代理 + DR)
- [[../03-参考文献/Ref-50-Craig-2020-MaterialsMechanics]] (静态标定理论)

---

*← 返回 [[../00-总览/主索引 MOC]]*
