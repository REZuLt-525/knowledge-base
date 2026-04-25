# [46] Fang et al. 2023 · AnyGrasp

> **引用**: H.-S. Fang, C. Wang, H. Fang, M. Gou, J. Liu, H. Yan, W. Liu, Y. Xie, C. Lu. *AnyGrasp: Robust and efficient grasp perception in spatial and temporal domains.* **IEEE TRO**, 39(5):3929-3945, 2023.

`#文献-方法` `#抓取感知` `#里程碑`

---

## 🎯 这篇论文想做什么

抓取位姿生成长期是机器人挑战之一：给定 RGB-D 输入，输出"在哪里、用什么角度抓"。上交大 Cewu Lu 团队的目标：

> 训出 **通用、鲁棒、实时** 的抓取位姿生成模型，能处理 **未知物体、动态场景、多视角输入**。

AnyGrasp 已成为业界事实标准——很多抓取系统直接调用其 API。

---

## 🛠️ 实现路径

### 1. 数据集: GraspNet-1Billion
作者团队 2020 年发布 **GraspNet-1Billion** —— 100 万张 RGB-D 图，每张含 ~1000 个标注抓取位姿，**总计 10 亿** 个 ground-truth 抓取。

### 2. 网络架构

```
RGB-D 输入
    │
    ▼ 点云转换
点云 (N × 3)
    │
    ▼ 主干: 3D 稀疏卷积 / PointNet++
体素特征
    │
    ├── Approach Net: 抓取主方向
    ├── Operation Net: 旋转角 + 夹爪宽度
    └── Score Net: 抓取置信度
```

### 3. 关键创新

**(a) 时空一致性**
对视频流，AnyGrasp 让相邻帧的抓取位姿保持平滑——避免抖动。
$$L_\text{temporal} = \sum_t \|g_t - g_{t-1}\|^2$$

**(b) 多视角融合**
多相机融合点云 → 减少遮挡。
**本论文 (Yi'25) 用了两个 RealSense D435** 即此风格。

**(c) 动态物体**
对运动物体也能预测合理抓取。

### 4. 评估
- AP@0.4 (mean Average Precision) 显著高于此前方法 (GraspNet baseline、6-DOF GraspNet)。
- 推理时间 < 50 ms。

---

## 📐 关键技术

### 技术 1：6-DoF 抓取参数
$$g = (t, R, w) \in \mathbb{R}^3 \times SO(3) \times \mathbb{R}^+$$
- $t$：夹爪中心位置。
- $R$：夹爪朝向。
- $w$：开合宽度。

**使用场景**：
- 通用机器人抓取系统的标准接口。
- **本论文输入接口**：AnyGrasp 输出 → 本论文位姿增广 → 神经代理打分。

### 技术 2：抓取打分
$$s(g | \text{scene}) \in [0, 1]$$
**使用场景**：
- 排序候选位姿。
- **本论文做法**：用 AnyGrasp 提供初始候选，再用 **本论文的代理网络重新打分**——因为 AnyGrasp 不知道具体软夹爪几何。

---

## 🔑 对本论文 (Yi'25) 的关键作用

整个抓取流程的"输入端"：
1. RealSense → 点云。
2. **AnyGrasp 生成 100+ 候选位姿**。
3. 论文 §3.3.1 用 SDF 增广 (避免碰撞)。
4. 神经代理对每个 (位姿, 刚度) 打分。
5. 选 Top-B 进入梯度优化。

如果没有 AnyGrasp，本论文需要自己造抓取候选生成器——工作量翻倍。

---

## 🧭 延伸阅读

- 数据集 YCB：[[Ref-44-Calli-2015-YCB]]
- 点云特征基础：[[Ref-47-Qi-2017-PointNet]]
- 概念：[[../01-核心概念/抓取位姿 Grasp Pose]]

---

*← [[_参考文献索引]]*
