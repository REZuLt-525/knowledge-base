# 神经物理模拟器 Neural Physics

> **一句话理解**：用神经网络逼近物理映射 $\Phi: (\text{state}_t, \text{params}) \to \text{state}_{t+1}$ 或 $(\text{params}, \text{geom}) \to \text{terminal state}$，在数据集上端到端训练，使得后续的优化、控制、反设计可以在 NN 上做毫秒级评估并通过自动微分获取梯度。

`#核心概念`

---

## 1. 形式化定义

### 1.1 逐步预测 (Step-wise / Rollout)

$$s_{t+1} = \Phi_\theta(s_t, a_t, \xi)$$

**变量含义**：
- $s_t \in \mathbb{R}^n$ — 第 $t$ 时刻 **状态向量**：所有顶点的位置 + 速度 + 应变 (典型 $n \sim 10^4-10^6$)。
- $s_{t+1}$ — 下一时刻状态。
- $a_t$ — **动作 / 控制输入**: 例如腱拉力、关节扭矩、气压。
- $\xi$ — **物理参数**: 材料模量、阻尼、几何尺寸、密度等不随时间变化的量。
- $\Phi_\theta$ — 神经网络模拟器，$\theta$ 是其权重。

**公式解读**：把"物理演化一步" 抽象成函数调用。神经网络学这个函数, 训练后可在毫秒级跑出 $s_{t+1}$。**预测一段轨迹** = $T$ 次迭代调用：$s_T = \Phi_\theta^T(s_0, a_{0:T-1}, \xi)$。

### 1.2 末态预测 (Terminal-state) ⭐ 本论文路线

$$y = \Psi_\theta(\xi, \text{geom})$$

**变量含义**：
- $y$ — **末态量**: 最终物理量，例如 (合力 $\mathbf{f}$, 位姿偏移 $\Delta q$, 是否触地 $c_g$)。
- $\xi$ — 设计 / 物理参数 (本论文 22 维刚度向量)。
- $\text{geom}$ — 物体几何信息 (本论文是局部点云)。

**公式解读**：跳过中间动力学过程, 直接学 "**输入参数 + 几何 → 最终结果**"。一次前向即可, 没有 rollout 累积误差。代价: 失去中间状态信息。

### 1.3 训练问题

$$\theta^* = \arg\min_\theta \;\; \mathbb{E}_{(\xi, y) \sim \mathcal{D}}\; \ell(\Phi_\theta(\xi), y)$$

**变量含义**：
- $\mathcal{D}$ — 训练数据集，由高保真物理模拟器 $f^\star$ 离线生成 (本论文 80,000 样本)。
- $\ell$ — **损失函数**: L1 (绝对差), L2 (平方差), 或 Huber。
- $\theta^*$ — 最小化损失的代理参数。

**公式解读**：在数据集上找最贴近真实物理的代理参数。**期望** 表示对所有可能输入做平均, 实际由 mini-batch 估计。

---

## 2. 三种主流神经物理架构

### 2.1 黑盒 MLP / PointNet (本论文方案)

完整架构 (本论文 §3.3):

```
物体局部点云 P ∈ ℝ^{N × 3}
        │
        ▼  PointNet 3 层共享 MLP
        │   - Conv1d(3 → 64)  (逐点 3 维 → 64 维)
        │   - Conv1d(64 → 128)
        │   - Conv1d(128 → 256)
        │   - MaxPool over N points  (聚合到全局, 顺序无关)
        │
特征向量 φ_obj ∈ ℝ^{256}
        │
        │ concat
        ▼
[φ_obj; centroid (3); density (1); stiffness k (22)] ∈ ℝ^{282}
        │
        ▼  5 层 MLP (282 → 256 → 256 → 128 → 64 → 10)
        │  每层 ReLU 激活
        │
输出:
   - 末态合力 f ∈ ℝ⁶  (3 力 + 3 力矩)
   - 末态位姿偏移 Δq ∈ ℝ³
   - 是否触地 c_g ∈ {0, 1}
```

**层定义解释**：
- **Conv1d(3 → 64)**: 1 维卷积, 把每个点的 3 维坐标映射到 64 维特征 (相当于逐点 MLP)。
- **MaxPool**: 对所有 $N$ 个点取每维最大值 → 输出 256 维全局特征。这是 PointNet 的关键, 让结果与点输入顺序无关。
- **concat**: 把全局点云特征 + 物体信息 + 22 维刚度拼成 282 维向量。
- **MLP 后 5 层**: 把 282 维压缩到 10 维输出。

**损失函数**：
$$L = \lambda_f\|\hat{\mathbf{f}} - \mathbf{f}\|_1 + \lambda_q\|\widehat{\Delta q} - \Delta q\|_1 + \lambda_g \cdot \text{BCE}(\hat{c}_g, c_g)$$

**变量含义**：
- $\hat{\mathbf{f}}, \widehat{\Delta q}, \hat{c}_g$ — 网络预测值。
- $\mathbf{f}, \Delta q, c_g$ — 真实仿真输出 (FEM 给出)。
- $\lambda_f, \lambda_q, \lambda_g$ — 各项权重 (调节多任务损失重要程度)。
- $\|\cdot\|_1$ — L1 范数 (绝对值之和)。
- BCE — Binary Cross Entropy, 二分类交叉熵, $\text{BCE}(p, q) = -q\log p - (1-q)\log(1-p)$。

**公式解读**：三项分别监督 (力, 位姿, 接地)。L1 比 L2 对离群样本更鲁棒; BCE 适合 0/1 二分类。

### 2.2 图神经网络 (GNN) — MeshGraphNets 风格

把网格视为图 $G = (V, E)$。**消息传递层**：

$$\mathbf{e}_{ij}' = f_e\big(\mathbf{e}_{ij}, \mathbf{v}_i, \mathbf{v}_j\big)$$
$$\mathbf{v}_i' = f_v\Big(\mathbf{v}_i,\; \sum_{j \in \mathcal{N}(i)} \mathbf{e}_{ij}'\Big)$$

**变量含义**：
- $V$ — 节点集合 (网格顶点 / 粒子)。
- $E$ — 边集合 (网格连接 / 近邻关系)。
- $\mathbf{v}_i$ — 节点 $i$ 的特征向量 (位置 + 速度 + 类型)。
- $\mathbf{e}_{ij}$ — 边 (从 $i$ 到 $j$) 的特征 (相对位移)。
- $\mathcal{N}(i)$ — 节点 $i$ 的邻居集合。
- $f_e, f_v$ — 边和节点更新函数 (都是 MLP)。
- $\mathbf{v}_i', \mathbf{e}_{ij}'$ — 一层消息传递后的新特征。

**公式解读**：
- 第一行: 每条边的特征由其两端节点 + 自身原特征更新。
- 第二行: 每个节点用 **自身 + 所有邻居边的聚合** 更新。这是"消息传递"——节点之间通过边传递信息。
- 堆叠 $L$ 层后, 节点知道 $L$-跳邻域信息。最后预测 $\Delta \mathbf{v}_i$, 然后用半隐式 Euler 积分得下一时刻位置。

**积分公式**：
$$\mathbf{x}_i^{t+1} = \mathbf{x}_i^t + \mathbf{v}_i^t \,dt + \tfrac{1}{2}\Delta\mathbf{v}_i \,dt$$

**变量含义**：
- $\mathbf{x}_i^t$ — 节点 $i$ 在 $t$ 时刻位置。
- $\mathbf{v}_i^t$ — 速度。
- $\Delta\mathbf{v}_i$ — 网络预测的速度增量。
- $dt$ — 时间步长。

**公式解读**：标准半隐式 Euler 积分, 让位置更新平滑。

### 2.3 物理-神经混合 (Hybrid)

物理 solver 提供主干，NN 学残差：

$$s_{t+1} = f_\text{coarse}(s_t, a_t) + g_\theta(s_t, a_t)$$

**变量含义**：
- $f_\text{coarse}$ — 低分辨率物理求解器 (粗但物理一致)。
- $g_\theta$ — NN 学习"粗解 vs 真值差距"。

**公式解读**：粗物理给主干, NN 补细节。物理一致性强 + 数据效率高。

---

## 3. 数据生成工作流

```
─── 步骤 0 · 设计采样空间 ─────────────────────────
设计变量 ξ ∈ Ξ  (本论文: 22D 刚度 + 6D 位姿)
几何 / 物体 obj ∈ Obj  (本论文: 45 件 YCB)

─── 步骤 1 · 拉丁超立方采样 (LHS) ──────────────────
for i = 1..N:
    ξ_i ~ LHS(Ξ)              # 在 Ξ 内分层均匀抽样
    obj_i ~ Uniform(Obj)
    pose_i ~ AnyGrasp + SDF 增广

─── 步骤 2 · 高保真模拟 ────────────────────────────
for each (ξ_i, obj_i, pose_i):
    跑 FEM 至稳态 (本论文: Warp, 4000 fps × 850 frames)
    记录 y_i = (f, Δq, c_g)
# 本论文: 80,000 有效样本, 4 × RTX3090 GPU 一天

─── 步骤 3 · 神经代理训练 ──────────────────────────
loss = L1(NN(ξ_i, obj_i), y_i)
Adam (lr=1e-3), batch 256
收敛 ~1 hr (1 GPU)

─── 步骤 4 · 验证 ─────────────────────────────────
在 held-out test set 上测均误差
本论文: ~0.14 (L1) @ 80k 数据 vs 0.19 @ 20k 数据
```

---

## 4. 神经代理用于反向设计

```
输入: 训好的代理 NN_θ; 损失 L_opt(...); 设计变量初值 k_0
输出: 优化后的设计 k*

for object o in 物体集 O:
    候选位姿集合 P_o = AnyGrasp(o) ∪ SDF_augmented_poses(o)

# 联合优化
k ← k_0
重复 t = 1..T_max:
    L_total = 0
    for o in O:
        ℓ_p = NN_θ(p, k, o)  for p in P_o    # 全部并行, ms 级
        排序: π = argsort(ℓ_p)
        # 联合损失 = 全部物体的 Top-B 之和
        L_total += Σ_{j=1}^B ℓ_{π(j)}
    # 对设计 k 反向传播
    k ← k - η · ∇_k L_total                   # PyTorch autograd
return k*
```

**关键**：$\nabla_k$ 由 PyTorch 自动微分通过 NN 链路计算，**绕过** FEM 的不可微 / 数值不稳。

---

## 5. 神经物理代理的核心权衡

### 5.1 速度 vs 精度

| 方案 | 单次评估 | 精度 | 应用 |
|---|---|---|---|
| 高保真 FEM | 秒-分钟 | 高 | 离线验证 |
| 简化 PCC / Cosserat | ms-s | 中 | 实时控制 |
| 神经代理 (本文) | ~1 ms | 中-高 | co-design |
| GNN MGN | 10-100 ms | 高 | 长 rollout |

### 5.2 末态 vs 逐步预测

末态 (terminal-state):
- ✅ 单次前向，无累积误差。
- ✅ 训练数据少，损失明确。
- ❌ 失去中间动力学信息。

逐步 (rollout):
- ✅ 完整轨迹。
- ❌ 误差累积，需 long-horizon loss。
- ❌ 训练昂贵。

**本论文选末态**：抓取任务关心 "**最终是否抓住**"，不关心动作过程。

### 5.3 OOD 鲁棒性

代理只在训练分布附近可信。OOD 检测：
- 训练 ensemble，输出不一致时报警；
- 训练 GP 给出后验方差；
- 显式 OOD 检测器 (例如 NN 输出 + auxiliary scoring)。

本论文 §4.1 在 KIT / EGAD (训练时未见) 上测试，性能仅小幅下降 (92% → 82%)，证明代理 **学到了通用的物体-夹爪交互规律**，不是过拟合 YCB。

---

## 6. 神经物理 ≠ 物理信息神经网络 (PINN)

|  | 神经物理代理 | PINN |
|---|---|---|
| 训练数据 | 来自模拟器/实验 | 仅需边界条件 |
| 物理约束 | 隐式 (从数据学) | 显式 (PDE 残差作损失) |
| 适用场景 | 数据丰富 + 反向设计 | 数据稀缺 + 正向求解 |
| 速度 | 推理快 | 训练 = 求解 |

本论文是 **代理路线**，不是 PINN。

---

## 7. 关联概念

- [[代理模型 Surrogate Model]]：代理的一般理论。
- [[可微分模拟 Differentiable Simulation]]：原生物理可微 vs 代理可微。
- [[FEM 有限元法]]：本论文数据来源。
- [[../02-方法分类/GNN 图神经网络模拟器]]：替代 NN 路线。

## 8. 关联文献

- [[../03-参考文献/Ref-28-Pfaff-2021-MeshGraphNets]]：MGN GNN 代理。
- [[../03-参考文献/Ref-20-Allen-2022-InverseDesignGNS]]：代理 + 反设计。
- [[../03-参考文献/Ref-30-Allen-2022-GDM]]：GD-M 高维设计。
- [[../03-参考文献/Ref-45-Son-2023-CollisionNet]]：碰撞 NN 代理。
- [[../03-参考文献/Ref-29-deAvilaBelbute-2020-GraphPDE]]：物理 + GNN 混合。
- [[../03-参考文献/Ref-47-Qi-2017-PointNet]]：本论文使用的点云编码。

---

*← 返回 [[../00-总览/主索引 MOC]]*
