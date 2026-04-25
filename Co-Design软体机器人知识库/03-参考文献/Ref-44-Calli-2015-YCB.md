# [44] Çalli et al. 2015 · YCB 物体与模型集

> **引用**: B. Çalli, A. Walsman, A. Singh, S. S. Srinivasa, P. Abbeel, A. M. Dollar. *Benchmarking in manipulation research: The YCB object and model set and benchmarking protocols.* CoRR abs/1502.03143, 2015.

`#文献-数据集` `#里程碑`

---

## 🎯 这个数据集想做什么

机器人抓取研究 **缺乏统一基准**：每个团队拿不同物体做实验，结果难以对比。Çalli 等的目标：

> 联合 Yale + CMU + Berkeley 资源建立 **YCB (Yale-CMU-Berkeley)** 物体集，并给出 **标准评测协议**，使全球抓取研究可比较。

YCB 数据集已成为机器人抓取研究的事实标准 (类似计算机视觉的 ImageNet)。

---

## 📦 数据集内容

### 1. 物理物体集 (Physical Objects)
77 件日常物体，分 5 类：
- **Food items** (饼干盒、罐头、水果)。
- **Kitchen items** (锅、碗、餐具)。
- **Tool items** (锤子、扳手、剪刀)。
- **Shape items** (球、立方体、圆环)。
- **Task items** (积木、九柱戏)。

### 2. 数字模型 (3D Models)

每件物体提供：
- **高分辨率 mesh** (~$10^4$-$10^6$ 三角形)。
- **纹理图** (PBR 材质)。
- **多视角 RGB-D 数据**。
- **质量**、**重心**、**摩擦** 等物理参数。

### 3. 标准评测协议

文章定义 6 套抓取/操作协议：
- Pick & Place。
- Stacking。
- Tool Use。
- Pouring。
- Threading。
- Insertion。

每套协议规定起始/终止条件、评分标准。

---

## 📐 关键数据格式

### 数据点示例
```json
{
  "object_id": "002_master_chef_can",
  "mass_kg": 0.414,
  "dimensions_mm": [102, 102, 139],
  "mesh_file": "google_512k/nontextured.ply",
  "rgbd_views": ["NP1_0.png", "NP1_45.png", ...]
}
```

### 引用规范
所有用 YCB 的文章必须 **报告物体 ID 和评测协议**，确保结果可重现。

---

## 🔑 对本论文 (Yi'25) 的关键作用

- **训练数据基础**：本论文 §4.1 用 **45 件 YCB 物体作为 in-domain 训练数据**，**28 件 (YCB + KIT + EGAD) 作为 out-of-domain 测试**。
- **公平基准**：与之前 co-design 论文比较时，使用同一物体集才有意义。
- **物理参数复用**：mesh + 质量 + 摩擦直接喂给 Warp FEM 仿真。

没有 YCB，本论文的实验部分就没有可信度基础。

---

## 🧭 延伸阅读

- KIT 物体集 (out-of-domain)：[[Ref-48-Kasper-2012-KIT]]
- EGAD 进化生成集：[[Ref-49-Morrison-2020-EGAD]]
- AnyGrasp 同样基于 YCB：[[Ref-46-Fang-2023-AnyGrasp]]
- Aaron Dollar 还作 YCB 维护：[[Ref-10-Odhner-2014-SDMhand]]

---

*← [[_参考文献索引]]*
