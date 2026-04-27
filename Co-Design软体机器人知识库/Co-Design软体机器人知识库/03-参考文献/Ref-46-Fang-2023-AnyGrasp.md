# [46] Fang et al. 2023 · AnyGrasp

> **原文**: H.-S. Fang, C. Wang, H. Fang, M. Gou, J. Liu, H. Yan, W. Liu, Y. Xie, C. Lu. *AnyGrasp: Robust and efficient grasp perception in spatial and temporal domains.* **IEEE TRO**, 39(5):3929-3945, 2023.
> **译名**: 《AnyGrasp: 时空域中的鲁棒高效抓取感知》 — *IEEE 机器人学汇刊*, 2023.

`#文献-方法` `#抓取感知` `#里程碑`

---

## 1 · 研究问题

抓取位姿生成长期是机器人挑战之一: 给定 RGB-D 输入, 输出 "**在哪里、用什么角度、张多大**" 抓物体。上交大 Cewu Lu 团队 (上一代 GraspNet 系列作者) 的目标:

> 训出 **通用、鲁棒、实时** 的抓取位姿生成模型, 能处理 **未知物体、动态场景、多视角输入**, 并 **公开开源 / 提供 API** 方便其他研究使用。

AnyGrasp 已成为 **抓取领域事实标准** 之一, 大量后续机器人系统直接调用其 API。

---

## 2 · 数据集: GraspNet-1Billion

### 2.1 规模
- 100 万张 RGB-D 图。
- 每张含 ~1000 个标注抓取位姿。
- **总计 10 亿** ground-truth 抓取候选。

### 2.2 标注

每个抓取位姿是 6-DoF:
$$g = (\mathbf{t}, \mathbf{R}, w)$$
- $\mathbf{t} \in \mathbb{R}^3$: 夹爪中心位置。
- $\mathbf{R} \in SO(3)$: 旋转 (常用 6D 向量参数化)。
- $w \in [w_\min, w_\max]$: 开合宽度。

每个抓取标注:
- 是否成功 (仿真 + 真机验证)。
- 力闭合分数 (force closure score)。
- 鲁棒性指标 (扰动下保持抓取概率)。

### 2.3 物体覆盖
- 数百件不同物体 (含 YCB)。
- 数百场景 (桌面、堆叠、单物体)。

---

## 3 · 网络架构

```
输入 RGB-D 图
        │
        ▼ 点云转换 (depth + intrinsics)
点云 P ∈ R^{N × 3}
        │
        ▼ 主干: 3D 稀疏卷积 / PointNet++
体素 / 点特征
        │
        ┌─── ApproachNet → 抓取主方向 a ∈ S²
        │   per-point classification 抓取方向
        │
        ├─── OperationNet → 旋转角 + 宽度
        │   预测 (旋转矩阵 R, 宽度 w)
        │
        └─── ScoreNet → 抓取置信度 s ∈ [0, 1]
            评估抓取成功概率
```

输出 $K$ 个候选 $\{(g_k, s_k)\}_{k=1}^K$, 按置信度排序。

---

## 4 · 关键创新: 时空一致性

### 4.1 单帧 vs 视频

单帧抓取容易抖动 (随机相机噪声 → 不同帧选不同位姿)。AnyGrasp 引入 **时空平滑**:
$$L_\text{temporal} = \sum_t \|g_t - g_{t-1}\|^2$$

让相邻帧的抓取位姿保持相似。

### 4.2 多视角融合

多相机点云融合 → 减少遮挡:
$$P_\text{fused} = \bigcup_{c=1}^{C} T_c \cdot P_c$$
$T_c$: 相机 $c$ 到世界的变换。

**本论文 (Yi'25)** 用 2 个 RealSense D435 即此风格。

### 4.3 动态物体

对运动物体也能预测合理抓取 (利用时序模型)。

---

## 5 · 损失函数

### 5.1 多任务损失

$$L_\text{total} = \alpha L_\text{approach} + \beta L_\text{operation} + \gamma L_\text{score} + \delta L_\text{temporal}$$

### 5.2 各项细节

- **Approach**: 分类损失 (per-point softmax)。
- **Operation**: 6D 旋转 cosine 损失 + 宽度 L2。
- **Score**: 二分类 BCE。
- **Temporal**: 帧间平滑。

---

## 6 · 性能基准

### 6.1 GraspNet 评测协议

| 方法 | AP @ 0.4 | 推理时间 |
|---|---|---|
| 6-DOF GraspNet (旧) | 35% | 200 ms |
| GraspNet baseline | 50% | 100 ms |
| **AnyGrasp** | **70%** | **<50 ms** |

显著提升 + 实时。

### 6.2 真机部署
- 多类机械臂 (Franka, UR, ABB)。
- 多种夹爪 (parallel, 三指, 软夹爪)。
- 单物体抓取成功率 90%+。
- 杂乱场景 (多物体堆叠) 成功率 80%+。

---

## 7 · 完整使用流程

```
─── AnyGrasp Inference Pipeline ────────────
1) 采集 RGB-D (RealSense, Kinect, etc.)
2) 投影为点云 P
3) 输入 AnyGrasp 模型:
    candidates = AnyGrasp(P, scene_points, env)
4) 候选 = list of (translation, rotation, width, score)
5) 按 score 排序
6) (可选) 后处理:
    - 过滤工作空间外
    - 防碰撞过滤
7) 选择 top-1 (或前 K) 发送给运动规划
```

---

## 8 · 关键贡献

1. **大规模数据集 GraspNet-1B**: 推动学术界标准。
2. **时空一致性**: 视频流抓取减少抖动。
3. **多视角支持**: 减少遮挡导致的失败。
4. **实时 + 通用**: 一个模型 vs 多种夹爪 / 物体。
5. **开源 API**: 极大降低应用门槛。

---

## 9 · 局限

1. **依赖训练物体覆盖**: 极特殊形状 (透明 / 反射) 仍弱。
2. **不知道夹爪几何**: 假设标准平行夹爪, 自定义夹爪 (例如本论文长指) 需后处理。
3. **视场限制**: 必须 RGB-D 可见。
4. **不联合优化设计**: 只输出抓取位姿, 不能改夹爪。

---

## 10 · 历史影响

- 推广 "**通用抓取生成模型**" 范式。
- 商业化 (上海交大孵化项目)。
- 数千个机器人项目使用其 API。
- GraspNet-1B 是抓取领域 "ImageNet"。

---

## 11 · 与本论文 (Yi'25) 的关系

本论文 **抓取候选完全来自 AnyGrasp**:
- 论文 §3.3 描述: AnyGrasp 输出 → SDF 增广 → 神经代理打分 → Franka 执行。
- AnyGrasp 是输入端不可缺。

由于 AnyGrasp 默认夹爪短, 本论文长指会与原候选碰撞 → 加 SDF 增广 + 自身代理 重新排序。

---

*← [[_参考文献索引]]*
