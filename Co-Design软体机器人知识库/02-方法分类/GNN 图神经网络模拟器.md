# GNN 图神经网络模拟器

> **一句话理解**：把物理系统抽象为 **图 (节点=粒子/顶点, 边=相邻)**，用图神经网络预测下一时刻状态——比 FEM 快数量级。

---

## 1. 为什么 GNN 适合物理？

物理系统往往具有两个特点：
1. **局部交互**：一粒子只跟邻居有相互作用。
2. **几何不变性**：平移/旋转下规律不变。

图网络天然满足这两点：
- 局部消息传递 = 相互作用。
- 可以嵌入相对坐标实现 **SE(3) 等变**。

此外，粒子/顶点数可变 → GNN 可泛化到不同分辨率。

---

## 2. 代表工作

### 2.1 MeshGraphNets — Pfaff et al. 2021
[[../03-参考文献/Ref-28-Pfaff-2021-MeshGraphNets]]
- ICLR 2021 力作。
- 用自适应网格 + GNN 模拟布料、流体、结构。
- 首次在网格层面赶上 FEM 精度但快 10-100 倍。

### 2.2 Learning Particle Simulators — Sanchez-Gonzalez
(未直接引用，但属于同线)
- DeepMind GNS, GNS-M。

### 2.3 Inverse Design with GNS — Allen et al. 2022
[[../03-参考文献/Ref-20-Allen-2022-InverseDesignGNS]] 与 [[../03-参考文献/Ref-30-Allen-2022-GDM]]
- 用 GNS 做 **反向设计** (流固耦合)：给定期望行为，反向优化初始几何。
- GD-M 是升级版，Neural 反向学习高维设计参数。

### 2.4 CFD + GNN — de Avila Belbute-Peres 2020
[[../03-参考文献/Ref-29-deAvilaBelbute-2020-GraphPDE]]
- 结合可微 PDE solver 与 GNN 做流体预测。
- 展示两者互补：solver 保证物理一致，GNN 补 residual。

---

## 3. 特性对比

| 维度 | MLP + Point Cloud (本论文) | GNN Simulator |
|---|---|---|
| 推理速度 | 最快 | 较快 |
| 精度 | 末态预测好 | 逐步 rollout 更准 |
| 泛化性 | 受训练分布限制 | 换网格/规模有优势 |
| 几何能力 | 弱 | 强 |
| 实现复杂度 | 低 | 高 |

本论文选 PointNet+MLP 而非 GNN 的理由：
- 末态预测足够用，不需要 rollout。
- 22 维刚度向量较低维，MLP 即可学。
- 训练/推理成本更低。

---

## 4. Co-Design 中的 GNN 前景

- **拓扑优化**：节点删除 / 添加 → 适合 GNN。
- **Soft robot 大规模粒子**：GNS 已展示类似能力。
- **多体机器人** (如机械臂组装)：图结构匹配度天然。

---

## 5. 相关文献

- [[../03-参考文献/Ref-28-Pfaff-2021-MeshGraphNets]]
- [[../03-参考文献/Ref-29-deAvilaBelbute-2020-GraphPDE]]
- [[../03-参考文献/Ref-20-Allen-2022-InverseDesignGNS]]
- [[../03-参考文献/Ref-30-Allen-2022-GDM]]

---

*← 返回 [[_方法分类MOC]]*
