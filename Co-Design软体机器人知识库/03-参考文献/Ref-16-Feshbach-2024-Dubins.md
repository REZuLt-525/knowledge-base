# [16] Feshbach et al. 2024 · CSC-Dubins 规划式机构树设计

> **引用**: D. Feshbach, W.-H. Chen, L. Xu, E. Schaumburg, I. Huang, C. Sung. *Algorithmic design of kinematic trees based on CSC Dubins planning for link shapes.* **WAFR 2024**.

`#文献-方法` `#规划式设计`

---

## 🎯 这篇论文想做什么

机器人连杆通常是直线段——但 **弯曲连杆** 可以更省空间、更适应复杂工作环境。如何系统设计弯曲连杆？UPenn Cynthia Sung 团队的目标：

> 借用 **Dubins 路径** (经典最短路径) 把"机器人连杆形状设计" 变成 **规划问题**：给定末端位姿约束，规划一条合法的连杆形状。

---

## 🛠️ 实现路径

### 1. Dubins 路径回顾
Dubins 1957 证明：在曲率受限平面上，从一点到另一点的最短路径必属于 6 种之一：
$$\text{LSL, LSR, RSL, RSR, LRL, RLR}$$
其中 L = 左弯，R = 右弯，S = 直线。**CSC** = 弧 + 直线 + 弧。

### 2. 把 Dubins 应用于连杆设计

- **状态空间**：连杆当前末端位置 + 切向。
- **动作**：弧段 (固定半径) 或直线段。
- **目标**：到达指定末端位姿。
- 解 = 一条由若干 CSC 组成的路径 → 即为连杆形状。

### 3. 树形机器人扩展

对于多自由度机器人 (kinematic tree)，作者把 Dubins 路径扩展到：
- 每个 link 是一段 CSC 路径；
- 关节连接点是路径切点；
- 整个机器人 = 多条 CSC 路径的"链接"。

### 4. 制造约束

- **平面 (planar)** 实现：用 laser cutter 切割。
- **可折叠 (foldable)**：Origami 结构。

---

## 📐 关键公式与使用场景

### Dubins 路径方程
对单段弧：
$$x(s) = x_0 + R\sin(\theta_0 + s/R) - R\sin\theta_0$$
$$y(s) = y_0 - R\cos(\theta_0 + s/R) + R\cos\theta_0$$

**使用场景**：
- 平面机器人连杆设计 (本论文相关)。
- 自动驾驶轨迹规划 (经典用途)。
- 折叠/origami 机器人设计。

---

## 🔑 对本论文 (Yi'25) 的意义

- **规划式 vs 优化式 co-design** 的两端：
  - 本论文：连续优化刚度。
  - Feshbach: 规划离散 link 形状。
- **结合可能**：未来给本论文夹爪加"弯曲手指" 选项时，可用 Dubins 规划生成几何。

---

## 🧭 延伸阅读

- 被动夹爪计算设计：[[Ref-17-Kodnongbua-2023-PassiveGrippers]]
- RoboGrammar 离散设计：[[Ref-08-Zhao-2020-RoboGrammar]]
- 方法分类：[[../02-方法分类/规划式设计]]

---

*← [[_参考文献索引]]*
