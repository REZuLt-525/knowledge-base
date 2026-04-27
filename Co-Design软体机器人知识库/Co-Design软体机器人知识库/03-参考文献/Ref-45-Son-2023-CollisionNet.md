# [45] Son & Kim 2023 · 局部物体裁剪碰撞网络

> **原文**: D. Son and B. Kim. *Local object crop collision network for efficient simulation of non-convex objects in GPU-based simulators.* arXiv:2304.09439, 2023.
> **译名**: 《用于 GPU 仿真器中非凸物体高效模拟的局部物体裁剪碰撞网络》 — arXiv 预印本, 2023.

`#文献-方法` `#神经物理` `#碰撞`

---

## 1 · 研究问题

GPU 仿真器 (Isaac Gym, Brax, MuJoCo MJX) 在 **凸物体** 上碰撞计算很快, 但 **非凸物体** 处理棘手:

- 选项 A: **凸分解 (V-HACD)** — 把非凸拆成多个凸部分, 每对凸做 GJK + EPA。慢且不准。
- 选项 B: **精确算法** — 直接处理非凸 mesh (e.g., Marching Cubes + SDF + EPA)。准但贵。

KAIST Beomjoon Kim 团队的目标:

> 训一个 **小型 CNN** 直接学习"两物体局部点云 → 是否碰撞 + 接触点 + 接触法线", **用 NN 替代传统几何算法** 加速 GPU 仿真。

---

## 2 · 形式化

### 2.1 输入: 局部裁剪 (Local Crop)

对物体对 $(O_1, O_2)$ 在某相对位姿 $T \in SE(3)$ 下:
1. 找接触候选点 (物体表面距离 < $\epsilon$)。
2. 以候选为中心, 截取边长 $L_\text{crop}$ 的小立方体。
3. 提取局部内的物体表面点云:
$$P_1 = \{p \in O_1: \|p - c\|_\infty \le L_\text{crop}/2\}$$
$$P_2 = \{p \in O_2: \|p - c\|_\infty \le L_\text{crop}/2\}$$

**关键直觉**: 碰撞只发生在物体表面附近的小区域, 不需看整个物体。

### 2.2 输出

NN 预测:
- $y_\text{collide} \in \{0, 1\}$: 是否碰撞。
- $\mathbf{p}_\text{contact} \in \mathbb{R}^3$: 接触点位置。
- $\mathbf{n}_\text{contact} \in S^2$: 接触法线 (单位向量)。
- $d_\text{penetrate} \in \mathbb{R}_+$: 穿透深度。

### 2.3 真值生成

用经典精确算法 (GJK + EPA) 计算 ground truth, 训练数据:
$$\mathcal{D} = \{(P_1^{(i)}, P_2^{(i)}, y_i, \mathbf{p}_i, \mathbf{n}_i, d_i)\}_{i=1}^N$$

---

## 3 · 网络架构

### 3.1 体素化 + 3D CNN

```
两物体 局部点云
   │
   ▼ Voxelization (32 × 32 × 32 voxel grid)
两通道 voxel 图 (channel 0: O_1, channel 1: O_2)
   │
   ▼ 3D CNN encoder (4 层卷积)
   │   - Conv3d(2 → 32, kernel 3) + ReLU
   │   - Conv3d(32 → 64, kernel 3) + Max pool
   │   - Conv3d(64 → 128, kernel 3) + Max pool
   │   - Conv3d(128 → 256, kernel 3) + Global pool
全局特征 (256-dim)
   │
   ├── (二分类头) MLP(256 → 64 → 1) → sigmoid → P(collision)
   ├── (位置回归头) MLP(256 → 64 → 3) → contact point
   └── (法线回归头) MLP(256 → 64 → 3) → unit-norm contact normal
```

### 3.2 损失函数

$$L_\text{total} = \alpha \cdot L_\text{BCE}(y, \hat{y}) + \beta \cdot \|\mathbf{p} - \hat{\mathbf{p}}\|^2 + \gamma \cdot (1 - \mathbf{n}^\top \hat{\mathbf{n}})$$

(法线用 cosine loss)

---

## 4 · 训练

### 4.1 数据集
- ShapeNet 100 类物体, 每类 10 件实例。
- 每对物体 1000 个随机相对位姿。
- 总样本 $\sim 10^5$。

### 4.2 训练参数
- 1 张 RTX 3090。
- 训练几小时收敛。
- Batch 128, Adam lr=1e-3。

### 4.3 验证
- Held-out test set 99% 碰撞分类准确。
- 接触点误差 < 1 mm (相对物体尺寸)。
- 法线误差 < 5°。

---

## 5 · 部署到仿真器

```
─── 仿真器主循环 ─────────────────────────
for each 接触候选 (O_1, O_2):
    crop_voxels = extract_local_voxels(O_1, O_2)
    collision, contact_pt, normal, depth = NN_collision(crop_voxels)
    if collision:
        # 添加接触约束 / 施加反力
        apply_contact_force(O_1, O_2, contact_pt, normal, depth)
```

实测: 非凸物体碰撞计算 **5-10× 加速** (相对 V-HACD + GJK + EPA)。

---

## 6 · 关键设计哲学

### 6.1 局部不变性

碰撞是局部现象 → 用局部 crop 替代全物体 → 网络结构稳定。
$$\text{Crop size 独立于物体复杂度} \Rightarrow \text{NN 推理时间恒定}$$

### 6.2 算法 → 函数

把"明确算法步骤" 转换为"参数化函数 + 数据"。
经典算法精确但慢, NN 近似但快, 在性能瓶颈处用 NN, 精度瓶颈处用算法 → 系统总体最优。

---

## 7 · 实验

### 7.1 场景: 1000 个非凸物体堆叠

仿真物体落入容器:
- 经典算法 (V-HACD + GJK + EPA): 5 fps。
- **CollisionNet**: **40-50 fps**。

8-10× 加速。

### 7.2 精度对比

| 方法 | 碰撞召回 | 接触点精度 |
|---|---|---|
| V-HACD (凸分解) | 95% | 2-5 mm |
| EPA (精确) | 100% | 0.1 mm |
| **CollisionNet** | **99%** | **<1 mm** |

NN 接近精确, 远超凸分解。

### 7.3 OOD 鲁棒
- 训练 100 类, 测试 50 新类: 性能下降 5-10%。
- 极端凹腔 (拓扑非平凡): 下降明显, 仍可作 baseline。

---

## 8 · 关键贡献

1. **碰撞检测的 NN 替代**: 首次系统化把碰撞算法变成 NN。
2. **局部 crop 设计**: 让网络对物体复杂度不敏感。
3. **GPU 友好**: 适合 RL 训练 / 大规模仿真。
4. **多输出统一**: 碰撞 + 接触点 + 法线 一次推理。

---

## 9 · 局限

1. **数据预生成**: 需大量真值训练。
2. **极端形状**: 拓扑非平凡场景仍弱于精确。
3. **网格分辨率固定**: 32³ voxel 限制细节。
4. **不学动力学**: 只判断 + 接触点, 后续力学需另算。

---

## 10 · 历史影响

- 开启 "**神经几何算法**" 子方向。
- 与 PINN (Physics-Informed NN) 不同 — PINN 强约束, CollisionNet 直接函数。
- 启发本论文 (Yi'25) 等 NN 替代物理思想。

---

## 11 · 与本论文 (Yi'25) 的关系

本论文 §3.3 引用此文作为 "**NN 可以高效编码物理**" 的论据。两者哲学相通:
- CollisionNet: NN 替代 **碰撞检测** (单一物理子任务)。
- 本论文: NN 替代 **整个 FEM 软体仿真** (更激进)。

代表 "NN 替代物理" 路线的两个粒度。

---

*← [[_参考文献索引]]*
