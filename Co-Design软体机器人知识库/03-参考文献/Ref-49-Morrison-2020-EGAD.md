# [49] Morrison, Corke & Leitner 2020 · EGAD 进化抓取分析数据集

> **引用**: D. Morrison, P. Corke, J. Leitner. *EGAD! An Evolved Grasping Analysis Dataset for diversity and reproducibility in robotic manipulation.* CoRR abs/2003.01314, 2020.

`#文献-数据集`

---

## 🎯 这个数据集想做什么

YCB / KIT 是 **真实物体扫描得来**——形状空间天然偏向"日常"。这导致抓取算法可能在"长得像日常物体" 的形状上表现好，**但对怪异形状泛化差**。Morrison (QUT) 等的目标：

> 用 **进化算法 (evolutionary algorithm)** 自动 **生成多样化 3D 物体集**，覆盖更广的形状空间，**所有物体可 3D 打印** 以便他人复现。

---

## 📦 数据集内容

- **2331 件 3D 物体**，按 "**复杂度 × 难度**" 网格排布：
  - 复杂度 (complexity)：表面凹凸、曲率变化。
  - 难度 (difficulty)：抓取分析得到的鲁棒性。
- 每件物体一个 STL → 可 3D 打印。
- 进化目标：**最大化物体集的形状多样性**。

### 进化算法生成

```
初始化: 随机几何
重复:
   1) 评估每个物体的复杂度+难度
   2) 在 (复杂度, 难度) 网格上保留最多样化代表
   3) 选出代表做交叉/变异 → 新一代
```

---

## 🔑 对本论文 (Yi'25) 的作用

本论文 §4.1 用 EGAD 物体作为 **out-of-domain 困难测试**：
- 形状极不规则 (训练数据没见过)。
- 测试代理模型的 **真泛化** (而非过拟合训练分布)。

实验结果显示：
- In-domain (YCB): 89-92% 成功率。
- Out-of-domain (含 EGAD): 81-84% —— 性能略降但仍合理，证明 co-design 找到的设计 **不是只对 YCB 形状特化**。

---

## 📐 关键概念：形状多样性 (Shape Diversity)
**含义**：物体集合在形状空间的"覆盖度"。
**使用场景**：
- 公平评估抓取算法泛化能力。
- 训练数据增强。

---

## 🧭 延伸阅读

- YCB：[[Ref-44-Calli-2015-YCB]]
- KIT：[[Ref-48-Kasper-2012-KIT]]

---

*← [[_参考文献索引]]*
