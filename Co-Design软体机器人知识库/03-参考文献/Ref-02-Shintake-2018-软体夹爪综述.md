# [2] Shintake et al. 2018 · 软体机器人夹爪综述

> **引用**: J. Shintake, V. Cacucciolo, D. Floreano, H. Shea. *Soft robotic grippers.* **Advanced Materials**, 30(29):1707035, 2018.

`#文献-综述` `#软体夹爪`

---

## 🎯 这篇综述想做什么

软体夹爪是软体机器人最成功的子方向，但 **设计思路五花八门**——有充气的、拉绳的、磁吸的、粘附的。Shintake (EPFL Floreano 学生) 等的目标：

> 提出一个 **统一的分类框架**，把所有软夹爪按 **驱动机制** 归到 4 大类，并指出每类的优缺点和适用场景。

---

## 📖 四大分类

| 分类 | 工作原理 | 代表 |
|---|---|---|
| **1. Actuation** (主动驱动) | 充气、液压、SMA、EAP、腱拉 | 本论文、Soft Robotics Inc. 商业气夹 |
| **2. Controlled stiffness** (变刚度) | 颗粒阻塞、层阻塞、相变 | [[Ref-34-Wei-2016-ParticleJamming]]、[[Ref-38-Narang-2017-LaminarJamming]] |
| **3. Controlled adhesion** (粘附) | 壁虎刚毛、电吸附、静电 | [[Ref-43-Glick-2018-GeckoGripper]] |
| **4. Hybrid / underactuated** (混合 / 欠驱动) | 多机制结合 | [[Ref-10-Odhner-2014-SDMhand]] |

### 类别 1 详解：Actuation

- **气动 (PneuNet)**：硅胶腔体充气产生弯曲。优点：力大；缺点：需空压机。
- **液压**：响应慢但定位准。
- **腱驱动 (本论文)**：用绳索远程驱动柔性骨架。
- **EAP**：电场下变形，无机械传动。

### 类别 2 详解：Controlled Stiffness

通过 **可逆改变内部约束** 实现"软-硬切换"：
- 颗粒阻塞：抽真空让粉末互锁。
- 层阻塞：薄片摩擦自锁。
- 磁流变 / 形状记忆。

### 类别 3 详解：Controlled Adhesion

利用表面附着力代替机械夹持：
- 壁虎仿生：剪切下产生范德华力。
- 静电附着：高压电极。
- 磁吸：仅适用于铁磁物体。

### 类别 4 详解：Hybrid / Underactuated

把多种机制结合在同一夹爪上 → 鲁棒、通用，但设计复杂。Shintake 把 SDM Hand 等列入此类。

---

## 📐 性能比较表 (论文核心贡献)

文章给出一张大表，比较 ~30 种代表夹爪：
- 抓取力 (g)、最大物体尺寸 (mm)、响应时间 (s)、能耗 (W)、可靠性。

**对本论文的启发**：每类夹爪都有短板，**没有"万能" 夹爪** —— 这正是 co-design 价值所在：根据任务调出最适合的设计。

---

## 🔑 对本论文 (Yi'25) 的意义

- **本论文夹爪定位**：腱驱动 (类 1) + 结构刚度分布 (类 2 思想) 的混合，在 Shintake 框架内属"类 1 + 类 4 混合"。
- **基线选择**：本论文的对比组 "Soft / Semi-rigid / Rigid" 取自 Shintake 综述的典型参数。
- **优化空间**：综述指出 "designing stiffness distributions remains challenging, mostly heuristic" → **正是本论文要解决的问题**。

---

## 🧭 延伸阅读

- 软体机器人总论：[[Ref-01-Rus-2015-软体机器人综述]]
- 机器手百年：[[Ref-31-Piazza-2019-机器手百年]]
- 拓扑优化软夹爪：[[Ref-12-Chen-2018-拓扑优化软夹爪]]
- 生成式夹爪：[[Ref-26-Ha-2021-Fit2Form]]
- 概念：[[../01-核心概念/软体机器人 Soft Robotics]]

---

*← [[_参考文献索引]]*
