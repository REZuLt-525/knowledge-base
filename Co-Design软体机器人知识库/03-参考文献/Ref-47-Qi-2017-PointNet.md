# [47] Qi et al. 2017 · PointNet

> **原文**: C. R. Qi, H. Su, K. Mo, L. J. Guibas. *PointNet: Deep learning on point sets for 3D classification and segmentation.* **CVPR 2017**, pp. 652-660.
> **译名**: 《PointNet: 用于三维分类与分割的点集深度学习》 — IEEE 计算机视觉与模式识别大会 (CVPR), 2017.

`#文献-方法` `#点云网络` `#里程碑`

---

## 1 · 研究问题

3D 数据有三种主流表示:
1. **体素 (voxel)** — 规则但贵 ($O(n^3)$ 内存)。
2. **网格 (mesh)** — 拓扑复杂。
3. **点云 (point cloud)** — 激光扫描、RGB-D 原生输出, **无序 + 不规则**。

传统 CNN 假设规则网格输入 → 不能直接处理点云。Stanford Leonidas Guibas 团队的目标:

> 设计一个 **直接吃点云、对输入顺序不敏感、对刚体变换不变** 的神经网络架构, 用于 3D 分类、分割、语义理解。

---

## 2 · 形式化定义

### 2.1 输入

无序点集:
$$S = \{p_1, p_2, ..., p_N\},\quad p_i \in \mathbb{R}^d \;(d=3\text{ 或 } 6\text{ 含色彩 / 法线})$$

**变量含义**：
- $S$ — 点集 (无顺序的集合)。
- $p_i \in \mathbb{R}^d$ — 第 $i$ 个点的特征 ($d=3$: 仅 $(x,y,z)$ 坐标; $d=6$: 加 RGB 或法线)。
- $N$ — 点的数量 (典型 1024-65536)。

### 2.2 置换不变性 (核心数学要求)

对任意排列 $\pi \in \text{Perm}(N)$:

$$f(\{p_1, ..., p_N\}) = f(\{p_{\pi(1)}, ..., p_{\pi(N)}\})$$

**变量含义**：
- $\pi$ — 置换 (排列), 把索引 $\{1, ..., N\}$ 重新排列。
- $f$ — 待学的函数 (例如分类函数)。

**公式解读**：函数对输入点的顺序不敏感 — 同一物体的点云无论顺序如何, 输出应当一样。这是处理点云的 **必要条件** (因为点云本质上是 **集合** 而非 **序列**)。

### 2.3 通用置换不变近似定理

**定理 (Qi et al.)**: 任意连续置换不变函数 $f: 2^X \to \mathbb{R}$ 都可以任意逼近为以下形式:

$$f(\{p_1, ..., p_N\}) \approx \gamma\!\left(\mathop{\text{POOL}}_{i=1,...,N}\, h(p_i)\right)$$

**变量含义**：
- $h: \mathbb{R}^d \to \mathbb{R}^K$ — **逐点函数** (本论文用共享 MLP, 所有点 $p_i$ 都通过同一个 MLP 转为 $K$ 维特征)。
- POOL — **对称聚合算子**, 例如 max / sum / mean (对输入顺序不敏感)。
- $\gamma: \mathbb{R}^K \to \mathbb{R}$ — **后处理函数** (本论文用 MLP)。

**公式解读 (证明思路)**：
- $h$ 把每个点映射到 $K$ 维特征 (高维)。
- POOL 选每个特征维度的极值 (或求和), 对输入点顺序不敏感。
- 当 $K$ 足够大且 max pool 足够"细分"时, 可拟合任意对称函数。

**直觉**：把无序集合的处理分解成"**逐点变换 + 对称聚合**" 两步。

---

## 3 · PointNet 架构

```
输入 N × 3 (点云)
  │
  ▼ T-Net₃: 学 3×3 仿射矩阵, 对齐输入
  │  (with regularization ‖I - TTᵀ‖² ≤ 0.001)
对齐输入 N × 3
  │
  ▼ 共享 MLP (3 → 64 → 64) per-point
  │
N × 64 特征
  │
  ▼ T-Net₆₄: 学 64×64 矩阵, 对齐特征
  │
对齐特征 N × 64
  │
  ▼ 共享 MLP (64 → 128 → 1024) per-point
  │
N × 1024 特征
  │
  ▼ 全局 Max Pool over N points (聚合)
  │
全局向量 1024
  │
  ├─── (分类) MLP (1024 → 512 → 256 → k) → softmax
  └─── (分割) 把全局 1024 拼回 per-point 64 特征 → MLP (1088 → 512 → 256 → 128 → m)
```

### 3.1 共享 MLP (1×1 Conv)

每个点独立通过同一 MLP:

$$h(p_i) = \text{MLP}(p_i; \theta_\text{shared})$$

**变量含义**：
- $h$ — 共享 MLP 函数。
- $\theta_\text{shared}$ — 所有点共享的参数。
- 等价于 1×1 卷积 (每个 "像素" = 一个点)。

**公式解读**：每个点独立特征化, 但用同一组参数 → 学到的特征对 **任意点都通用**。

### 3.2 Max Pooling 对称函数

$$f_\text{global} = \max_{i=1,...,N} h(p_i)$$

**变量含义**：
- $f_\text{global} \in \mathbb{R}^{1024}$ — 全局特征向量。
- $\max$ — 逐元素 max (即对每个特征维度取所有点中的最大值)。

**公式解读**：max 操作 **天然置换不变** (集合中元素顺序不影响最大值), 因此 PointNet 输出与点的顺序无关。

### 3.3 T-Net (Transformer Network)

学一个仿射矩阵 $\mathbf{T} \in \mathbb{R}^{d \times d}$ 把输入对齐:

$$p_i' = \mathbf{T} \cdot p_i$$

**变量含义**：
- $\mathbf{T}$ — 学习的变换矩阵。
- $p_i'$ — 对齐后的点。

**正则项** 强制接近正交:

$$L_\text{reg} = \|\mathbf{I} - \mathbf{T}\mathbf{T}^\top\|_F^2$$

**变量含义**：
- $\mathbf{I}$ — 单位矩阵。
- $\mathbf{T}\mathbf{T}^\top$ — 矩阵乘积 (若 $\mathbf{T}$ 正交, 则 $\mathbf{T}\mathbf{T}^\top = \mathbf{I}$)。
- $\|\cdot\|_F^2$ — Frobenius 范数平方 (所有元素平方和)。

**公式解读**：约束 $\mathbf{T}$ **接近正交矩阵** (旋转), 防止训练时矩阵任意发散。

---

## 4 · 损失函数

### 4.1 分类

$$L_\text{cls} = -\sum_{i=1}^k y_i \log \hat{y}_i + 0.001 \cdot L_\text{reg}^{(64)}$$

**变量含义**：
- $k$ — 分类数。
- $y_i \in \{0, 1\}$ — 真实标签 (one-hot)。
- $\hat{y}_i$ — 预测概率。
- $-\sum y_i \log \hat{y}_i$ — 交叉熵损失。
- $L_\text{reg}^{(64)}$ — 64×64 T-Net 的正则项。
- 0.001 — 正则权重 (主损失主导)。

### 4.2 分割 (per-point)

$$L_\text{seg} = -\frac{1}{N}\sum_{j=1}^N \sum_{i=1}^m y_{ji} \log \hat{y}_{ji}$$

**变量含义**：
- $j$ — 点索引。
- $i$ — 部件类别索引。
- $m$ — 总部件数。
- $y_{ji}, \hat{y}_{ji}$ — 第 $j$ 个点是否属于部件 $i$ 的真实和预测。

**公式解读**：对每个点做交叉熵, 平均得分割损失。

---

## 5 · 实验

### 5.1 ModelNet40 分类
- 40 类 CAD 模型, 12,311 物体。
- PointNet: 89.2% 准确率。
- 当时 Voxel CNN baseline: 85%。
- 推理时间: 1024 点 ~3 ms (vs Voxel CNN ~100 ms)。

### 5.2 ShapeNet Part 分割
- 16 类, 50 部件。
- PointNet mIoU: 83.7%。

### 5.3 鲁棒性
- 缺失 50% 点: 性能下降 2%。
- 加入 25% 离群点: 性能下降 5%。
- 平移/旋转: T-Net 提供基本不变性。

---

## 6 · 关键贡献

1. **理论**: 通用置换不变函数定理为点云网络提供理论基础。
2. **架构**: 简洁的"共享 MLP + max pool" 范式。
3. **效率**: 内存 $O(N)$ vs Voxel $O(N^3)$。
4. **通用性**: 同一架构做分类 + 分割 + 语义理解。

---

## 7 · 局限

1. **不显式建模局部邻域** — 全部依赖 max pool 的全局聚合, 失去局部细节。
2. **大点云推理慢** — 分割 head 需 per-point 计算。
3. **对密度敏感** — 不同密度点云需重采样。

(这些被 PointNet++ 用 hierarchical sampling + grouping 改进。)

---

## 8 · 历史影响

- 引发 **3D 点云深度学习** 浪潮。
- 衍生 PointNet++, DGCNN, PointTransformer 等。
- 几乎所有现代抓取算法 (DexNet, GraspNet, AnyGrasp) 都基于 PointNet 系列。
- CVPR 2017 best paper finalist。

---

## 9 · 与本论文 (Yi'25) 的关系

本论文神经代理的 **前端编码器** 用 3 层 PointNet 处理物体局部点云:
```
N × 3 → 3 → 64 → 128 → 256 (max pool) → 全局 256 向量
```

选 PointNet 而非 PointNet++ 的理由:
- 末态预测任务对局部几何要求不高。
- 训练快 (~1 hr vs ~半天)。
- 推理 ~1 ms 满足实时部署。

---

*← [[_参考文献索引]]*
