# [47] Qi et al. 2017 · PointNet

> **引用**: C. R. Qi, H. Su, K. Mo, L. J. Guibas. *PointNet: Deep learning on point sets for 3D classification and segmentation.* **CVPR 2017**, pp. 652-660.

`#文献-方法` `#点云网络` `#里程碑`

---

## 🎯 这篇论文想做什么

3D 数据有两种主流表示：**体素** 和 **点云**。点云是激光扫描、Kinect、双目立体的原生输出，但 **点的顺序不固定** —— 任意排列都代表同一个物体。CNN 假设 grid 顺序，无法直接处理点云。Qi 等的目标：

> 设计一个 **可直接吃点云、输出分类/分割** 的神经网络，且 **对输入顺序不敏感** (置换不变)。

应用场景：3D 物体识别、自动驾驶 (LiDAR)、机器人感知 (本论文)。

---

## 🛠️ 实现路径

### 关键洞察 1：置换不变 (permutation invariance)

数学要求：对任意置换 $\pi$，
$$f(\{\mathbf{p}_{\pi(1)}, \ldots, \mathbf{p}_{\pi(N)}\}) = f(\{\mathbf{p}_1, \ldots, \mathbf{p}_N\})$$

通用形式：
$$f(S) = \rho\left(\bigoplus_{i=1}^{N} \phi(\mathbf{p}_i)\right)$$
- $\phi$：作用在每个点上的函数 (如 MLP)。
- $\bigoplus$：对称聚合算子 (max / sum / mean)。
- $\rho$：后处理函数。

PointNet 选 **max pool** 作为聚合算子。

### 关键洞察 2：T-Net 学习对齐

为消除 **刚体变换** (旋转/平移) 的差异，论文加入 T-Net：
- 输入 T-Net：学一个 $3\times 3$ 矩阵 $\mathbf{T}_3$，把所有输入点旋转到对齐位置。
- 特征 T-Net：在中间层学 $64\times 64$ 矩阵对齐特征。

正则项：$L_\text{reg} = \|\mathbf{I} - \mathbf{T}\mathbf{T}^\top\|_F^2$ 鼓励 T 接近正交。

### 完整架构

```
输入: N × 3 (点云)
      │
      ▼  T-Net₃
对齐输入 (N × 3)
      │
      ▼  共享 MLP (3→64→64)
N × 64 特征
      │
      ▼  T-Net₆₄
对齐特征 (N × 64)
      │
      ▼  共享 MLP (64→128→1024)
N × 1024 特征
      │
      ▼  Max Pool
全局向量 1024
      │
      ├── (分类) MLP (1024→512→256→k)
      └── (分割) 拼接逐点特征 → MLP
```

---

## 📐 关键公式与使用场景

### 公式 1：通用对称函数定理
$$f(\{x_1, ..., x_n\}) \approx \gamma\left(\max_{i=1,...,n} h(x_i)\right)$$
论文证明：**任意连续置换不变函数** 都可以用这种"逐点 MLP + max pool" 形式 **任意逼近**。

**使用场景**：
- 3D 物体分类 (ModelNet40)。
- 点云语义分割。
- **机器人抓取的物体几何编码** (本论文核心用法)。

### 公式 2：T-Net 正则
$$L_\text{reg} = \|\mathbf{I} - \mathbf{T}\mathbf{T}^\top\|_F^2$$

**作用**：约束变换矩阵接近正交，避免训练发散。

**使用场景**：处理同一物体不同朝向时的鲁棒性。

---

## 🎯 局限与改进

PointNet **不显式建模局部邻域**。改进作品：
- **PointNet++** (Qi 2017b)：分层抽样 + 邻域聚合。
- **DGCNN**：动态图卷积。
- **PointTransformer**：注意力机制。

不过 **PointNet 本身因简洁高效仍被大量使用**——本论文就选 PointNet 而非 PointNet++。

---

## 🔑 对本论文的关键作用

本论文神经代理网络的 **前端编码器** 就是 3 层 PointNet：

```
物体局部点云 (N × 3)
        │ 1×1 Conv (即逐点 MLP)
        ▼  3 → 64 → 128 → 256
        │ Max pool
        ▼
全局特征向量 (256)
        │ concat with [质心(3), 密度(1), 刚度(22)]
        ▼  ← 共 282 维
5 层 MLP → 末态 [f(6), Δq(3), c_g(1)]
```

### 选 PointNet 的具体理由

1. **置换不变** —— 物体点云是无序的，不需要预定义顶点编号。
2. **轻量** —— 三层 MLP，参数 < 100k，训练 1 小时即可收敛。
3. **能直接吃 RealSense 点云** —— 真机部署 zero overhead。
4. **不需要 mesh 信息** —— 训练数据来自 YCB mesh，但部署时只需 RGB-D 点云。

### 与 GNN 的对比

如果换成 [[Ref-28-Pfaff-2021-MeshGraphNets|MeshGraphNets]]：
- ✅ 能学局部物理交互；
- ❌ 需要 mesh 拓扑 (真机抓取场景没有)；
- ❌ 训练成本和推理成本都更高。

PointNet 在"末态预测 + 输入是无序点云" 这一具体场景下是 **最优选择**。

---

## 🧭 延伸阅读

- 抓取应用 (PointNet 系列衍生)：[[Ref-46-Fang-2023-AnyGrasp]]
- GNN 模拟器对比：[[Ref-28-Pfaff-2021-MeshGraphNets]]
- 概念：[[../01-核心概念/神经物理模拟器 Neural Physics]]

---

*← [[_参考文献索引]]*
