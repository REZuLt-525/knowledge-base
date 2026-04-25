# [48] Kasper, Xue & Dillmann 2012 · KIT 物体模型数据库

> **引用**: A. Kasper, Z. Xue, R. Dillmann. *The KIT object models database: An object model database for object recognition, localization and manipulation in service robotics.* **IJRR**, 31(8):927-934, 2012.

`#文献-数据集`

---

## 🎯 这个数据集想做什么

德国 Karlsruhe Institute of Technology (KIT) 团队建立的 **服务机器人物体数据库**。目标：

> 为家庭服务机器人 (清洁、收纳、烹饪辅助) 提供 **高精度 3D 模型 + 多视角图像** 的标准物体集。

特色与 [[Ref-44-Calli-2015-YCB|YCB]] 不同：
- KIT 偏重 **欧式厨房 / 浴室** 风格物体。
- YCB 偏重 **美式日用品**。

---

## 📦 数据集内容

- **110+ 件家居物体**：杯子、瓶子、餐具、玩具。
- **高精度 3D 网格** (扫描自真实物体)。
- **纹理图、多视角 RGB-D**。
- **物理参数** (质量、重心)。

---

## 🔑 对本论文 (Yi'25) 的作用

本论文 §4.1 把 KIT 物体作为 **out-of-domain 测试集**：
- In-domain: 45 件 YCB (训过)。
- Out-of-domain: 28 件 (KIT + EGAD + 部分 YCB)。

KIT 的物体形状与 YCB 风格差异显著 → 考验代理模型的 **真实泛化能力**。

---

## 🧭 延伸阅读

- YCB：[[Ref-44-Calli-2015-YCB]]
- EGAD：[[Ref-49-Morrison-2020-EGAD]]

---

*← [[_参考文献索引]]*
